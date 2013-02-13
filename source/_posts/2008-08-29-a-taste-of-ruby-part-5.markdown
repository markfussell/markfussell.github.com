---
comments: true
date: 2008-08-29 23:42
layout: post
slug: a-taste-of-ruby-part-5
title: A Taste of Ruby (Part 5)
wordpress_id: 64
categories:
- Dynamic
- Ruby
- RubyShoes
- Smalltalk
---

This is a multi-part series.  The first in the series is [here](/blog/a-taste-of-ruby).



## Tower of Hanoi — With Graphics


To follow the flow of code progress within "A Taste of Smalltalk", we next need to include a graphical representation of the disks and their movement between the poles.  As mentioned previously, converting this part to Ruby has a problem: Ruby doesn't include graphic capabilities.  So we have to pick an extension or add-on to Ruby that will enable us to hook into the graphics system.

<!-- more -->

Because it seems the easiest to work with, I chose 'Shoes' ([http://code.whytheluckystiff.net/shoes/](http://code.whytheluckystiff.net/shoes/)).  The installation of Shoes is very simple and you just run your 'app' inside Shoes application world, which includes Ruby itself, the graphic capabilities, and the UI framework.

Shoes seems simple and clean, the only problem is it doesn't work within an IDE, so I lost a bit of tooling while doing this.  With the final running program, this isn't a problem but it can be painful if you are learning, tweaking, or studying things.  So because this could be painful to others as well, I made the main code base able to run inside a normal irb, and in that case it just logs to the console like it always has been.

The main new class is AnimatedTowerOfHanoi, which is really just a notifying version of ModeledTowerOfHanoi.  This pass also has some cleanup of the previous code of ModeledTowerOfHanoi.  So I include all three classes below



### The New Classes




#### TowerOfHanoi 


This is still the same as the original from 'Part 2'

```ruby
# A Taste of Ruby.  Based on a Taste of Smalltalk (Kaehler and Patterson)
# Tower of Hanoi -- Variation 5

# TowerOfHanoi
#
# @stacks is an Array of DiskHolders (pins)
class TowerOfHanoi < Object
  #Tower of Hanoi program.
  def hanoi
    # Do the tower
    puts "How tall a tower?"
    print ">"
    height = 5 #gets.to_i

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
```



#### ModeledTowerOfHanoi


The ModeledTowerOfHanoi has been cleaned up a little to make subclassing easier and just because it was the right thing to do.  Specifically,



	
  * There was a fix to the 'move' algorithm, so we tell the disk onto which disk it is moving

	
  * Some common methods are pulled out

	
  * Disks are now responsible for describing the move





```ruby
# A ModeledTowerOfHanoi is the TowerOfHanoi algorithm
# but it actually keeps track of the state of the Disks
# with actual objects.  This makes it suitable for
# model-based behavior (say tracking moves, animation or
# other event listening) on top of those Disks

class ModeledTowerOfHanoi < TowerOfHanoi
  def hanoi
    ask_for_height

    setup_disks

    puts "Start"
    print_stacks

    puts
    move_tower(@height,1,3,2)
    puts

    puts "Result"
    print_stacks
  end

  def print_stacks
    @stacks.each do | eachStack |
      puts "  "
      puts eachStack.reverse
      puts "  "
    end
  end

  def ask_for_height
    puts "How tall a tower?"
    print ">"
    height = 5 #gets.to_i
    puts ""
    @height = height
  end

  def setup_disks
    HanoiDisk.set_towers(self)

    @stacks = Array.new(3).collect { Array.new }
    firstStack = @stacks[0]
    1.upto(@height) do |size|
      disk = createDisk.initWidth_pole(size,1)
      firstStack.unshift(disk)
    end

    @mockDisks = Array.new(3)
    1.upto(3) do |i|
      @mockDisks[i-1]=(createDisk.initWidth_pole(1000,i))
    end
  end

  def createDisk()
    return HanoiDisk.new
  end

  def move_disk(fromPin, toPin)

    supportDisk = if (@stacks[toPin-1].empty?) then
      @mockDisks[toPin-1]
    else
      @stacks[toPin-1].last
    end

    disk = @stacks[fromPin-1].pop
    @stacks[toPin-1].push(disk)
    disk.move_upon(supportDisk)


  end

  def log(outString)
    print outString, "\n"
  end
end

class HanoiDisk
  attr_reader :name
  attr_reader :pole
  attr_reader :width

  def self.set_towers(owner)
    @@the_towers = owner
    @@the_thickness = 14
    @@the_diskgap = 2
  end

  def initWidth_pole(width, pole)
    @pole = pole
    @width = width

    if (width < 1000) then
      @name="Disk-#{(?A+width-1).chr}"
    else
      @name="Base-#{pole}"
    end

    @move_count = 0

    return self
  end

  def to_s
    return "#{@name}(#{@pole},#{@move_count})" #{@@the_towers}
  end

  def move_upon(destination)
    @@the_towers.log( "#{self} moved #{@pole.to_s} -> #{destination.pole.to_s}  (#{destination.to_s})" )

    @pole = destination.pole
    @move_count += 1
  end
end

```


#### AnimatedTowerOfHanoi 


Now we are finally to the new "animated" class.  The main changes for this class are to enable it to talk to an owning 'application' if it exists and also it starts hooking into the Shoes code.  For convenience, I made the behavior of the class branch based on whether that '@app' exists.  The final change is that AnimatedHanoiDisk does a sleep so we can see the animation.  This affects both console and graphic behavior.

```ruby
class AnimatedTowerOfHanoi < ModeledTowerOfHanoi
  attr_reader :stacks

  def initApp(shoesApp)
    @app = shoesApp

    return self
  end

  def setup_disks
    super

    @app.noteChange if @app
  end

  def createDisk()
    return AnimatedHanoiDisk.new
  end

  def ask_for_height
    @height = 5
    return nil unless @app
    answer ask("How Tall A Tower?")
  end

  def answer(v)
    @height = v.to_i
    @app.answer(v)
  end

  def log(outString)
    super unless @app
    @app.appendLog(outString) if @app
  end

  def move_disk(fromPin, toPin)
    super

    @app.noteChange if @app
  end
end

class AnimatedHanoiDisk < HanoiDisk
  def move_upon(destination)
    super(destination)

    sleep 0.3
    #@@the_towers.log("Foo")
    #@@the_towers.draw_board
  end

end
```


### Running via 'irb'


You can test the above code by running it in the normal Ruby console
```ruby
AnimatedTowerOfHanoi.new.hanoi
```



### Putting on the Shoes


If you have Shoes running successfully, you can combine the above code with a Shoes app which visualizes the state of the towers.  The Shoes code is:
```ruby
Shoes.app :width => 520, :height => 600, :resizable => false do

  def answer(v)
    @answer.replace "Doing a #{v} story Hanoi"
  end

  def appendLog(outString)
    @log.append { para outString }
  end

  def noteChange
    redrawStacks
  end

  #redraw the stacks by simply clearing the area and drawing them again
  def redrawStacks

    @render_area.clear do
      @towerOfHanoi.stacks.each_with_index do |eachStack,i|
        poleCenterX = i*100 + 50

        eachStack.each_with_index do |eachDisk,j|
          height = 180 - (j * 15)
          width = eachDisk.width * 10

          fill "#A00"
          rect(poleCenterX - width/2 , height, width, 10)
          j += 1
        end
        i += 1
      end
    end
  end

  #=============================================

  @hanoiArea = stack do
    @answer = para "Answers appear here"

    @render_area = flow :width => 300, :height => 200 do
      background "#999"
    end
  end

  @log = stack

  @towerOfHanoi = AnimatedTowerOfHanoi.new.initApp(self)
  Thread.new do
    @towerOfHanoi.hanoi
  end
end
```

The Shoes app has the main drawing area at the top and then a logging area beneath it.  The HanoiStacks are cleared and redrawn any time they change.  The original Smalltalk code used deltas (disks were moved individually) but that didn't seem critical to match, and partially the code is relying on being on a black-and-white screen.



### Comments


Not having a standard Graphics capability is not surprising or a problem for a scripting language (limited in context of usage) but is quite a problem for a language that wants to be mainstream.  There is a similar problem on the server-side with a standard (off-screen) image processing.  Shoes and RMagick and the like are trying to fill these holes but Ruby has been around for a long time for these aspects to not be addressed and part of the standard.

In terms of the resulting code with Shoes, the Ruby version is certainly nice and clean.



### Where Next?


The last code-changing chapter deals with making the Algorithm a little more human-natural.  If people are interested, I may do that change too.  But comparatively it is a pretty minor change and does not bring out any interesting language aspects.






### 
