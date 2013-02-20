---
layout: post
title: "Being a git about everything (Intro)"
date: 2013-01-11 18:16
comments: true
categories: Git
---

There are times when a new technology comes along, that at first appears to be pretty similar to
existing technology, but certain characteristics make for radically different or just nicely new solutions.
A recent example of this is 'git' and similar distributed version control systems (DVCS).  They may
at first appear to be an interesting version of centralized version/content management systems, but
they are really much more... a core piece of technology useful for many things.

This is a series about how to use git to solve many different problems, some obvious and some more unusual.
I hope a few of them are interesting to readers.

<!-- more -->

There are alternative DVCSs but I am not going to compare or translate examples... at least not on a first pass.
Git was created by Linus Torvalds in 2005.  It was initially quite 'raw' and still maintains much of that rawness,
but other tools (e.g. github) and general improvements have made it more accessible.  For source-code control, git has some big wins
in collaboration and offline work compared to SVN, Perforce, and the like.  Whether these are worth some trade-offs depends on your team
and company... but that is a different topic.

## Git Syntax / Overview

You can learn more about git somewhere like github:

   * [http://learn.github.com/p/intro.html](http://learn.github.com/p/intro.html)

Git has a lot of commands, but around seven are core to standard git flows:

   * clone &mdash; copy a repository from a remote location
   * fetch &mdash; get updates from a remote repository
   * merge &mdash; merge changes from one branch into the current branch
   * pull &mdash; 'fetch' and then 'merge'
   * add &mdash; add changes to the commit stage of the current branch
   * commit &mdash; commit the changes into the current branch
   * push &mdash; try to make a remote branch look like the current branch

Actually of those seven 'pull' basically replaces/combines two of them, so you get
down to about five with a core loop like this:

   * clone
      * pull
      * make changes (if needed)
         * add
         * commit
         * push

Of these commands only 'push' can fail due to timing.  'push' is transactional
and you can be contending with other people or machines that pushed to the same branch
as you between when you pulled and you pushed.  Merging can fail, but that
represents some actual file-level conflict vs. a timing issue (someone beat you to the 'push').
A proper 'commit' can't fail because it is local.


### Repositories and branches

At least to begin, all these solutions will use a standard centralized repository approach
and generally not use branches.  So you don't have to worry about git's ability to
deal with many remotes and which branch a given git repository is using.  By default
at any current moment there is only

   * local: 'master' <-> remote: origin/master

where things get interesting because there could be 100 different machines each with their own 'local'.

### Problems with git / DVCS

The biggest issue with git and similar DVCSs is that their model doesn't work well for large amounts of
binary assets.  Large amounts of binary assets could occur either because there are a large number of
assets available at any time but only a subset are needed (so with Perforce or SVN many people would not
check out those directories) or more commonly, a modest number of binary assets are frequently
changing.  Because a git repository contains all assets throughout time and to work with a git repository
you clone the whole thing, having a large amount of binary assets punishes everyone.

What is a large amount?  That depends on the circumstances, but generally passing 100MB can start to
become painful depending on the purpose git is being used for.  Having 1GB of textual files in
a single git repository (even over time) is an unusual thing.  GBs of images is common.

So our first problems and solution is going to have to deal with this critical issue.

Enter the annex: [Annex](/blog/git-about-everything-annex/)







