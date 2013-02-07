---
comments: true
date: 2008-08-28 04:34
layout: post
slug: being-ephemeral
title: Being Ephemeral
wordpress_id: 35
categories:
- Ephemeral
- Ruby
- Smalltalk
---

There are lots of topics in programming languages, IT infrastructure, and system design that turn out to be all about persistence vs. being ephemeral.  I don't believe people realize it, but "being ephemeral" is the core question and the standard answer should be?  42.  Which given it is >0 is 'Yes'.  So "Be Ephemeral" unless you can't possibly be ephemeral.

<!-- more -->

When can't you be ephemeral?... turns out it is in a very small number of cases.  Ruby is more ephemeral than the standard Smalltalk 'image' approach, so it wins.  Xen/VMWare is more ephemeral than a physical machine, so it wins.  _Scripted_ vm image building is more ephemeral than image snapshots, so its win.  Credit-card receipts in a retail store... well those have to be remembered (persistent) or you could lose money or customers or both.   So make that persistent :-)

Performance is always a consideration: 'rebuilding' an image could be time-expensive, but the old adage [annotated] of:
   * Make it work (ephemerally if possible)
   * Make it right
   * Make it fast (pull the ephemerally _iff necessary_)
is the right approach.   You may be amazed at how ephemeral things can be.  NowUR built virtual desktop in less than 30 seconds -- by being ephemeral.  An with Amazon EC2, for many things I can just plug in a script and have lots of 'yummy' things happen in the same amount of time.

Why is "Being Ephemeral" right?  Well, there is one simple reason:
   * Inventory of outdated parts is very expensive (Theory Of Constraints, Lean Methods, etc.)

And if you can make it through the old adage without needing any Inventory, you have a better (less expensive, more agile, higher velocity) solution than someone that cheats by using Inventory.
