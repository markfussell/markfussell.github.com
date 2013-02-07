---
comments: true
date: 2008-09-02 07:13
layout: post
slug: a-taste-of-flex-part-4
title: A Taste of Flex (Part 4)
wordpress_id: 112
categories:
- Flex
- Ruby
- Smalltalk
---



This is a multi-part series.  The first in the series is [here](../a-taste-of-flex).



## Tower of Hanoi &mdash; With Model Objects


The book "A Taste of Smalltalk" jumps from showing a single 'TowerOfHanoi' object to both adding in Disk model objects and putting on a graphical representation of those Disk objects.  Even though Flex has no issue with graphics, I want to separate these two changes both to match Ruby and because the changes are logically somewhat separate.  So first we can put in Disk objects and study that change.  After that, we can go into the GUI code.

<!-- more -->



### The Code

```actionscript
package {

/**
 * A ModeledTowerOfHanoi is the TowerOfHanoi algorithm
 * but it keeps track of the state of the Disks
 * with actual objects.  This makes it suitable for
 * model-based behavior (say tracking moves, animation or
 * other event listening) on top of those Disks
 */
public class ModeledTowerOfHanoi extends TowerOfHanoi{
    protected var my_height : Number;
    protected var my_mockDisks : Array;

    public override function doHanoi(height : int) : void {
        my_height = height;

        setupDisks();

        logToConsole("\n\nStart\n");
        printStacks();

        logToConsole("\n\nMoves\n");
        moveTower(my_height, 1, 3, 2);

        logToConsole("\n\nResult\n");
        printStacks();
    }

    protected function setupDisks() : void {
        my_stacks = new Array();
        for (var i:int = 0; i<3; i++) {
            my_stacks.push(new Array());
        }

        var firstStack : Array = my_stacks.first(); //[0];
        for (var j:int = 0; j<my_height; j++) {
            firstStack.unshift(new HanoiDisk().initWidth_pole(j,1));
        }

        my_mockDisks = new Array();
        for (var k:int = 0; k<3; k++) {
            my_mockDisks[k]=new HanoiDisk().initWidth_pole(1000,k);
        }
    }

    protected function printStacks() : void {
        for (var i:int = 0; i<my_stacks.length; i++) {
            var eachStack : Array = my_stacks[i];
            logToConsole("   <stack>\n");
            for (var j:int = eachStack.length-1; j>=0; j--) {
                var eachDisk : * = eachStack[j];
                logToConsole(eachDisk.toString()+"\n");
            }
            logToConsole("   </stack>\n");
        }
    }


    protected override function moveDisk(fromPin : *, toPin : *) : void {
        var supportDisk = (my_stacks[toPin-1].length == 0) ? my_mockDisks[toPin-1] : my_stacks[toPin-1][0];

        var disk : HanoiDisk = my_stacks[fromPin-1].pop();
        my_stacks[toPin-1].push(disk);

        disk.moveUpon(supportDisk);

        logToConsole(disk + " moved " + fromPin + "->" + toPin+"\n");
    }

    public function logToConsole(string : String) {
        my_view.logToConsole(string);
    }

}

} //package
```

#### HanoiDisk
```actionscript
package {

public class HanoiDisk {
    protected const CONST_asciiA : Number = "A".charCodeAt();

    public var width : Number;
    public var pole : Number;

    protected var my_name : String;
    protected var my_moveCount : Number = 0;

    public function initWidth_pole(aWidth : Number, aPole : Number) : HanoiDisk {
        width = aWidth;
        pole = aPole;

        if (width < 1000) {
            my_name = "Disk-#"+String.fromCharCode(CONST_asciiA+width);
        } else {
            my_name = "Base-#"+pole;
        }

        return this;
    }


    public function toString() : String {
        return my_name+"("+my_moveCount+")";
    }

    public function moveUpon(destination : HanoiDisk) : void {
        pole = destination.pole;
        my_moveCount++;
    }

}

} //package
```


#### ModeledTowerOfHanoi



I believe the following are the most interesting aspects:




  1. Public Attributes (which may be wrapped or unwrapped instance variables)


  2. Explicit 'override'


  3. Explicit return values





### Public Attributes


Flex allows you to have attributes that are either (1) just instance variables or (2) one or a pair of 'set' and 'get' accessors.  The notation of access looks the same, so it is up to the receiver/implementer to decide whether 'wrapping' is necessary.  Although the Macro approach of Ruby is nicer than having to do add accessors manually (or even 'refactor' them in with a tool), the Flex approach is by far the best for this kind of situation.  The language/compiler actually understands the intent of the programmer (declaratively) vs. running through a macro and the results just happen to be the same.

Note that Flex goes beyond simple public attributes and can attach notification capabilities on top of it.



### Explicit 'override'


Flex and ActionScript again clearly try to help the programmer avoid making a mistake, in this case by making sure code _knows_ it is overriding inherited code and that within that override you have to consider what the superclass wants, whether you need to call 'super', etc.



### Explicit return values


Flex enables you to avoid specifying the type of a return value (defaulting to 'Any'), but you have to decide whether you are going to return anything or nothing at all.  And if you are going to return anything, you need to explicitly say what you are going to return.

As I mentions in Part 4 of a Taste of Ruby, this approach is clearly the best -- actually better than what I mentioned there.  So then returning 'null' by default becomes a good second best (no language so far mentioned),  Returning 'self' by default is third best (Smalltalk).  And Ruby returning of "whatever happens to be returned by the last statement" is a distant fourth.



### What Next


Now onto the GUI itself.


