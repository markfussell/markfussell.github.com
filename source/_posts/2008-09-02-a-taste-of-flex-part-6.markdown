---
comments: true
date: 2008-09-02 16:46
layout: post
slug: a-taste-of-flex-part-6
title: A Taste of Flex (Part 6)
wordpress_id: 130
categories:
- Flex
- Java
- Ruby
- RubyShoes
- Smalltalk
---



This is a multi-part series.  The first in the series is [here](/blog/a-taste-of-flex).



## Tower of Hanoi &mdash; With Rule Disks (and no stack)


The first passes at the Tower of Hanoi algorithm were all done with the algorithm being a recursive call on the stack.  This is 'true' as an algorithm, but comes off as a bit unnatural for humans.  But it also has a problem in that the algorithm can not be 'suspended' in mid activity unless the language allows a feature (called a continuation) that can suspend the stack itself.  For a multi-threaded language, this inability to suspend may not be a problem because you could create one thread for the algorithm and another thread to listen for whenever a new (interesting) change in state occurs.  This is how the RubyShoes version works: the algorithm is in a new thread separate from the GUI thread (otherwise things behave badly).  Similarly, the Smalltalk-80 GUI is being drawn in a different thread than the main execution thread.  But in many circumstances even this multi-thread version would not work: say you have a client-server version (e.g. Rails) or want to pause/suspend the execution of the 100-tall tower of hanoi.  And finally, if you only have a single-threaded language like Flex, things just don't work at all for intermediate renderings unless you can make the algorithm not be dependent on the call stack.

<!-- more -->



### Tower of Hanoi — With Rule Disks (and no stack) -- Flex


I think it is easiest to work up from the Disk perspective, although see the book for a different flow.  The main changes to HanoiDisk are to have it be able to figure out whether it has any legal moves, and what the bestMove (next move) would be.  To do each of these, the disk needs to communicate back with the towers.  In the Smalltalk code this was done through a Class Variable, but it is far simpler and more scalable to do this through instance variables: each Disk needs to know what tower it belongs to.



#### HanoiDiskRules


```actionscript
package {

public class HanoiDiskRules extends HanoiDisk {
    protected var my_previousPole : Number;
    protected var my_towers : AnimatedRulesTowerOfHanoi;

    public function setupTowers(towers : AnimatedRulesTowerOfHanoi) : void {
        my_towers = towers;
    }

    public function hasLegalMove() : Boolean {
        var otherTops : Array = my_towers.selectPolesOtherThan(this);
        for (var i:int = 0; i<otherTops.length; i++) {
            var eachTopDisk : HanoiDisk = otherTops[i];
            if (eachTopDisk.width > this.width) return true;
        }
        return false;
    }

    public function bestMove() : HanoiDisk {
        var otherTops : Array = my_towers.selectPolesOtherThan(this);
        for (var i:int = 0; i<otherTops.length; i++) {
            var eachTopDisk : HanoiDisk = otherTops[i];
            if ( (eachTopDisk.width > this.width) && (eachTopDisk.pole != my_previousPole) ) return eachTopDisk;
        }
        return null;
    }

    public override function moveUpon(destination : HanoiDisk) : void {
        my_previousPole = pole;

        super.moveUpon(destination);
    }

}

} //package
```

Compared to the Smalltalk version of this, the main difference is we are creating Array objects instead of passing a function into an iterator.  Again this seems the more natural for Flex... but... it is starting to get annoying looking and has real performance impact, so later we should try to do an iterator + function-based version.



#### AnimatedRulesTowerOfHanoi (No Animation)


The change to the Tower code for Flex is two parts:




  * Provide methods that support the rules that the smarter HanoiDiskRules has


  * Drive the algorithm in a way to enable Flex to render the intermediary results


We can do these in two steps, with the current step focused on just the algorithm changes

```actionscript
package {

public class AnimatedRulesTowerOfHanoi extends AnimatedTowerOfHanoi {
    protected var my_oldDisk : HanoiDisk;
    protected var my_currentDisk : HanoiDisk;
    protected var my_destinationDisk : HanoiDisk;

    protected override function moveTower(height : int, fromPin : *, toPin : *, usingPin : *) : void {
        //Now that the disks know all the rules... we can ignore all the arguments!

        while (!doNextMove_IsDone) {};
    }

    protected function doNextMove_IsDone() : Boolean {
        my_currentDisk = this.decide();

        (my_stacks[my_currentDisk.pole-1] as Array).pop();
        (my_stacks[my_destinationDisk.pole-1] as Array).push(my_currentDisk);
        my_currentDisk.moveUpon(my_destinationDisk);

        my_oldDisk = my_currentDisk;

        my_view.noteChange();

        return isAllOnOneTower();
    }

    protected function isAllOnOneTower() : Boolean {
        for (var i:int = 0; i<my_stacks.length; i++) {
            var eachStack : Array = my_stacks[i];
            if (eachStack.length == my_height) return true;
        }
        return false;
    }

    public function selectTopsOtherThan(disk : HanoiDisk) : Array {
        var result : Array = new Array();
        for (var i:int = 0; i<my_stacks.length; i++) {
            var eachStack : Array = my_stacks[i];

            if (eachStack.length == 0) continue;

            var topDisk : HanoiDisk = eachStack[eachStack.length-1];
            if (topDisk !== disk) {
                result.push(topDisk);
            }
        }
        return result;
    }

    public function selectPolesOtherThan(disk : HanoiDisk) : Array {
        var result : Array = new Array();
        for (var i:int = 0; i<my_stacks.length; i++) {
            var eachStack : Array = my_stacks[i];

            if (i == disk.pole-1) continue;

            if (eachStack.length == 0)  {
                result.push(my_mockDisks[i]);
            } else {
                var topDisk : HanoiDisk = eachStack[eachStack.length-1];
                result.push(topDisk);
            }
        }
        return result;
    }

    protected function decide() : HanoiDisk {
        var tops : Array = selectTopsOtherThan(my_oldDisk);
        for (var i:int = 0; i<tops.length; i++) {
            var movingDisk : HanoiDiskRules = tops[i];
            if (movingDisk.hasLegalMove()) {
                my_destinationDisk = movingDisk.bestMove();

                return movingDisk;
            }
        }
        //This should never happen
        return null;
    }

    protected override function createDisk() : HanoiDisk {
        var result : HanoiDiskRules = new HanoiDiskRules();
        result.setupTowers(this);
        return result;
    }
}

} //package
```

