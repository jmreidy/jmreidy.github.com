---
layout: post
title: "Error Isolation with Promises"
date: 2012-10-01 09:39
comments: true
categories: [development, Node.js, JavaScript, promises, Q]
---

The entirety of the Javascript and Node.js universe is based around a
continuation-passing coding style -- that is to say, callbacks. This style of
async coding greatly aggrevates Node critics, who constantly argue that
continuation-passing invariably results in "callback hell"
spaghetti code for any non-trivial problem. Node.js proponents respond that
continuation-passing presents an easier mental model for async coding
than alternative approaches, and that properly abstracted code avoids any
gross "chains of doom."

This post has nothing to do with that argument.

If you're coding in Javascript, you're going to be using callbacks, and if you're
building a library, you're going to be supporting callbacks. It's the reality
of the ecosystem. But there are certain problems for which some syntactic sugar is helpful.
[Promises](http://wiki.commonjs.org/wiki/Promises) provide a wrapper for callbacks that
allow for a chainable API around a sequence of continuations. They're not a native part of
the language, but there's [a strawman](http://wiki.ecmascript.org/doku.php?id=strawman:concurrency#promises_and_promise_states)
for the inclusion of Promises in a future version of JS. In the meantime, you can use
the excellent [Q Library](https://github.com/kriskowal/q) from
[Kris Kowal](https://github.com/kriskowal).
jQuery includes its own Promises implementation (`$.ajax` calls return Promises), but they're slightly
divergent from the Q spec. You can read [this excellent guide](https://github.com/kriskowal/q/wiki/Coming-from-jQuery)
explaining the differences between jQuery and Q promises, if you're coming from one direction
or the other.

So much for introductions. I've been using promises for awhile now in some of my
Node code, and there's several cases where their inclusion can clean up an API. (One
area: Mongo integration, where any simple action can require several callbacks.) One benefit
of Promises is that they can provide exception isolation, just like continuation-passing.
[The Q wiki](https://github.com/kriskowal/q/wiki/On-Exceptions) provides the following
example:


``` javascript
var info, processed;
try {
  info = JSON.parse(json);
  processed = process(info);
} catch (exception) {
  console.log("Information JSON malformed.");
}
```

In the above snippet, errors generated from the parser and the process function arne't isolated.
Promises, meanwhile, offer this exception isolation out of the box:

``` javascript
var processed = Q.call(function (json) {
  return JSON.parse(json);
})
.then(function (info) {
  return process(info);
}, function (exception) {
  console.log("Information JSON malformed.");
})
```

In the second example, the process function is only executed if `JSON.parse`
doesn't throw an error. Conversely, this same goal could be achieved
with continuation-passing:

``` javascript
var processFn = function(cb) {
  asyncJSONParser(json, function(err, info) {
    if (err) {
      console.log("Information JSON malformed.");
      return cb(err);
    }
    cb(null, process(info));
  });
};
```

One caveat is that when you're chaining your promises together, the behavior is slightly
different than you might expect:

``` javascript
var existingUser;
var promise = User.findOne({username: username})
  .then(function (user) {
    if (!user) {
      throw new Error("Username is incorrect");
    }
    existingUser = user;
    return User._hashPassword(password, user.get('salt'));
    }, function(error) {
      console.error('error in findOne');
      throw error;
  })
  .then(function (hashedPassword) {
    if (existingUser.get('hashedPassword') === hashedPassword) {
      return existingUser;
    } else {
      throw new Error("Password is incorrect");
    }
  }, function(error) {
    console.error('error in _hashPassword');
    throw error;
  });
```

In the above example, if there is an error in `User.findOne`, the error is
propagated throughout the chain, which means that your console will have both
`error in findOne` and `error in _hashPassword` printed. There's no way
of "breaking" from the promise chain early. But here's the thing: you don't have to.
Unless you're interested in transforming error objects, there's not much
point in catching promise errors in intermediary steps in the chain. So the
above code would be abbreviated:

``` javascript
var promise = User.findOne({username: username})
  .then(function (user) {
    if (!user) {
      throw new Error("Username is incorrect");
    }
    existingUser = user;
    return User._hashPassword(password, user.get('salt'));
  })
  .then(function (hashedPassword) {
    if (existingUser.get('hashedPassword') === hashedPassword) {
      return existingUser;
    } else {
      throw new Error("Password is incorrect");
    }
  });

//later in chain or in a diffent function
promise.then(function (eventualResult) {
    //do something with result
  }, function (error) {
    //handle error
  });
}
```

There's simply no point to having `then` failure handlers that just throw
an error, since the library throws and propagates errors for you automatically.
In promise chains, then, you have the benefit of isolating exceptions (only proceeding
from step to step if everything is ok) without the extra semantics of `if (err) return cb(err)`
from continuation passing.


