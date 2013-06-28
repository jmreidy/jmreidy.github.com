---
layout: post
title: |
  "This" in Anonymous Functions
comments: true
---

Sometimes it's difficult to find an easy answer to common coding questions. Because
Google ignores most punctuation, even the most exact queries fail to produce
relevant results. Questions about the nature and value of `this` in Javascript present
a particularly thorny problem, since "this" is about as common a word as you can find.

A common point of confusion in Javascript is that the value of `this` depends entirely
on a function's context, and can even be overridden depending on how a function is
invoked. Hopefully this post will save time for new Javascript devs trolling through
Google's depths:

>In anonymous functions, the value of `this` refers to the anonymous function itself,
and not the calling scope. However, the nature of closures allows the anonymous function
to still refer to the calling object.

The above rule is demonstrated with the following nodejs code:

``` javascript
var that = this;
that.foo = 'bar';
process.nextTick(function() {
  console.log("this " + this.foo + " and that " + that.foo) });
```
