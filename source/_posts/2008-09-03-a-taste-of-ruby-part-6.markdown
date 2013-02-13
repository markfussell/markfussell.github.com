---
comments: true
date: 2008-09-03 02:18
layout: post
slug: a-taste-of-ruby-part-6
title: A Taste of Ruby (Part 6)
wordpress_id: 138
categories:
- Flex
- Rails
- Ruby
- Smalltalk
---

This is a multi-part series.  The first in the series is [here](/blog/a-taste-of-ruby).



## Tower of Hanoi — With Rule Disks (and no stack)


The first passes at the Tower of Hanoi algorithm were all done with the algorithm being a recursive call on the stack.  This is 'true' as an algorithm, but comes off as a bit unnatural for humans.  

<!-- more -->

But it also has a problem in that the algorithm can not be 'suspended' in mid activity unless the language allows a feature (called a continuation) that can suspend the stack itself.  Although Ruby is multi-threaded and running two threads works in RubyShoes, Rails does not work either with a stateful server (and multiple threads) or with continuations.  At the end of each controller dispatch the session or the database has to have persisted all state.  So we need to do something like formalizing the state of the Disks and Towers to make the Tower of Hanoi work with Rails.

Also, the Smalltalk code in this section was among the most 'closure-oriented', so this helps show how that maps to Ruby.  




### You aren't capitalizing right?


Before continuing into the code, people should realize I actively object to Ruby's choice to have only one 'word' delimiter by convention.  See my article [Ruby Naming Convention Failure](/blog/ruby-naming-convention-failure) for details and the naming conventions I use, especially for methods.  But the quick summary is:



	
  * I use CamelCase for combining words into a 'part' of a method name or variable name

	
  * I use an underscore as a placeholder for the parameters of the method

	
  * I use an underscore to break up a phrase into logical parts



If Ruby supported some other word delimiter than underscore (say hyphen "-" or ":"), I would be agnostic to dropping the CamelCase, but without that second word delimiter, dropping CamelCase is a losing proposition in expressiveness.  In a team setting we might have to discuss this, but in this case this is my code alone... and my standard actually makes it easier to convert to-from Smalltalk, Ruby, Flex, Java, etc.  The standard works cleanly with all of them.



### Tower... 


To migrate to the Rules version of the HanoiTower, it is easiest to start with the Disks themselves.



#### RulesHanoiDisk


A Rules based HanoiDisk just needs to be able to say "Can I move somewhere" and "If so, where".  It does this in collaboration with the Tower.  "@tower" is now an instance variable that every Disk has set when it is created.  By using an instance variable, it is cleaner than using class variables (we can have multiple towers at the same time) and it works under Ruby serialization/marshalling.  

```ruby
class RulesHanoiDisk < AnimatedHanoiDisk
  def hasLegalMove() 
    @towers.forPolesOtherThan_do(self) do | eachTopDisk |
      if (eachTopDisk.width  > self.width) then return true end
    end
    return false
  end

  def bestMove()
    @towers.forPolesOtherThan_do(self) do | eachTopDisk |
      if ( (eachTopDisk.width > self.width) && (eachTopDisk.pole != @previousPole) ) then return eachTopDisk end
    end
    return nil
  end

  def move_upon(destination)
    @previousPole = @pole

    super(destination)
  end

end
```

Among the interesting things with the new RulesHanoiDisk is the extensive use of Blocks.  By using Blocks, we can iterate over the collection of stacks without creating new collections.  The Disk and the Tower collaborate to create custom iterators with minimal code.  See RulesTowerOfHanoi for the example.



#### RulesTowerOfHanoi 


```ruby
class RulesTowerOfHanoi < AnimatedTowerOfHanoi
  def initHeight(height)
    @height = height

    return self
  end

  def move_tower(height, fromPin, toPin, usingPin)
    while (!doNextMove_IsDone()) do end
  end
  
  def createDisk()
    result = RulesHanoiDisk.new
    result.setupTowers(self)
    
    return result
  end

  def doNextMove_IsDone()
    @currentDisk = decide()
    
    @stacks[@currentDisk.pole-1].pop()
    @stacks[@destinationDisk.pole-1].push(@currentDisk)
    @currentDisk.move_upon(@destinationDisk)
    
    @oldDisk = @currentDisk
    
    @app.noteChange if @app

    return isAllOnOneTower()
  end

  def isAllOnOneTower()
    foundStack = @stacks.detect { |eachStack|  eachStack.size == @height }
    return foundStack != nil
  end

  def forTopsOtherThan_do(disk)
    @stacks.each do | eachStack |
      if (eachStack.empty?) then next end

      topDisk = eachStack.last
      if (topDisk == disk) then next end

      yield topDisk
    end
  end
  
  def forPolesOtherThan_do(disk)
    @stacks.each_with_index do | eachStack, i |
      if (i == disk.pole-1) then next end;

      topDisk = if (eachStack.empty?) then @mockDisks[i] else eachStack.last end;

      yield topDisk
    end
  end

  def decide
    forTopsOtherThan_do(@oldDisk) do | eachDisk |
      if (eachDisk.hasLegalMove) then
        @destinationDisk = eachDisk.bestMove

        return eachDisk
      end
    end
  end
  
end

```