---
comments: true
date: 2008-09-02 07:44
layout: post
slug: a-taste-of-flex-part-5
title: A Taste of Flex (Part 5)
wordpress_id: 120
categories:
- Flex
- Ruby
- RubyShoes
- Smalltalk
---

This is a multi-part series.  The first in the series is [here](/blog/a-taste-of-flex).



## Tower of Hanoi — With Graphics


To follow the flow of code progress within "A Taste of Smalltalk", we next need to include a graphical representation of the disks and their movement between the poles.  Flex lives inside a Flash player (or app), so it is inherently capable of doing sophisticated graphics.  Actually it is hard to 'contain' yourself to just doing the simplest possible thing when animating sprites around the screen is very easy.  But to try to compare Flex to Ruby and Smalltalk, we want to keep things in about the same ballpark -- and again study how different languages work with the same problem.  

<!-- more -->



### Tower of Hanoi — With Graphics -- Flex


Given our Flex version is already in a graphics environment, and the model (TowerOfHanoi) is already aware of having to talk the View, the changes are relatively minimal to get things working.  We need to render the disks in the Application/View and we need to notify the view whenever the model changes.



#### Main Application


To add rendering in, we just create a render canvas area and then create the UIComponents (shapes) on each redraw.  Normally we would be more likely to animate a single set of disks (vs. destroying them and recreating them), which would be both better in performance and be _much cooler_, but this matches the RubyShoes version better.

```xml
<?xml version="1.0" encoding="utf-8"?>
<mx:Application xmlns:mx="http://www.adobe.com/2006/mxml" layout="vertical"
	creationComplete="handleCreationComplete()"
>
<mx:Script>
    <![CDATA[
        import mx.core.UIComponent;
        var my_model : AnimatedTowerOfHanoi;

        function handleCreationComplete() : void {
            Array.prototype.first = function():Object { return this[0]; }

            my_model = new AnimatedTowerOfHanoi();
            my_model.initView(this);
        }

        function handleDoItClicked() : void {
            logToConsole("\n=== Doing a Hanoi tower "+vTowerInput.text+" tall\n");
            my_model.doHanoi(new Number(vTowerInput.text));
        }

        public function logToConsole(string : String) : void {
            vConsole.text = vConsole.text+string;
        }

        public function noteChange() : void {
            vRenderArea.removeAllChildren();

            var stacks : Array = my_model.getStacks();
            for (var i : int = 0; i<stacks.length; i++) {
                var eachStack : Array = stacks[i];
                var poleCenterX : Number = i*100 + 50;

                for (var j:int = 0; j<eachStack.length; j++) {
                    var eachDisk : HanoiDisk = eachStack[j];

                    var diskHeight : int = 180 - (j * 15);
                    var diskWidth  : int = eachDisk.width * 10;

                    var eachShape : UIComponent = new UIComponent();
                    eachShape.graphics.beginFill(0xA00000);
                    eachShape.graphics.drawRect(poleCenterX - diskWidth / 2, diskHeight, diskWidth, 10);

                    vRenderArea.addChild(eachShape);
                }
            }
        }
    ]]]]><![CDATA[>
</mx:Script>
<mx:HBox>
    <mx:Text text="How tall a tower?" />
    <mx:TextInput id="vTowerInput" />
    <mx:Button label="Do It" click="handleDoItClicked()" />
</mx:HBox>
<mx:Canvas id="vRenderArea" width="300" height="200" borderStyle="solid" />
<mx:TextArea id="vConsole" width="100%" height="100%" />
</mx:Application>
```


#### AnimatedTowerOfHanoi


The AnimatedTower just needs to send out change events (via a simple callback vs. true event listeners that are in Flex).

```actionscript
package {

public class AnimatedTowerOfHanoi extends ModeledTowerOfHanoi {
    protected override function setupDisks() : void {
        super.setupDisks();
        
        my_view.noteChange();
    }
    
    public function getStacks() : Array {
        return my_stacks;
    }

    protected override function moveDisk(fromPin : *, toPin : *) : void {
        super.moveDisk(fromPin, toPin);
        
        my_view.noteChange();
    }
}

} //package
```


### Super Easy...But doesn't work


Well, that was really easy... except it doesn't really work :-(  You can only see the final result of all the nice disks on the last stack.  None of the intermediary steps before this last rendering are shown at all.  

Clearly we are missing something... where is the "Sleep" equivalent?  If we could add a 'sleep' then everything would work right?  Unfortunately both "No" and "NO!".  When Flex/Flash are running user code, they are doing it between screen renderings and a bunch of other tasks Flash does.  So you can't just keep rendering things in the same thread (same stack) and have any of that be visible except the last version.  If you could 'sleep', you would just make Flash pause for no good reason.

The solution is to do each rendering on some kind of event.  Something simple like do the next step of the hanoi algorithm every 0.3 seconds.  That sounds easy but: (1) Flex doesn't have continuations either so you can't just 'pause' the stack and (2) The current hanoi algorithm is completely stack based.

Fortunately the fix is both "not that hard" and also happens to be the next chapter of "A Taste of Smalltalk" called "An Algorithm for the Rest of Us".  So for the moment, lets punt on getting the animation in Flex right until we get to the next part.



### What's Next


A new non-stack algorithm.



