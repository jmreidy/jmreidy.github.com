---
layout: post
comments: true
---
In the last week, I've been experimenting a greal deal with node.js and
CoffeeScript. I've been thoroughly enjoying the node programming approach and
the general "feel" of the CoffeeScript language (or really, dialect would be a
better term, since CoffeeScript is really just Javascript). I'll be writing
much more about my experiments soon.

In the meantime, a quick note about getting up and running yourself. I'm using
[npm](http://npmjs.org/) to manage dependencies for my node projects.
If you have an old version of npm sitting around,
you should note that it handles the installation of depencies differently now
than it did pre-1.0. <code>npm install "module"</code> defaults to installing
the dependency in the local directory - which is fine for a particular project,
but doesn't help you when you want to invoke a module from the command line or
use a version acorss multiple projects. Globally installing a module requires
you to supply a "-g" argument: <code>npm install -g "module"</code>. You can
learn more about the rationale for the change
on [the node.js blog](http://blog.nodejs.org/2011/03/23/npm-1-0-global-vs-local-installation/).
