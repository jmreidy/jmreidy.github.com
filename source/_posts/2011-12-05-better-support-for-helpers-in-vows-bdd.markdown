---
layout: post
title: "Better Support for Helpers in vows-bdd"
date: 2011-12-05 15:31
comments: true
categories: [vows-bdd,vows,testing]
---

This last week has been a particularly busy one, so I've been delayed in blogging,
but I released v0.2 of vows-bdd to NPM a week ago. There's two changes of note.
The first is proper documentation for the library: you can see the docco documentation
[here](http://rzrsharp.net/vows-bdd). If you're new to vows-bdd, or just want to learn
more about how it's working, the docs should provide all the answers you're looking
for.

The more important change relates to the testing syntax. In some conversations I had
online with [Graeme Foster](http://graemef.com/), I realized that the vows-bdd syntax
left some room for improvement. Specifically, the syntax introduced a potential for
redundancy when creating labels for integration tests. While named functions could
be used for test statements that were repeated, there wasn't an easy to way to handle
repeated labels.

In order to make the use of test helpers even easier with vows-bdd, I have adjusted
the `given/when/then/and` syntax to allow for array arguments *in addition to* the
previous arguments of `label,test_function`. It's easiest to demonstrate the change with
an example. One area of code that frequently is repeated is login code in integration tests.
Here's how I had my tests setup with vows-bdd before:

``` coffeescript

# a test snippet

  .given "I am an admin user", ->
    @user = Model.Factory.create "user", {role: 'admin'}
    @callback()
    return

  .when "I go to login", ->
    zombie.visit Server.path_for("/login"), @callback

  .when "I click login", (browser,status) ->
    browser.fill("email", "#{@user.email}")
           .fill("password", "#{@user.password}")
           .pressButton "Login", @callback

  .then "I should be on the admin index", (err,browser,status) ->
    Test.assert.shouldBeOn browser, "/admin"

# snippet from another test

  .given "I am an admin user", >
    @password = "foo"
    @user = Model.Factory.build( "user", password: @password, passwordConf: @password, role: "admin" )
    @user.save @callback

  .when "I am logged in", >
    user = @user
    cb = @callback
    pw = @password
    zombie.visit Server.path_for("/login"), (e,browser,status) >
      browser.fill "email", user.email
      browser.fill "password", pw
      browser.pressButton "Login", cb
    return

```

Even with the previous syntax, this code could be improved by creating a named `login`
function that handles the duplicated code. And you could add a property to that function
that provides a label. But I felt that the vows-bdd syntax should be improved to make
this code repetition even easier. So the new `[label,function]` syntax looks like
this:

``` coffeescript

# first test

  .scenario("'/admin' is accessible to logged in admin users")

  .when(I.login_as "an admin")

  .then "I should be on the admin index", (err,browser,status) ->
    Test.assert.shouldBeOn browser, "/admin"

# other test

  .scenario("Logging Out")

  .given(I.login_as "an admin")

  .when "I click logout", (browser,status) ->
    browser.pressButton "Logout", @callback

  .then "I should be redirected to the login page", (err,browser,status) ->
    #continues

```

The `login_as` function is included in a helper file:

``` coffeescript

exports.login_as = (type_of_user) ->
  label = "I login as #{type_of_user}"
  fixture = () ->
    cb = @callback
    @user = user = Model.Factory.create type_of_user.match(/a[n]? (\w+)/)[1]
    zombie.visit Server.path_for("/login"), (err,browser,status) ->
      browser.fill("email", "#{user.email}")
             .fill("password", "#{user.password}")
             .pressButton "Login", cb
    return
  return [label,fixture]

```

Note that the function returns an array of arguments to be supplied to the vows-bdd
method. Also note that the fixture method is using `this` &mdash; which will point to the
vows `context` when the test is evaluated &mdash; and that the fixture returns `undefined`
to indicate to vows that the test should be performed asynchronously.

I think this new syntax works really perfectly for integration tests; it goes a long
way to supporting declarative testing style.

As always, please let me know if you run into any problems with the library or
have any suggestions!
