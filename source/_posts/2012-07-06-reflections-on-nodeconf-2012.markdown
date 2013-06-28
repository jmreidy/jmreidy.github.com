---
layout: post
title: "Reflections on NodeConf 2012"
date: 2012-07-06 20:16
comments: true
categories: [development, Node.js, JavaScript, NodeConf]
---
NodeConf 2012 was my first real "developer" conference. I had attended
conferences in the past, but because of the heavy sponsorship of industry, most
had felt like trade shows or job fairs. NodeConf was advertised as a conference
for developers, by developers; Mikeal Rogers, the event's main organizer,
boasted that NodeConf 2012 would feature more core committers as presenters
than any other Node.js conference.

As the event grew closer, my curiosity about what NodeConf would entail
heightened. Mikeal didn't divulge a clue as to the event's schedule, and even
perusing the twitter feeds of probable speakers and
[NodeConf's site on lanyard](http://lanyrd.com/2012/nodeconf/) didn't reveal any helpful
intelligence. At the opening party on Sunday, a general sense of cautious
curiosity permeated the crowd. No one had ever attended a conference where the
schedule would be revealed in realtime (appropriately for Node); my own
conference inexperience left me more curious than most.

Mikeal *had* revealed two details:  the major themes of each day of the
conference.  Day 1 was structured around the world of Node; Day 2 around the
power of Node. NodeConf began with Node creator Ryan Dahl reconstructing the
origins of the framework from livejournal entries and blurry photos of late
night coding session in Germany.  But as the day continued, I couldn't see how
the talks related to each other, or what I should be taking away from them.
Each of the presentations, while excellent and often inspiring, didn't leave me
with a consistent narrative. I had expected "the world of Node" to consist
mostly of talks like Isaac Schlueter's "State of Node" presentation: perhaps a
roundup of the big open source projects and frameworks that dominate the Node
ecosystem, and a discussion of where they were headed.  Instead, talks covered
everything from realtime solutions with [socket.io](http://socket.io/), to
[the creation of a whole new language](https://github.com/indutny/candor). I had
started the first day of NodeConf curious, but ended it confused.

Day Two changed all that.

As one talk on the "power of Node" led into the next, the topics and structure
of the first day of NodeConf started to make sense to me. Each speaker
demonstrated amazing new contributions to the Node ecosystem, from the
Node-inspired [Luvit framework](http://luvit.io/) built with
[Lua](http://www.lua.org/), to bleeding edge debugging tooling with dtrace, to
ridiculous hardware demonstrations featuring human-detecting arduinos,
three-dimensional saws, and
[radar laser robots](https://dl.dropbox.com/u/3531958/nodeconf/index.html#/) controlled by
Javascript. These talks didn't just demonstrate powerful and frequently
unexpected uses of Node: they helped me make sense of the previous day's
selection of talks.

Node is not *just* the Node framework: it's a philosophy, an approach to
solving problems. The first inkling of "Node as philosophy" came in
[substack's post on Node's aesthetic](http://substack.net/posts/b96642/the-node-js-aesthetic),
discussing the "limited surface area" approach to Node's modular problem-solving. But
NodeConf demonstrated that the node philosophy goes much further than that:

1. async all of the things: Ryan's initial work with libuv focused on the
challenges of creating an async IO layer that worked across platforms, but that
work has been extended with new languages like
[Fedor Indutny's](https://twitter.com/indutny/) Candor and
[Tim Caswell's](https://github.com/creationix/) Luvit framework. But it's not just
file IO - [Max Ogden](https://twitter.com/maxogden) showed that the importance
of a beautiful async API
[could be extended to browsers](https://github.com/maxogden/domnode) (utilizing ideas put forth by
substack in day one), while [Rick Waldron](https://twitter.com/rwaldron)
brilliantly demonstrated how JavaScript's async nature could be used to solve
problems which had always been approached synchronously. (Like robots. With
lasers.)

2. embrace the hard problems: JavaScript may frequently be maligned for its odd
"bad parts", and JS developers have been looked down on in the past because of
the history of DOM based spaghetti code, but NodeConf 2012 demonstrated a
community that was unabashed in its tackling of serious computing issues. From
[Felix Geisendorfer's's](https://twitter.com/felixge/) writing of a binary MySQL parser that
surpassed a c library in performance, to the community's embracing of the
concept of IO streams, to the use of dtrace in debugging: Node developers on
the bleeding edge are not shying away from the toughest technical problems, but
instead are taking them on at full speed and with high expectations.

3. a virtuous cycle of framework and community: Node's "limited surface area"
design of the core framework, along with the ease of use of NPM for publishing
open source code and small useful modules, have led to an explosion of code. At
NodeConf 2011, around 1800 packages had been published to NPM; now there are
over 12000! Modules range from
[steaming cups of earl grey](https://github.com/substack/earl-grey), to
[karaoke bar APIs](http://voiceboxpdx.com/), to websocket adapters and arduino libraries.
The design of the framework drove its community adoption, but the community
adoption has further developed the framework.

In the last talk of day two, [Elijah Insua](https://twitter.com/tmpvar)
discussed the hardware hacking he'd done with Node. At one point, he mentioned
that he came up with an idea that needed a midi driver, and he said something
along the lines of: he never expected needing a midi driver, but as soon as he
needed one, he came across one on NPM. The word that came to mind was
serendipity: the contributions to Node, and the explorations of its philosophy,
are feeding on themselves and driving completely unexpected innovation.

As I thought of the serendipitous nature of the Node ecosystem, Mikeal's
construction of NodeConf finally clicked for me. The two days of talks had been
divided into groups of four presentations around a single theme; neither the
themes nor the talks were made public, but each talk and theme bled into the
next, with later speakers surprisingly stating how happy they were that a
previous speaker had touched on a similar topic. The talks seemed to develop
naturally, if unexpectedly; the overall direction was unclear to everyone
except Mikeal, and gave the impression of the community simply exploring new
ideas and pushing boundaries organically. The serendipity of Node's
development, then, found itself reflected in Mikeal's construction of the
conference. Does this sound ridiculously meta? Yes, but it completely worked.

As Node grows up and attracts adoption, it will be increasingly difficult to
maintain the philosophically-driven, community-minded hacker ethos of the Node
ecosystem, but I hope that Mikeal and the other stewards of the Node community
can continue to foster talks and events that focus on the behind-the-scenes
developers who are laying the building blocks for the future of this
fascinating set of technologies.
