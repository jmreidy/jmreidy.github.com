---
layout: post
title: "Towards a Unified JS Stack"
date: 2013-04-02 19:49
comments: true
categories: [development, Node.js, JavaScript, Backbone]
---
It’s been a long time since I’ve written anything for this site. Neither neglect nor procrastination prompted my absence, but rather, lots and lots of coding. All that code has left me with a surfeit of topics to cover here, so instead of talking about what I’ve been working on, I’m going to dive straight into the good stuff.

I want to write about the current “Holy Grail” of web application development: a unified application stack. Any dev who’s built a thick-client web app (regardless of library - Ember, Backbone, this problem is independent of tool choice) has dealt with the frustration of recreating logic in multiple parts of the application stack. It’s incredibly annoying to be writing the same code to handle model accessors, view state, and rendering logic in two different places; it’s even more irksome to be fixing bugs in one or both areas as your application matures. But let’s back up a bit. Why do we need to be writing the same logic in multiple places at all?

Generally speaking, modern web app development requires a choice from one of three architectural thrusts:

* thick server, thin client: view logic is written on the server, and rendering happens there as well. The client code deals with interactivity and AJAX requests, but is mostly just updating areas of the page with rendered view content returned by the server. This approach is [most famously advocated for by 37 Signals](http://37signals.com/svn/posts/3112-how-basecamp-next-got-to-be-so-damn-fast-without-using-much-client-side-ui). A thick server simplifies view code, but does so at the expense of larger AJAX payloads; why pull fully rendered code when you can just pull JSON data and render it with a client-side template?
	
* thick client, thin server: This architecture inverts the first approach; the server provides an API, which is basically just a wrapper around data access and caching. All of the app logic resides in the client. This approach can make for quick interactivity and rendering on the client, but presents a massive problem with “first load” delay, as the client needs to make two connections to the server: the first to load the client application code, and the second to pull the appropriate data to render. You may remember this problem from [Old New Twitter](http://engineering.twitter.com/2012/05/improving-performance-on-twittercom.html).

* thick client, thick server: Unsurprisingly, this approach combines both of the previous options. Views are rendered on the server, which means the initial load is fast, but the client handles all the rendering from that point forward. The good news is that you have two fairly independent, (hopefully) highly performant application structures; the bad news is that you’re now repeating logic in view templates, view states, model helpers, even route handlers.

One exciting prospect of Node.js is that it makes Javascript an “isomorphic language”, which is to say that “any given line of code (with notable exceptions) can execute both on the client and the server” (h/t [Nodejitsu](http://blog.nodejitsu.com/scaling-isomorphic-javascript-code)). But that said, there haven’t been very compelling demonstrations that solve the code-duplication problem.

In the last few months, I’ve come up with a solution that allows for code reuse on client and server, leading to a unified codebase for handling view rendering, view states, and model interaction. It’s using components and libraries that you already know: Browserify, Backbone, Express, and Handlebars. But I’m going to hold off on describing that solution for now, and instead write about Rendr.

Rendr [is the result of Airbnb’s first Node app](http://nerds.airbnb.com/weve-launched-our-first-nodejs-app-to-product), and it’s also the library that finally prompted me to close Vim and start writing here again. It’s the fulfillment of an idea that I first heard about from [Tim Branyen](https://github.com/tbranyen): what if we used Express to render the same Backbone Views that we render on the client? That question is what prompted me on my own path towards application stack unity, so I’ve been very eager to learn more about Rendr. 

Rendr took Tim Branyen’s initial idea and made it a reality. It’s important to note that Airbnb’s experiments here weren’t an excuse for architecture aeronautics; rather, they were expressly concerned with solving the “time to content” problem, which is to say, the rendering delay that Twitter famously faced. And by unifying client and server code, Rendr not only makes developers’ lives easier, it solves that time to content problem.

Rendr has a number of features and design approaches that I very much like:

* it’s described as “a library, not a framework”

* it doesn’t create a server side DOM

* it doesn’t try to collapse backend data access into the client tier, as some other frameworks have done

* it utilizes existing, familiar code (Express and Backbone)

* it keeps the rendered DOM on first load and attaches client objects to it, rather than instantiating client views and replacing the DOM with the results of those client views.
	
This last item is particularly important: rendering a page on the server, sending it to the client, and then recreating the DOM on the client can lead to odd rendering issues, interactivity problems, and problems with event listeners. But dynamically instantiating client objects and attaching them to an already-rendered DOM is not a trivial problem, and I commend [Spike Brehm](https://github.com/spikebrehm) and the rest of the Airbnb team for taking the time to get this right.

You should definitely take the time to [explore Rendr’s codebase](https://github.com/airbnb/rendr). If it existed when I first set out to solve this same set of problems last year, then I probably would’ve abandoned my own quest and just used this library. But the key point of differentiation with the approach I took from Rendr’s design is modularity. Is it possible to achieve application stack unity, while using objects and code that can be implemented with any number of frameworks in any number of ways?

Stay tuned.