---
comments: true
date: 2008-09-02 00:29
layout: post
slug: a-taste-of-flex
title: A Taste of Flex
wordpress_id: 76
categories:
- Flex
- Ruby
- Smalltalk
---

One of my favored books in the 1980s was ‘A Taste of Smalltalk’. This was a very short book that gave you the flavor of Smalltalk programming (the language and IDE) as compared to Pascal, C, and Lisp. I believe it is always important to show multiple programming languages for concepts and this book followed that rule. The goal of this post-series is to take ‘A Taste of Smalltalk’ and apply it to Flex.  This is the second of my comparisons in this series, where the first one was in Ruby.

<!-- more -->


You can find the original book here:



	
  * [A Taste of Smalltalk](http://www.iam.unibe.ch/~ducasse/FreeBooks/Taste/)

	
  * [Free Smalltalk Books](http://stephane.ducasse.free.fr/FreeBooks.html)


Given the original book is available online, I will not repeat it.

To run these Flex examples, you should have a Flex SDK, which you can find at [FlexSDK](http://opensource.adobe.com/wiki/display/flexsdk/Flex+SDK) or via [http://www.adobe.com/](http://www.adobe.com/).  The SDK is free.

For playing with Smalltalk, you can pick up Squeak from [http://squeak.org/](http://squeak.org/).  Squeak's UI is a bit different from Smalltalk-80 (in the book), but if you go through a quick Squeak tutorial, you should be able to connect the two.



## Tower of Hanoi -- Procedural -- Flex


The first example in 'A Taste' is in multiple languages (Chapter 1).  This is Tower of Hanoi done totally procedurally.  Converting to Flex is very simple, where the 'creationComplete' event is about the equivalent of a 'main' / script entrypoint.

```xml
<?xml version="1.0" encoding="utf-8"?>
<mx:Application xmlns:mx="http://www.adobe.com/2006/mxml" layout="absolute"
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
            trace(fromPin + "->" + toPin);
        }

        function handleCreationComplete() : void {
            moveTower(3,1,3,2);
        }
    ]]]]>
</mx:Script>

</mx:Application>
```

Note that Flex will be treated a bit unfairly in the formatting department (as is Smalltalk) because the 'sourcode plugin' in WordPress isn't working for them.  If someone recommends a 'snippet' location that understand Flex or AS3 (and Smalltalk and everything else :-) ), I can put this up there to format it more nicely.

So what is interesting about Flex in this example?  



	
  * Multiple Syntaxes
  * Pascal Declaration Ordering
  * As Statically Typed As You Want
  * No Main
  * The Cheat





### Multiple Syntaxes, each for a different purpose


Well, obviously first Flex is not a single syntax but a composite of at least two syntaxes used for different purposes:



	
  * XML is used to create objects, including the outer 'Application' object

	
  * AS3 (ActionScript3) is used to define classes



So any Flex application will have both of these syntaxes, and they can either be in the same file (as above), or a better practice is to separate the two as much as possible where the XML becomes the View/Layout and the AS files manage behavior.  But for the moment, I will keep them together.

The third syntax is CSS, which can again be either embedded or kept in a separate file.  Although learning three syntaxes is an increase in initial work, two of the three syntaxes are almost universal (XML, CSS) and necessary for doing most anything.  And because each is used for a different purpose, they blend together quite naturally.



### Pascal Notation


ActionScript chose to go down the far more natural 'type suffix' notation of Pascal and related languages.  This isn't really interesting except that C (for compiler ease) went down a different path and a lot of mainstream languages (e.g. Java) followed in that path.  Especially given that ActionScript does not require type declarations, it seems to have chosen wisely, but people are touchy about this kind of syntax choice.



### As Statically Typed As You Want


In the code, you can see types declared as ': *' -- meaning 'Any Type'.  You could also leave off the type declaration and it would mean the same thing, but including the ': *' makes it clear I intend to allow any type of object to be used.  So ActionScript can be as statically typed as you want: either completely dynamic like Smalltalk or Ruby or others, or basically as statically as Java (with a similar static type model).



### No Main


Flex is not a scripting language... it is a graphical application language and application environment that can run either standalone (in Air) or within a browser.  So there is not really a 'main' script that executes.  The equivalent to 'main' is that when an Application is constructed, at the very end of that construction, an event of 'creationComplete' will be sent out.  So this is about as close to a 'main' as we can get.  You could pick up command-line arguments or browser parameters within this 'handleCreationComplete' handler.



### The Cheat


OK, now for the cheat that is in the above code.  If anyone executed the above code within Flex in Debug mode, it would work pretty much as you expect and put lines out to a debugging console.  But it would have to be run in Debug mode for that.  If you run it normally, the 'trace' output wouldn't show anywhere.

I only cheated to make it easier to see the main core of the application code separate from the GUI aspects of the language (just as I tried to do with Ruby).  Flex has graphics capabilities (just like Smalltalk) but is not part of an IDE (as opposed to Smalltalk) and is not a scripting language (like Perl or Ruby) and did not include 'System output' (like Java) because it would be useless in a Browser and for 99.9% of any Flex / Flash deployment.  But it does include tracing for development and debugging purposes... so I cheated and used that.

The important thing about a cheat is to recognize it and fix it... so let's next enhance the Flex application to show output.



## Showing Console-like output in Flex


Adding a console output only requires two lines of code


    
    
            function moveDisk(fromPin : *, toPin : *) : void {
                vConsole.text = vConsole.text+fromPin + "->" + toPin+"\n";
            }
    

      

and

```xml
<mx:TextArea id="vConsole" width="100%" height="100%" />
```

so it would have been just as easy to do it this way initially, but then a "language" difference in its application environment would have been a bit more hidden.






## Augmenting Existing Classes (Monkey-Patching)


As described in 'A Taste of Ruby', the Smalltalk example from the book actually modifies 'Object', so 'moveTower' is available on any object.  This is certainly possible in Flex but would require actively talking to Object.prototype to add behavior.  If you do this, the system will run but the compiler will have no idea that it will run correctly (and really it will have no 'understanding' of the code you added).  Modifying Object was contrived but easy in Smalltalk, contrived but easy in Ruby, and contrived but 'not-so-hard' in Flex.  So for Flex it is the strangest to do.  But here is the very contrived Flex code anyway :-)


    
    
    Foo
    






### Monkey-Patching vs. Source Patching/Augmenting


For Smalltalk and Flex you have the vendor-supplied source, which you could easily patch as part of the source version/modification control system.  So the question becomes:


> Why wouldn't patching occur at the source-code level so people can see what changes have happened?




Smalltalk wrestled with this for years with different vendors and different teams choosing different approaches.  I personally believe that the current Monticello for Squeak and the heavyweight 'Envy' (and similar source code tools) get this right in that you could certainly modify an existing class but those changes had to be documented as source changes and had to _not conflict_ (with Monticello having a nicer model than Envy [if memory serves for Envy]).

So although I don't think Flex needs these kinds of changes particularly for what Flex is used for, something that enables you to bring system classes into your own modifiable version-control space and then allow patching on that space is nicer than having everyone hit the main space without clear visibility to what they are doing (effectively Ruby's current mainstream choice).



## Comments


This example from "A Taste of Smalltalk" is the highly procedural version.  It is not meant to be a particularly good program, but it is a simple and good introductory example.  Flex does fine with this example: it does not artificially type the 'moveTower' parameters.  It has a bit more static typing, which helps avoid typos.  But everything is an object, so dynamic typing works fine too.  

Because Flex includes an inherent GUI environment (but no inherent Console) we had to add one line of MX code to create that console.  Flex is a three-syntax set of families so it can look a bit complex to begin with (wrapper XML and the CDATA ActionScript) but the use of the XML is clean and for a specific purpose that ActionScript and most languages (say other than Lisp) are not as good for.  And using XML for page layout is very familiar to people (e.g. XHTML) and goes nicely with the CSS capabilities in Flex as well.



### Where Next?


Next we should go to Chapter 3 -- Page 36.  The Second Example.

