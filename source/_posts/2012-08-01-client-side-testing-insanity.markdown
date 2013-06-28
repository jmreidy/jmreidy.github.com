---
layout: post
title: "Client-Side Testing Insanity"
date: 2012-08-01 10:31
comments: true
categories: [development, Node.js, JavaScript, tdd, unit testing]
---
Client-side unit testing is crazy.

Before you skip below to yell at me in the comments section, I don't mean that
the *idea* of client-side unit testing is crazy. On the contrary: we need
client-side tests more than ever! With ever-increasing complexity in our
frontend code, not to mention the reliance on frameworks, building a web
application without client-side tests is completely inadvisible.

But the process. Ohhhh, the *process* of client-side testing is bonkers.

Let's take the example of a modern Node.js web app setup.
I’ve got the app’s configuration wired with [grunt.js](http://gruntjs.com).
(If you’re not using grunt, you should be!) During development, server
files are watched for changes and automatically kickoff [mocha](http://visionmedia.github.com/mocha/) tests, which
display their results in the grunt console. Great!

Client-side changes kickoff their own unit tests, which of course are also
using mocha for consitency in testing vocabulary. But because of the nature of
client-side testing, we can’t just call `require` on the unit under test.
Instead, we need something like the following setup:

* [requirejs](http://requirejs.org/) for managing dependencies

* [PhantomJS](http://phantomjs.org/) for running a headless browser

* [grunt-mocha](https://github.com/kmiyashiro/grunt-mocha), which manages the
PhantomJS process and reports test results to the grunt console

* an HTML fixture for loading the requirejs config, the browser environment, and test files

This setup worked. It also felt downright medieval compared to the simple
server-side mocha testing configuration.

I’m not the only person who’s frustrated with this process. Chris Dickinson at
[Urban Airship](http://urbanairship.com/) recently released [Drive.js](https://github.com/urbanairship/drive.js),
a unit testing framework and test driver that eases some of the above
configuration. But in any of the setups I've seen, your client-side “unit” tests feel
much more like integration tests. If you need to load your entire client-side
framework, load a headless browser, or even fire up an instance of your server
to test a single function of your client-side code... why not just ignore
client-side unit testing and skip straight to integration tests? The lack of
true isolation of the system under test could lead to some frustrating false
results anyway.

There needs to be a better way. I’m not looking for a lot in my testing setup:

* I want to run my client-side tests with the same mocha vocabulary and
configuration as my server tests

* I want to isolate my tests to requiring a single file and its immediate
dependencies, and testing it

That’s really it. And I think I’ve found a solution.

First, I’ve needed to refactor my client-side code to follow the
[CommonJS requires model](http://wiki.commonjs.org/wiki/Modules/1.1). I’ve also switched to
[browserify](https://github.com/substack/node-browserify) from requirejs, although this switch
is no longer a necessity — requirejs has introduced a
[new configuration option](https://github.com/jrburke/r.js/blob/master/build/example.build.js#L412),
`cjsTranslate`, which will wrap CommonJS defined modules. Using the CommonJS
approach means that my mocha tests can require my client-side files. Great.

But the much larger problem with client-side testing is those pesky browser
globals: the `window`object, the DOM itself, not to mention global code like
jQuery, Backbone, etc.  How can we test code that relies on the presence of
those globals, *without* resorting to running PhantomJS?

The answer comes in two parts: [jsdom](https://github.com/tmpvar/jsdom/), and
[the Node.js vm module](http://nodejs.org/api/vm.html).

jsdom provides a server-side implementation of the DOM, while the vm module
provides the “global” object that we can use to provide a fake browser
environment to our client-side files. While I’m still fleshing out the
implementation, the results so far have drastically improved my ease with
client-side testing.

Here's how it works. In a mocha test file, instead of a strict require call, I’m using a helper
(passing `this`, the mocha test context):

``` javascript

setupClientScript.call(this, "admin/postEdit", done);

```

`setupClientScript` loads a jsdom window, injects global javascripts (like jQuery),
and wires the client-side file being tested. Here it looks in a slightly simplified form:

``` javascript

var setupClientScript = function(path, done) {
  var relPath = context.clientRoot + path;
  var self = this;
  jsdom.env({
    html: '<html><head></head><body></body></html>',
    src: context.globalScripts,
    done: function(errors, window) {
    self.clientRequires = requireClientScript(relPath, window, {});
    self.clientRequires[path] = self.clientRequires[relPath];
      done();
    }
  });
};

```

Remember, the client-side `require` can’t *just* inject the DOM into the
file under test, it also needs to make sure the file’s dependencies have access to
the DOM:

``` javascript

var requireClientScript = function(path, window, r) {
  var script = fs.readFileSync(require.resolve(path)).toString();
  var module = { exports: {} };
  var context = vm.createContext(global);

  context.window = window;
  context.exports = module.exports;
  context.module = module;
  context.require = function(path) {
    r[path] = requireClientScript(path, window);
    return r[path];
  };
  vm.runInNewContext(script, context);

  if (r) {
    r[path] = module.exports;
    r.window = window;
    return r;
  } else {
    return module.exports;
  }
};

```

And that’s it. Now, any test function can call `this.clientRequires` to have
access to the client-side file under test. No test server being spun up. No
complex configuration or requirejs rewiring. No full-on headless browsers like
PhantomJS. Just a single `before` setup call, and you can test with
mocha as you normally would.

As I wrote above, I’m still fleshing out the implementation, working out the
interface and fixing deficiencies. I’ll be publishing the code when things are more
mature, but in the meantime I'd love to hear some feedback.





