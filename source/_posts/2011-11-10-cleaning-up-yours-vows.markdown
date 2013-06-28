---
layout: post
title: "Cleaning Up Yours Vows"
date: 2011-11-10 11:07
comments: true
categories: [javascript,vows,node.js]
---
Learning a framework as you're TDD-ing a project can be dangerous. But even if you
know what you're doing, some of the conveniences of node.js can make you a bit sloppy
with your code. Namely, the ability to pull functions from files as you need them and pass
them around makes for some interesting problems. 

``` javascript foo.js
exports.add = function(a,b) {
  return a+b; 
}
exports.subtract = function(a,b) {
  return a-b;
}
```
``` javascript bar.js
var add = require('foo').add;
console.log(add(1,2)); // 3
```
If your functions are free of side effects, you're not exactly committing
a serious coding sin. But go beyond the simple example above -- starting to pass 
around whole objects, or breaking the law of demeter -- and soon your code doesn't feel right.

I quickly ran into this problem with my vows tests. As I was about to merge a feature
branch to master, I reviewed my code and was a bit horrified at what I saw in my tests:
``` coffeescript user_spec.coffee
{cleanupDB} = require "../helpers"
{vows,server} = require "vows"
mongoose = require "mongoose"
User = mongoose.model "User"
assert = require "assert"
Sinon = require "sinon"
{Factory} = require "../helpers"
```
And this was just *one* of my tests. Yikes. Lots of bad things going on here:

* My test has a dependency on my model implementation, and not just the model itself;
if I switch the model off mongoose in the future, my test itself will need to change.
* My test has a dependency on a particulra spying/mocking framework (Sinon)
* There's a lot of confusing imports; why is `server` required off of vows? 
* Repeating requires - one for `cleanupDB` and one for `Factory`.

Bad bad bad.

My tests needed to be refactored to deal with this madness. I realized that there was
repeated code throughout my vows tests; not only could I clean things up with some proper
encapsulation, I could DRY my vows as well. 

One cool feature of node is that if you include an `index` file in a directory, that
file is loaded when you require the directory. So I created a new `spec/helpers` directory,
and placed an `index` file inside.

``` coffeescript spec/helpers/index.coffee
process.env.NODE_ENV = 'test'

module.exports =
  Server:  require "#{__dirname}/server_helpers"
  Test:  require "#{__dirname}/test_helpers"
  Model:  require "#{__dirname}/model_helpers"
```

And each of these helper files now encapsulates logic and functionality that I use
across my tests: 
``` coffeescript server_helpers
http = require "http"

exports.port = port = 3003
server = (require "#{__dirname}/../../app")
server.ready = (callback) ->
  if @active
    process.nextTick callback
  else
    @active = true
    server.listen port, (err,result) ->
      process.nextTick callback
  return

process.on "exit", ->
  if @active then server.close()

wait = ->
  if !@active 
    process.nextTick wait
  else
    return

server.ready wait

makeRequest = (url,params,method,callback) ->
  params ||= ""
  encoding = 'utf-8'
  server.ready ->
    request = http.request 
      host: "127.0.0.1"
      port: "#{port}"
      path: url
      method: "#{method}"
      headers:
        'content-type': "application/x-www-form-urlencoded"
        'content-length': params.length

    request.on 'response', (response) ->
      response.body = ''
      response.setEncoding encoding
      response.on 'data', (chunk) -> 
        response.body += chunk
      response.on 'end', ->
        callback null, response
    if params 
      request.write params
    request.end()


exports.server = server

exports.get = (url,params,callback) ->
  if arguments.length == 2 
    callback = arguments[1] 
    params = null
  makeRequest url,params,"GET",callback

exports.post = (url,params,callback) ->
  makeRequest url,params,"POST",callback
```

``` coffeescript test_helpers.coffee
Sinon = require "sinon"
assert = require "assert"
http = require "http"
Model = require "./model_helpers"

exports.spy = (object,method) ->
  Sinon.spy object, method

exports.stub = (object, method, fn) ->
  Sinon.stub object, method, fn 
  
exports.spyRender = ->
  @response = http.ServerResponse.prototype
  Sinon.spy @response, "render"

exports.spyModel = (klass,method) ->
  Sinon.spy (Model.getType klass), method

exports.assert = assert

```

```coffeescript model_helpers.coffee
mongoose = require "mongoose"
cleaner = new (require "database-cleaner")("mongodb")

exports.cleanupDB = ->
  mongo = mongoose.connection.db
  cleaner.clean mongo 

exports.Factory = (require "#{__dirname}/factories").Factory

exports.getType = (Klass) ->
  mongoose.model Klass

```

My test logic dependencies are clean and encapsulated, making for both clearer
`require`s blocks and clearer tests themselves.

```coffeescript user_spec.coffee
{Model,Test,Server} = require "../helpers"
vows = require 'vows'
User = Model.getType "User"

#code removed for brevity

  .addBatch
    "Hashes password:":
      "when a new user is created":
        topic: ->
          @user = Model.Factory.build "User" 
          Test.spy User, "makeSalt"
          Test.spy User, "hashPassword"
          @user.save @callback
          return

        "it creates a salt and stores it": (err,user) ->
          Test.assert.ok User.makeSalt.calledOnce
          Test.assert.ok user.salt == User.makeSalt.returnValues[0]
        
        "it encrpyts the password": (err,user) ->
          Test.assert.ok User.hashPassword.calledOnce
          Test.assert.ok User.hashPassword.calledWith @user.password 

      teardown: ->
        User.hashPassword.restore()
        User.makeSalt.restore()
        Model.cleanupDB()
```








