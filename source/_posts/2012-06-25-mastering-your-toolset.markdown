---
layout: post
title: "Mastering Your Toolset"
date: 2012-06-25 15:29
comments: true
categories: [development, tmux, Vim]
---

When I originally started thinking about writing this post, I was going to
phrase it in terms of spring cleaning. When I moved to Portland, I took the
opportunity to reasses my development environment and to cleanup the libraries
and source repositories I had scattered across my system. Since then, I've
become the proud owner of a new retina MBP, so I've ended up building a new
environment from scratch. Provisioning a new laptop is incredibly fast
now; if you have your projects stored in version control, and you keep your
dotfiles controlled as well, setting up a new laptop is as easy as:

* installing [Homebrew](http://mxcl.github.com/homebrew/)
* installing [iTerm2](http://www.iterm2.com/#/section/home)
* brew install git, macvim, tmux, tmuxinator, whatever langs and dbs you need
* pulling down your [.dotfiles](https://github.com/jmreidy/dotfiles)
* installing and syncing your [Dropbox](http://www.dropbox.com)

I was developing on my new machine the day it arrived.

That experience of provisioning a machine so easily, and getting productive
with it in no time at all, left me reconsidiering my original post on my
toolset, which primarily boils down to zsh, tmux, and Vim.  There's plenty of
material online to learn about these tools; a few of note are:

* [The Text Triumvirate](http://www.drbunsen.org/text-triumvirate.html)
* [Unix as IDE](http://blog.sanctum.geek.nz/series/unix-as-ide/)
* [Coming Home to Vim](http://stevelosh.com/blog/2010/09/coming-home-to-vim/)
* [A Tmux Crash Course](http://robots.thoughtbot.com/post/2641409235/a-tmux-crash-course)
* [Dotfile Discovery](http://wynnnetherland.com/journal/dotfiles-discovery)
* [Beautiful Tools](http://www.mahdiyusuf.com/post/24784023641/beautiful-tools)

So why should you adopt tools with such a steep learning curve? Most discussions of IDEs
and development environments quickly devolve into pointless arguments over the
superiority of one tool over another, when in most cases the tool isn't the most
important factor; differences in productivity will more frequently boil down to
experience and mastery of a toolset, rather than the toolset itself. I've seen
masters of Vim, Eclipse, IntelliJ, and Textmate; in every case, the interface
breaks down and gets out of the way of the user, who always "does" without any
thought as to "how to do".

So if mastery is more important than toolset, does your choice really matter?

Yes.

There's two huge benefits of the Vim/tmux/shell toolset: ubiquity, and
permanence.

Mastery of a tool takes many, many, many hours: <blockquote
class="twitter-tweet"><p>"Malcolm Gladwell's observation that it takes 10,000
hours to become truly expert at something." A little more than 3 years
typically</p>&mdash; Jeff Atwood (@codinghorror) <a
href="https://twitter.com/codinghorror/status/1066301753"
data-datetime="2008-12-19T02:48:10+00:00">December 19, 2008</a></blockquote>
<script src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

So whatever tool you choose, you want to be able to use it as much of the time
as possible, in as many environments as possible.

A mastery of Eclipse, for example, doesn't help you when you need to ssh into a
unix environment (but vi/m would).  Knowing Visual Studio inside and out can't
help you if you need to work on a Max or Linux box (but gvim and cygwin will
still work on Windows). There's not a lot that can compete with Vim in the ubiquity
category; Emacs is the only other tool that comes to mind.

Permanence is a whole other matter. Technology changes constantly; some tools
are improved, others are abandoned, still others fall out of use entirely. Are
you a Flex developer who spent spent huge amounts of time mastering FlexBuilder
and Eclipse?  Out of luck now. Textmate? The tool that influenced countless
developers to leave the Windows ecosystem eventually became pseudo-abandonware,
leading to the current growth of tools liks [Sublime Text 2](http://www.sublimetext.com/2).

Vim has been around since 1991. Its progenitor vi was written in 1976. (Again,
Emacs is the only competitor here, having also been written in 1976.)

All of this is to say: **don't** go and abandon your current toolset for mine!
Just because what I use works for me, doesn't mean it's the best choice for
you. If you've already mastered your toolset and are happy with it, **stay** there:
you'd be wasting many productive hours on a new tool for the potential of
hating it. And as wonderful as Vim/tmux/shell are for most, it's not the right
toolset for everyone or everything. If you're doing mostly front-end web UX
work, for example, [Coda 2](http://panic.com/coda/) would probably be perfect
for your needs; iOS means you're going to be stuck with Xcode, for better or worse.

But if you're feeling constrained by your current toolset, or are making the jump
from one technology to another, and haven't given "the text triumvirate" a try,
I'd very strongly recommend doing so. It's worth the effort.





