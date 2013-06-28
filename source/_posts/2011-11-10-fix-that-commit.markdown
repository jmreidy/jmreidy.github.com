---
layout: post
title: "Fix That Commit!"
date: 2011-11-10 16:55
comments: true
categories: git
---
While it sometimes seems as if everyone has already switched to git (or a similar distributed
version control system), I still meet people who are only leaving subversion 
behind now. A lot of git's idioms are impenetrable to those who are new to it, but the presence
of free resources like [Pro Git](http://progit.org/book/) and [git ready](http://gitready.com/) make the adjustment much easier.

Even though I've used git for awhile, I still need to look up the best way of performing
a task every now and then. One trick that comes in handy is editing existing commits. 
`git --amend` allows for adding new files or changes to a previous commit, but you can
have even more control than that -- changing *any* pre-pushed commit in your history. 

Just get the hash of the commit you want to change. With a simple `git rebase <commit>^ --interactive`, 
you can then make any changes you want. For each file, just type `git add <file>`. When
you're done, `git commit --amend` and `git rebase --continue` will finish up your editing.

It's always useful to review and cleanup your commit history before pushing your changes
upstream or issuing a pull request. With the ability to revisit any previous commit, you
can be more comfortable in a 'commit-all-the-time' workflow.
