---
comments: true
date: 2008-09-02 04:12
layout: post
slug: a-taste-of-flex-part-2
title: A Taste of Flex (Part 2)
wordpress_id: 90
categories:
- Flex
- RubyShoes
- Smalltalk
---

This is a multi-part series.  The first in the series is [here](../a-taste-of-flex).



## Tower of Hanoi -- Procedural With Input


The second example from "A Taste of Smalltalk" is in Chapter 3, page 36 and does basic IO.  Except that Smalltalk-80 includes a Windowing graphics system so it can automatically open Dialogs and other things.    Fortunately Flex includes a Windowing system too, and can easily open up dialogs.  Opening up a Dialog might match the Smalltalk example the best, but given Flex is already running within a Window, opening up an extra dialog is a bit artificial.  Instead we can just add an input field for the value and a button to kick off the Hanoi.

<!-- more -->



### Tower of Hanoi -- Procedural With Input -- Flex


```xml
<mx:Application xmlns:mx="http://www.adobe.com/2006/mxml" layout="vertical"
	creationComplete="handleCreationComplete()"
>
<mx:Script>
    <![CDATA[
        function moveTower(height : int, fromPin : *, toPin : *, usingPin : *) : void {
            if (height > 0) {
                moveTower(height-1, fromPin, usingPin, toPin);
                moveDisk(fromPin, toPin);
                moveTower(height-1, usingPin, toPin, fromPin);
            }
        }

        function moveDisk(fromPin : *, toPin : *) : void {
            vConsole.text = vConsole.text+fromPin + "->" + toPin+"\n";
        }

        function handleCreationComplete() : void {
            //Don't need to do anything anymore
        }

        function handleDoItClicked() : void {
            vConsole.text = vConsole.text+"\n=== Doing a Hanoi tower "+vTowerInput.text+" tall\n";
            moveTower(new Number(vTowerInput.text), 1, 3, 2);
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


To replace the dialog (in Smalltalk) or the command line request (in Ruby), we just create a horizontal row of a question label, an input field, and a Do It button.  Whenever a user clicks Do It, it takes the current value of the field and runs the Hanoi algorithm.

This is both pleasingly simply and also has the added benefit that we produced an interactive tool: you can just keep typing new numbers into it and it will give you new results.



## Comments


Flex is highly capable with user input and interactivity.  It has a cleaner / simpler UI model than Smalltalk (IMO), which is good given we have had about 30 years of working with UIs to take things forward.  Compared to Ruby, Flex is as easy as RubyShoes for simple things like this example.  



### Where Next?


Next we should go to Chapter 4 — Page 44. The Third Example (with Real Hanoi Classes)


