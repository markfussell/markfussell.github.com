---
comments: true
date: 2008-09-02 05:11
layout: post
slug: a-taste-of-flex-part-3
title: A Taste of Flex (Part 3)
wordpress_id: 99
categories:
- Flex
- Ruby
- Smalltalk
---



This is a multi-part series.  The first in the series is [here](/blog/a-taste-of-flex).



## Tower of Hanoi -- With A Real Object


The first two examples were not really object-based.  They are purely procedural, the algorithm is expressed completely recursively, and execution state is maintained on the call stack.  This works fine if we are just 'tracing' the algorithm, but it does not let us show the state of the towers at any given time.  We also don't see much in the way of real objects.  By adding the state of the towers in, we now have a reason to have stateful objects and new Classes of objects to maintain that state.  The added code with some explanation is in Chapter 4, Page 45.

<!-- more -->



```actionscript
package {

public class TowerOfHanoi {
    protected const CONST_asciiA : Number = "A".charCodeAt();

    protected var my_view : *;
    protected var my_stacks : Array;

    public function initView(view : *) : void {
        my_view = view;
    }

    public function doHanoi(height : int) : void {
        my_stacks = new Array();
        for (var i:int = 0; i<3; i++) {
            my_stacks.push(new Array());
        }

        var firstStack : Array = my_stacks[0];
        for (var j:int = 0; j<height; j++) {
            firstStack.unshift("Disk-#"+String.fromCharCode(CONST_asciiA+j));
        }

        moveTower(height, 1, 3, 2);
    }


    protected function moveTower(height : int, fromPin : *, toPin : *, usingPin : *) : void {
        if (height > 0) {
            moveTower(height-1, fromPin, usingPin, toPin);
            moveDisk(fromPin, toPin);
            moveTower(height-1, usingPin, toPin, fromPin);
        }
    }

    protected function moveDisk(fromPin : *, toPin : *) : void {
        var disk = my_stacks[fromPin-1].pop();
        my_stacks[toPin-1].push(disk);

        my_view.logToConsole(disk + " moved " + fromPin + "->" + toPin+"\n");
    }

}

} //package
```



### TowerOfHanoi


The TowerOfHanoi main class is now able to be a pure ActionScript class, so it looks much cleaner and formats nicely now.  The only additional feature it needs (compared to say the Smalltalk or Ruby version) is a reference to its View so it can print into the Console of that view.  I could actually just use 'Application.application' to get the outer view, but this is cleaner, more scalable, and makes it clearer what is happening [even though many people like to cheat with those kinds of Globals].




### Main Application


The main Application is now clean of model code and just (1) holds onto the outer model and (2) provides view services to others.  I made logToConsole public so people could identify what is meant to be used by others vs. what is not.  Ultimately 'logToConsole' could be put into an interface to make it clearer that is all the 'TowerOfHanoi' needs from its view.

```xml
<?xml version="1.0" encoding="utf-8"?>
<mx:Application xmlns:mx="http://www.adobe.com/2006/mxml" layout="vertical"
	creationComplete="handleCreationComplete()"
>
<mx:Script>
    <![CDATA[
        var my_model : TowerOfHanoi;

        function handleCreationComplete() : void {
            my_model = new TowerOfHanoi();
            my_model.initView(this);
        }

        function handleDoItClicked() : void {
            logToConsole("\n=== Doing a Hanoi tower "+vTowerInput.text+" tall\n");
            my_model.doHanoi(new Number(vTowerInput.text));
        }

        public function logToConsole(string : String) : void {
            vConsole.text = vConsole.text+string;
        }
    ]]]]><![CDATA[>
</mx:Script>
<mx:HBox>
    <mx:Text text="How tall a tower?" />
    <mx:TextInput id="vTowerInput" />
    <mx:Button label="Do It" click="handleDoItClicked()" />
</mx:HBox>
<mx:TextArea id="vConsole" width="100%" height="100%" />
</mx:Application>
```


## Comments


By moving the ActionScript portion out of the main Application XML, the syntax becomes clearer.  Another feature of Flex that is starting to show is that we can document and compile-time verify visibility (as needed), contractual interfaces (as needed), etc.  But we can also leave them off and things work fine.

We do need to document the existence of instance variables, local variables, and methods of whatever types we use (e.g. 'self' / 'this') or the compiler will complain.  I believe this is far better (most of the time) than leaving that as a pure runtime issue and having typos potentially take significant time to debug.  And this does not interfere with things being 'dynamic' ('duck typed') in most cases although it is related to some meta-language capabilities.

Going into the Flex code itself, we can see a stronger relationship to 'C' style control structures and not the higher-level block/closure structuring of things.  Although iterators and Functions are available in Flex/AS, they are not as natural in the language as in Smalltalk or Ruby because they don't work well with collections (as defined in the current API) and are just a bit too verbose in general.  You could modify (monkey-patch) collections, but then the compiler would not understand the new functionality.    Although that is no worse than in Java/Ruby, it would be unusual in the community.  Maybe if Adobe adds it to AS4 just to be cross-language similar, it would be more natural.  But I didn't want to augment the existing classes just to have the language match another language's idioms

Examples of this kind of Monkey-Patch are:




  * [RocketBoots](http://www.rocketboots.com.au/blog/index.cfm?mode=entry&entry=A5C356B1-E081-51EF-A79F47A1601ADEDF)


  * [Kirupa](http://www.kirupa.com/forum/archive/index.php/t-246045.html)



A minor note, the CharCode conversion is actually a Unicode conversion.  So you can put pretty much any character in the world languages you want to in there :-)  For example, try it with '50023' for the CONST value.



### Where Next?


Next we should go to Chapter 5 (Animating the Program) — Page 64.  This involves some graphic work for the 'model' aspects to be realistic, and for Flex this is not at all difficult, but I will follow the same pattern of 'Parts' that I did with Ruby, so the next Part is just changing to having real Domain Model Objects (vs. the UI Model separation that occurred above for TowerOfHanoi vs the Application)

