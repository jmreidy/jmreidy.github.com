---
layout: post
title: In Theory, Not Practice
date: 2013-04-11 19:49
comments: true
categories: [development, Node.js, JavaScript, Backbone]
---

In [my previous post](http://rzrsharp.net/2013/04/02/towards-a-unified-js-stack.html), I wrote about the quest towards unifying web application architecture around a single shared codebase. [Airbnb’s Rendr library](https://github.com/airbnb/rendr) provides one path forward on that quest. In this post, I’d like to provide a high-level explanation of my own solution for unifying application code.

There’s a few segments of an application’s code that will almost certainly be repeated on both the client and server: views, view state, and data models (excluding data access). To a certain extent, routing could be repeated as well; if you’re not using hashbangs, you’ll need to implement the same named routes on the client and server (although the actions you perform for those routes will be dramatically different).

The solution I’ve put together is still a work in progress, and is more a set of practices and conventions than a library. This flexibility means that while I’ve been implementing the approach using Handlebars, Backbone, and Express, you could theoretically use a completely different set of libraries. In this post, I’ll go over the basic ideas behind my approach; I’ll be putting together a sample implementation on Github that I’ll be able to write about in more detail in the future.

##Unifying View Templates
Sharing template code is perhaps the easiest step towards achieving web application unity. With a full-JS stack, you’re compiling the same templates for rendering both server-side and client-side. So why not use the same templates for each?

In my experiments, I’ve been using [Handlebars](http://handlebarsjs.com/) as my templating system. It’s based on the Mustache syntax, but has a number of powerful features, including partials and custom helpers. Handlebars is also the templating system behind Ember.js.

The key to sharing templates is the realization that what constitutes a “partial” can be relative to where the template is used — that is to say, the same template can be used as a full template on the client, and a partial template on the server. My templates are divided into three groups:

* client - client only templates
* shared - used on both the client and server
* server - server only templates

For the client, the templates in the client and shared folders are pre-compiled in a Grunt step, so for all intents and purposes there’s a single flat collection of templates. The server, however, references all of the templates in the shared folder as partials. What’s more, view helpers (in this case, special Handlebars tags) are stored in a `shared/helpers` directory that is referenced by both the client (in a Grunt step) and the server.

If you’re confused now, you won’t be when you see the actual implementation in my next post. What sounds complicated here is quite straightforward in practice.

##Populating View Templates
If it’s easy to share templates on the client and server, getting the right data into those templates is a bit of a harder proposition. Handlebars renders data by running the template against a “context”, which is just a plain old hash. But how can we make sure that the contents of that hash are synchronized when rendering the same view on both the client and server?

The problem is exacerbated with how Backbone muddles view rendering logic. In Backbone, there’s not a fine line between the View logic (in Backbone.View) and Model logic (in Backbone.Model and Backbone.Collection). Backbone Views end up performing view state management by manipulating model instances, in order to avoid bloating Models with View specific code. But adding stateful logic into a Backbone View causes two problems:

1) it makes for larger, more complex views, as concerns are split between UX event management and controlling model state;

2) it makes it harder to share code with the server, as rendering state gets locked in a Backbone view.

We can get around these problems by reconsidering the role of a Backbone View — separating its concerns into that of Mediator and View Model. The Mediator role is one of event manager and template marshal; the view model is, most basically, a state controller.

By splitting the state controller into an object of its own concern, you can extract the logic into a module that can be shared by both the client (via Browserify) and the server (via require). The Backbone Mediator simply invokes the View Model, declaratively calling functions on it as users interact with the app — transitioning Backbone away from its traditional role of MVC framework into more of [an MVVM implementation](http://addyosmani.com/blog/understanding-mvvm-a-guide-for-javascript-developers/).

A View Model just returns a hash with the current state of data, which is supplied to the template. On the server, a View Model is instantiated with data accessed from the backend; the results of the View Model instance’s `toJSON` method are passed to the template for rendering. On the client, the same pattern is repeated, but the data is accessed via AJAX instead.

##Sharing Models on Client and Server
Synchronizing model code on both the client and server is perhaps the hardest step towards achieving web application unity. The challenge primarily derives from a characteristic of Model code that we’ve inherited from the Rails world: specifically, a conflation of Model concerns between “data model object” and “data access object”. A data model object should identify a model object’s properties, its getters and setters, its relationships, and some convenient accessors; a data access object should deal with querying data sources for populating the data model objects.

Unfortunately, Backbone follows in the tradition of Rails’ ActiveRecord, flattening data accessing and data modeling into a single object. This flattening causes problems even beyond unifying a web application stack. How many Backbone applications have you seen that globally override the sync method, and use properties on model classes to determine how sync invocation works? (Not to pick on Backbone; it’s just the web application framework with which I’m most familiar.)

For now, I’ve abandoned trying to get Backbone Models and Collections to work on both the client and server. But there’s alternatives that can fill the gap nicely. The first of which I’m familiar is [Dan Dean’s Tubbs library](https://github.com/dandean/tubbs). Tubbs provides a DSL for defining data model objects, but also provides a pluggable dataStore. A single model module can be defined in a shared folder, and then required on both the client and server, with both supplying their own dataStore.

In order to make the separation between data modeling and data accessing even clearer, and the sharing of model code on the server and client easier, I created by own [voltron-model](https://github.com/jmreidy/voltron-model/tree/0.1.x) library. While Voltron isn’t ready for primetime (yet), it should make the unified definition of data models significantly easier.

Whichever modeling library is chosen, the basic approach should be the same: data models are defined in a shared folder. These models are then imported by code on the server (or packaged with Browserify for the client). The server and client should decorate the models appropriately (either by adding to the model’s prototype or adding an accessor component). Thus, a model can be made to access Mongo or Postgres on the backend, or a REST service on the client.

The benefit of using Backbone on the client is that it doesn’t care if you only use limited parts of the library. Arrays of event-emitting models can provide much of the same benefit of Backbone Collections and Models, and Backbone Views and Routers see no difference.


##Wrapping Up
All this discussion may be a bit confusing without a concrete implementation to reference; hopefully, I’ll be able to put together a sample application in the next few weeks. But the basic takeaway of the post is that unification of JS web applications can be made significantly easier with two factors:

* structuring your code so that it can be required by the server or packaged for the client in the same ways, whether by ingesting templates as partials or packaging code with Browserify;

* following the Node.js approach of writing small, single-purpose modules means that there’s less boilerplate to prevent you from using code in multiple environments


