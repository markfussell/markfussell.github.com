---
comments: true
date: 2008-08-29 00:08
layout: post
slug: a-taste-of-ruby-part-2
title: A Taste of Ruby (Part 2)
wordpress_id: 42
categories:
- Dynamic
- Ruby
- Smalltalk
---

This is a multi-part series.  The first in the series is [here](/blog/a-taste-of-ruby).



## Tower of Hanoi -- Procedural With Input


The second example from "A Taste of Smalltalk" is in Chapter 3, page 36 and does basic IO.  Except that Smalltalk-80 includes a Windowing graphics system so it can automatically open Dialogs and other things.  Smalltalk (started in 1972) was one of the environments that _created_ personal computers and included abilities in its VM and Libraries that are now submerged into the operating system (some 30 years later).  Now that these abilities are 'intrisically' present, there are many different ways to access them -- so Ruby does not include graphics itself (in its own terms) but calls extensions that provide those graphics.  Given I don't really want to go into Tk or Web programming when discussing a language itself, I will simplify the problem and make it so we only need to get the value somehow... So 'gets' is fine.

<!-- more -->



### Tower of Hanoi -- Procedural with Input -- Ruby


The only thing we need to do is get the height from the user, so we can just ask for it:
```ruby
# Do the tower
puts "How tall a tower?"
print ">"
height = gets
move_tower(height.to_i,1,3,2)
```

Quite simple and probably the only interesting thing is that 'height' did not need to be declared at all.  Personally I find this _unpleasant_ and bad for system stability, but I can see the convenience of it.  I would rather Ruby had a way to formally declare local variables, instance variables, etc. and in a 'strict' mode would give feedback.

Smalltalk does formally declare variables, so it is more compile-time checked than Ruby.  The Smalltalk IDEs also warn about things in a way that usually avoids typos, but that is not part of the language itself.  But I would think the simplicity of the syntax and programming model makes the Smalltalk compiler much easier to write and optimize, but that is just an impression.



### Where Next?


Next we should go to Chapter 4 — Page 44. The Third Example (with Real Hanoi Classes)
