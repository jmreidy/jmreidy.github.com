---
layout: post
title: Introducing vows-bdd
comments: true
---

My work with Node.js has recently transformed from the occassional experiment to
the building of a full-on side project. It's fascinating to be working with a set
of tools that are so rapidly developing; the energy and excitement in the Node
ecosystem far outweighs any annoyances with needing to adjust to changing APIs.

### Embracing Vows

One of the tools which I've really enjoyed learning is [Vows.js](http://vowsjs.org).
Vows.js is a fully async Javascript testing library that allows for blazingly fast
execution. (It's amazing how little changes in test speed can make TDD feel
much less onerous.)  Vows' syntax can be a little tough to adjust to, but the speed,
combined with the ability to construct "test batches", make the barrier to entry
worth the effort.

While Vows is a near pefect tool for unit testing, I began to run into trouble with
full-stack integration testing. Check out the contrived example below:

``` coffeescript

vows.describe("Creating a User")
  .addBatch
    "To create a user":

      "Given the server has started":
        topic:
          server.ready, @callback

        "and the DB is populated":
          topic:
            setupFixtures @callback

          "and another setup function is required":
            topic: ->
              anotherSetup @callback

            "When visiting the form at /user/new":
              topic: ->
                ctx = this
                server.ready ->
                  zombie.visit "http://localhost:#{port}/user/new", ctx.callback
                return

              "then should allow entry of a username": (err,browser,status) ->
                assert.ok browser.querySelector ":input[name=username]"

              "should allow entry of first and last name": (err,b,s) ->
                assert.ok b.querySelector ":input[name=firstName]"
                assert.ok b.querySelector ":input[name=lastName]"

              "should allow entry of a password": (err,b,s) ->
                assert.ok b.querySelector ":input[name=password]"

              "should allow confirmation of a password": (err,b,s) ->
                assert.ok b.querySelector ":input[name=passwordConfirm]"

              "and submitting the form":
                topic: (browser,status) ->
                  browser.fill("username", "test")
                         .fill("firstName", "Justin")
                         .fill("lastName", "Reidy")
                         .fill("password", "foobar")
                         .fill("password", "foobar")
                         .pressButton "Sign Up!", @callback

                "should create a new User": (err,browser,status) ->
                  assert.ok findNewUser()

        .export module

```

Ouch. All that nesting makes for pretty unreadable code - even in CoffeeScript.

### vows-bdd: an easier way to test 

I've never been a huge Cucumber fan, but modeling integration tests in a "given-when
-then" format makes them much easier for me to construct mentally. While the 
[prenup](https://github.com/jadell/prenup) library made long Vows tests easier to read by introducing a 
fluent interface, I thought that I'd love to marry this fluent approach with BDD's
"given-when-then". 

A couple days later and vows-bdd was born: 

```

Feature("Creating a User")
  .scenario("Create a User via Form")

  .given "the server is running", ->
    server.ready @callback

  .and "the DB is popuplated", ->
    setupFixtures @callback

  .and "another setup function is called", ->
    anotherSetup @callback

  .when "I visit the form at user/new", ->
    zombie.visit "http://localhost:#{port}/user/new", @callback

  .then "I should see a username field", (err,browser,status) ->
    assert.ok browser.querySelector ":input[name=username]"

  .and "I should see entries for first and last name", (err,browser,status) ->
    assert.ok browser.querySelector ":input[name=firstName]"
    assert.ok browser.querySelector ":input[name=lastName]"

  .and "I should see a password entry", (err,browser,status) ->
    assert.ok browser.querySelector, ":input[name=password]"

  .and "I should see a password confirmation", (err,browser,status) ->
    assert.ok browser.querySelector, ":input[name=passwordConfirm]"

  .when "I submit the form", (browser,status) ->
    browser.fill("username", "test")
           .fill("firstName", "Justin")
           .fill("lastName", "Reidy")
           .fill("password", "foobar")
           .fill("password", "foobar")
           .pressButton "Sign Up!", @callback

  .then "a new User should be created", (err,browser,status) ->
    assert.ok findNewUser()

  .complete()
  .finish(module)

```

Checkout the [repo on Github](https://github.com/jmreidy/vows-bdd)
for more documentation and info.

### Open-Source is scary!

The vast majority of my career has taken place in enterprise world, where code is
rarely, if ever, shared with anyone on the outside, and even *using* open-source
code is considered dangerous. While I've contributed to OSS libraries
before, I've never posted my own creation. It's fairly nerve-wracking to put your
work out in the wild, especially when you're building on top of such amazing
contributions from incredible developers.

In fact, I probably would've sat on this little library forever, constantly trying
to perfect it, if not for a 
[fantastic blog post](http://blog.nodejitsu.com/the-nodejs-philosophy)
written by Charlie Robbins([@indexzero](http://twitter.com/indexzero))
over at Nodejitsu. I think his post perfectly expresses the spirt and mentality 
of the Node community right now, 
and his emphasis on experimentation via small bits of functionality made me realize:
this code may not be perfect, and it might not even do that much, but if *I* find it
useful, then maybe someone else will too.

So please - [checkout vows-bdd](https://github.com/jmreidy/vows-bdd) and give it a whirl. 
Any suggestions or comments would be very much appreciated!



