---
comments: true
date: 2008-08-29 00:27
layout: post
slug: a-taste-of-ruby-part-3
title: A Taste of Ruby (Part 3)
wordpress_id: 46
categories:
- Dynamic
- Ruby
- Smalltalk
---

This is a multi-part series.  The first in the series is [here](/blog/a-taste-of-ruby).



## Tower of Hanoi -- With A Real Object


The first two examples were not really object-based.  They are purely procedural, the algorithm is expressed completely recursively, and execution state is maintained on the call stack.  This works fine if we are just 'tracing' the algorithm, but it does not let us show the state of the towers at any given time.  We also don't see much in the way of real objects.  By adding the state of the towers in, we now have a reason to have stateful objects and new Classes of objects to maintain that state.  The added code with some explanation is in Chapter 4, Page 45.

<!-- more -->

```ruby
# A Taste of Ruby.  Based on a Taste of Smalltalk (Kaehler and Patterson)
# Tower of Hanoi -- Variation 3

# TowerOfHanoi
#
# @stacks is an Array of DiskHolders (pins)
class TowerOfHanoi < Object
  #Tower of Hanoi program.
  def hanoi
    # Do the tower
    puts "How tall a tower?"
    print ">"
    height = gets.to_i

    @stacks = Array.new(3).collect { Array.new }
    firstStack = @stacks[0]
    1.upto(height) {|i| firstStack.unshift("Disk-#{(?A+i-1).chr}") }

    move_tower(height,1,3,2)
  end

  # Recursive procedure to move the disk at a height
  # from one pin to another pin using a third pin
  def move_tower(height, fromPin, toPin, usingPin)
    if (height>0) then
      move_tower(height-1, fromPin, usingPin, toPin)
      move_disk(fromPin, toPin)
      move_tower(height-1, usingPin, toPin, fromPin)
    end
  end

  # Actually move the disk between the pins
  def move_disk(fromPin,toPin)
    disk = (@stacks[fromPin-1]).pop
    @stacks[toPin-1].push(disk)
    print disk, " moved ", fromPin.to_s, "->", toPin.to_s, "\n"
  end

end

TowerOfHanoi.new.hanoi
```

So we moved to adding state to an object by simply referencing an instance variable (notationally via the '@').  Again, I find that unpleasant and would rather the compiler (and human reader) was told a bit more formally what instance variables exist.  In quickly typing the above, I made a typo ("@stack") that caused a runtime error and should have been easily detected at compile time.

Comparing with Smalltalk, the similarities are extremely high again.  Basically the core method calls and flow (e.g. the bytecodes) would look almost identical and it is only syntactic differences (built-in keywords in Ruby, shortening of selectors/method-names, etc.) that make the two read differently.  The idioms can be maintained (1.upto(height)) during translation and both versions are natural for their own language.  There is no paradigm change, impedance mismatch, etc.



### Where Next?


Next we should go to Chapter 5 (Animating the Program) — Page 64.  This involves some graphic work for the 'model' aspects to be realistic, so I guess I will go into the Tk toolkit or Shoes as among the easiest Ruby graphic extensions to use as an example.  If anyone has suggestions for other ones (meant to be super simple), please let me know.
