---
title: Classes in Coffeescript
layout: post
comments: true
---

Javascript is a prototypal language, a concept which is quite challenging for 
programmers coming from almost any other language.
The most well-known explanation of Javascript's prototypal nature is 
[written by Douglas Crockford](http://javascript.crockford.com/inheritance.html),
the famous author of [the book Javascript: The Good Parts](http://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742). 
Many Javascript libraries attempt to plaster class structures into the language, 
but they all depend on varying amounts of magic which can make me slightly 
uncomfortable.

To a certain extent, Javascript's prototypal nature should be embraced; as a 
developer begins to understand how prototypes work, they [recognize their power over traditional class-based code](http://blog.izs.me/post/4731036392/evolution-of-a-prototypal-language-user).
That said, classical inheritance wouldn't be as popular as it is if it weren't a 
fantastic means of modeling real world objects, so it'd be nice to "map"
Javascript code cleanly to a classical style, without manipulating the language unnecessarily.

Enter Coffeescript's `class` directive.

Coffeescript doesn't add classes to Javascript. However, 
Coffeescript *does* combine some syntactic sugar with code generation to allow 
classical-style coding.

Observe the following Coffeescript:

``` coffeescript
class Ninja
  constructor: (@numOfShurikens) ->
  
  throwShuriken: ->
    @numOfShurikens--

class Ronin extends Ninja
  
  constructor: (numOfShurikens) ->
    super numOfShurikens+1 #ronins know to carry a spare

```

I find Coffeescript code to be quite readable, but if you're completely unfamiliar
with the syntax, be sure to check out its [fantastic documentation](http://www.coffeescript.org).
So with the Coffeescript above, we've created two "classes": a superclass, and a subclass
which modifies its parent's constructor while inheriting a function. What I find 
exciting about this example is that Coffeescript isn't actually creating any "pseudoclass"
objects behind the scenes; it's simply mapping existing Javascript concepts into
a more developer-friendly DSL.

Here, take a look at the generated JS:

``` javascript
var Ninja, Ronin;
var __hasProp = Object.prototype.hasOwnProperty, __extends = function(child, parent) {
  for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; }
  function ctor() { this.constructor = child; }
  ctor.prototype = parent.prototype;
  child.prototype = new ctor;
  child.__super__ = parent.prototype;
  return child;
};
Ninja = (function() {
  function Ninja(numOfShurikens) {
    this.numOfShurikens = numOfShurikens;
  }
  Ninja.prototype.throwShuriken = function() {
    return this.numOfShurikens--;
  };
  return Ninja;
})();
Ronin = (function() {
  __extends(Ronin, Ninja);
  function Ronin(numOfShurikens) {
    Ronin.__super__.constructor.call(this, numOfShurikens + 1);
  }
  return Ronin;
})();
```

See? No weird manipulation of Object prototypes, creation of fake classes, anything
like that. Just proper Javascript. Still, I found some of the idioms here a little
confusing when I first started looking at generated Coffeescript code. 

First, why are the "classes" implemented as self-executing functions? Coffeescript
is simply protecting the code against potential namespace violations. Everything that
happens within the Coffeescript "class" is actually executable Javascript code - 
similar to how Ruby's classes work, actually. The "class" that's returned, then,
is really just your constructor function, its modified prototype, and any properties
attached to the function object. Pretty straightforward. The major downside with this
approach is that you can't create any private variables! (Normally in JS you can hide a
pseudo-private variable within a constructor function's closure, but Coffeescript's
class implementation prevents this.) Coffeescript's solution is to simply refer to
"private" variables with an underscore and hope developers act like adults, but I'm
guessing this open-mindedness will be tightened up a bit as Ecmascript 5 becomes 
the standard.

But... what is that *mess* of generated code at the top? The first part is pretty
clear - just copy object properties from one "class" to another, so that (for example)
Ninja.belt will be inherited by Ronin.belt. And assigning a `__super` pointer to
the parent class is straightforward. But what is that "constructor" code?
Why not just point the "subclass" prototype to the parent prototype? 

It turns out this approach is the cleanest means of maintaining the prototype chain
across functions. Since there's only one prototype object for any function, simply
assigning `Ronin.prototype=Ninja.prototype` will share a single object instance, so
adding properties to Ronin's prototype would actually also add properties to Ninja's
prototype -- they'd be the same object instance. By creating an intermediary
object, the prototype chain is kept intact while breaking any chance for incorrectly
shared state. Finally, all Javascript objects have a constructor property which 
point to the "creator" function, whose initial value is determined by the object's 
prototype. The intermediary object (which serves as the
child's prototype) thus gets a pointer back to the appropriate function.

Taken in full, Coffeescript's "class" DSL demonstrates what I love most about the 
language: distilling Javascript idioms into clean, readable, maintainable code, and
eliminating boilerplate, but avoiding any non-Javascript abstractions or manipulations.
It just can take awhile to fully understand what's going on under the hood.