As mentioned in HanoiDiskRules section, the main annoyance of this particular implementation of the new algorithm compared to Smalltalk is having to create intermediate array objects just to communicate between the Disk and the Tower.

But things should work again.  Except we will still get only one rendering because the whole algorithm is executed in one call within a "while" loop.



### Tower of Hanoi — With Rule Disks and Correct Animation -- Flex


To make things work in Flex now, all we do is have to unwrap the immediacy [single call stack] of the while loop.  Instead of calling each 'doNextMove' immediately, we will wait for an event.  That event could be anything, like clicking a button or the server sending a response.  Each time we get an event, we will do the next step.  To be easy to present and view, we will make the events be driven by a simple timer.  Each time the timer fires an event, we will do the next step.  Repeating this until we are done.

The only method changes are to 'moveTower' and to 'handleTimer'.  Everything else works just the way it was.



#### AnimatedRulesTowerOfHanoi (No Animation)



```actionscript
package {
    import flash.events.TimerEvent;
    import flash.utils.Timer;


public class AnimatedRulesTowerOfHanoi extends AnimatedTowerOfHanoi {
    protected var my_oldDisk : HanoiDisk;
    protected var my_currentDisk : HanoiDisk;
    protected var my_destinationDisk : HanoiDisk;

    protected var my_timer : Timer;

    protected override function moveTower(height : int, fromPin : *, toPin : *, usingPin : *) : void {
        //Now that the disks know all the rules... we can ignore all the arguments!

        my_timer = new Timer(300)
        my_timer.addEventListener(TimerEvent.TIMER, handleTimer);
        my_timer.start();

    }

    public function handleTimer(evt:TimerEvent):void {
        var isDone : Boolean = doNextMove_IsDone();

        if (isDone) {
            my_timer.stop();
            my_timer = null;
        }
    }




    protected function doNextMove_IsDone() : Boolean {
        my_currentDisk = this.decide();

        (my_stacks[my_currentDisk.pole-1] as Array).pop();
        (my_stacks[my_destinationDisk.pole-1] as Array).push(my_currentDisk);
        my_currentDisk.moveUpon(my_destinationDisk);

        my_oldDisk = my_currentDisk;

        my_view.noteChange();

        return isAllOnOneTower();
    }

    protected function isAllOnOneTower() : Boolean {
        for (var i:int = 0; i<my_stacks.length; i++) {
            var eachStack : Array = my_stacks[i];
            if (eachStack.length == my_height) return true;
        }
        return false;
    }

    public function selectTopsOtherThan(disk : HanoiDisk) : Array {
        var result : Array = new Array();
        for (var i:int = 0; i<my_stacks.length; i++) {
            var eachStack : Array = my_stacks[i];

            if (eachStack.length == 0) continue;

            var topDisk : HanoiDisk = eachStack[eachStack.length-1];
            if (topDisk !== disk) {
                result.push(topDisk);
            }
        }
        return result;
    }

    public function selectPolesOtherThan(disk : HanoiDisk) : Array {
        var result : Array = new Array();
        for (var i:int = 0; i<my_stacks.length; i++) {
            var eachStack : Array = my_stacks[i];

            if (i == disk.pole-1) continue;

            if (eachStack.length == 0)  {
                result.push(my_mockDisks[i]);
            } else {
                var topDisk : HanoiDisk = eachStack[eachStack.length-1];
                result.push(topDisk);
            }
        }
        return result;
    }

    protected function decide() : HanoiDisk {
        var tops : Array = selectTopsOtherThan(my_oldDisk);
        for (var i:int = 0; i<tops.length; i++) {
            var movingDisk : HanoiDiskRules = tops[i];
            if (movingDisk.hasLegalMove()) {
                my_destinationDisk = movingDisk.bestMove();

                return movingDisk;
            }
        }
        //This should never happen
        return null;
    }

    protected override function createDisk() : HanoiDisk {
        var result : HanoiDiskRules = new HanoiDiskRules();
        result.setupTowers(this);
        return result;
    }
}

} //package
```



