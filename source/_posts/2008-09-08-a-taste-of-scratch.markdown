---
comments: true
date: 2008-09-08 04:15
layout: post
slug: a-taste-of-scratch
title: A Taste of Scratch
wordpress_id: 169
categories:
- Ruby
- Scratch
- Smalltalk
---

One of my favored books in the 1980s was ‘A Taste of Smalltalk’. This was a very short book that gave you the flavor of Smalltalk programming (the language and IDE) as compared to Pascal, C, and Lisp. I believe it is always important to show multiple programming languages for concepts and this book followed that rule. The goal of this post-series is to take ‘A Taste of Smalltalk’ and apply it to Scratch (a visual programming language).  This is the third of my comparisons in this series, where the first one was in Ruby.


<!-- more -->

You can find the original book here:



	
  * [A Taste of Smalltalk](http://www.iam.unibe.ch/~ducasse/FreeBooks/Taste/)

	
  * [Free Smalltalk Books](http://stephane.ducasse.free.fr/FreeBooks.html)


Given the original book is available online, I will not repeat it.

You can both see the example in Scratch and get tools and background on the Scratch project at:
[Tower of Hanoi](http://scratch.mit.edu/projects/parseroo/260752).  The scratch tools and the community web site are free.



## Tower of Hanoi -- Object-Oriented (V4) -- Scratch


The standard pattern for this series is to translate as closely as possible each of the chapters in "A Taste of Smalltalk" to the new target language.  This worked reasonably well for Ruby and Flex -- the Smalltalk was translated and interesting features of the language came out with each translation.  But this approach utterly fails for Scratch because the first two 'models' of the Hanoi algorithm are completely dependent on recursion, which Scratch does not have.  Even the third, stack-less version of Hanoi requires inter-object calls that Scratch can not handle.  So _none_ of the algorithms within "A Taste of Smalltalk" can be directly translated to Scratch.

This might appear to be a limitation of the language (and in some ways it certainly is), but somewhat impressively, I believe thinking about this problem, with the restrictions of Scratch, actually leads to a better design solution than any of the ones in the book.  It is certainly a very object-oriented solution, where the 'Hanoi Disks' have a lot of intelligence and are made as responsible as possible for figuring out what to do next.  This version of the algorithm, I will call 'V4'.



### The V4 Algorithm


The V4 Algorithm works as follows:



	
  1. Ask each of the disks whether they have a legal move

	
  2. Decide which of the disks with a legal move should move

        
  3. Tell that disk to move



The interesting parts of this algorithm are that:

   
  * Step 1 can be completely parallel.  You can ask 1 to 1000000 disks at the same moment whether they have a legal move.

   
  * Step 2 is really trivial: just don't move the same disk you did last time


Compared to the V3 version, the V4 version puts more intelligence in the disks (and less in the tower), couples them less, and supports mass parallel-execution.  The parallel-execution is not useful in Hanoi, but the concepts behind it are definitely very interesting -- and Scratch's restrictions forces this kind of 'sophisticated' approach [or at least it forced me down this path to maintain code-sanity and maintainability].



### The main Scratch Player view


Scratch is designed to be an easy-to-learn language and environment.  Its heritage is along the lines of the spirit of Logo: there is a Stage drawing area and a default Sprite (a Cat) that you interact with for animation and drawing.  But this Stage can also show the variables involved with the program.  So it produces a nice overview of the whole Hanoi program:

[![](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d3.png)](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d3.png)

Not all application variables are shown, but those shown are the most important ones.  The details of the variables will be gone over later, but some are self-evident:



	
  * num-disks: The number of disks (the height) of the Tower of Hanoi

        
  * pole-#: A List (treated as a Stack) of Disk identifiers, where '6' = a base (immovable) disk

        
  * movable-disks: A List of disks that can be moved.  This is transitory for each iteration of the algorithm.



If you click the 'Green Flag', the program executes and on pretty much every step the display is updated, so you can see disks being proposed and moved (in both animation and variables).  Independently of Scratch having a graphical programming language, it has a very nice graphical development and run environment.



### Application Variables


The application variables are shown below.  All of these are 'Globals' and can be accessed by any object.  Those variables colored in red are lists and those colored in orange are numbers/strings/etc.  All the variables that have check-marks next to them are being displayed on top of the Stage above.

[![](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d1.png)](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d1.png)

There is no such thing as a 'local' variable (private to a block of code), so it might be useful to use a naming prefix to differentiate between 'local' variables (like 'potential-disk') from instance variables and even 'parameter' variables that go with a broadcast ('next-disk').  But in-the-small, that is not particularly important.



### Application Initialization


Now that we have an overview of the application and its global state variables, we can start working through the algorithm itself.  The main 'Object' in Scratch is the Stage, and I made this own the outer-most aspects of the Hanoi application and algorithm.  The first activity is 'Initialization', which is triggered by the 'Green Flag' [my particular choice].  

[![](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d2.png)](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d2.png)

Because this is the first example of Scratch, I will walk through the sections of the code and what they do with a few visual marker numbers (see the right column of blue).

When the Stage gets the Green Flag event: 




  1. It sets the number of disks to '5'.  This is hard-coded because Scratch is currently unable to clone Sprites (objects) on the fly, so we have to precreate all the needed sprites.  In any case, this is simple enough to change if someone wants.  The additional variable 'base-disk' is defined so it can be used below.  'base-disk' is just a local variable and is never used again. After this, we just clear the lists so they are 'clean'


  2. Next we add the 'base-disk' to the bottom of each pole.  The base-disks are invisible and immovable (no matching Sprite/Object), so this is a very clean way of making the algorithm simpler later on. 


  3. Next we add the disks to 'pole-1' using what should be another local variable.  Later I reuse this variable [something I don't like doing] just because we otherwise have a clutter of variables.


  4. Next we send out a broadcast to _all objects_ telling them to initialize themselves.  There is no direct message send to an object, so it is up to the other objects to know what to respond to and what to ignore.  The broadcast waits until all object's acknowledge they are finished.


  5. Finally, we go into the main loop.  Until all the disks have moved to a new pole, we will execute 'find-next-disk' which both finds and moves the next disk.  When we are done, we make a noise :-)





### Find and Move Next Disk


The main part of the algorithm has some nice aspects and some nastier aspects.  The nice side: We simply send out a broadcast of 'propose-next-disk', which all Disks should respond to by adding themselves to 'movable-disks' if they have a desire to move.  This will only ever be one or two movable disks at any given time.  One of those disks may have been moved immediately previously, so we skip over it.  The selected disk is put into 'next-disk' and another broadcast goes out "move-next-disk".  The second broadcast is not really a broadcast.  It is a message send to 'next-disk', but it is only by the disks themselves ignoring a broadcast that isn't to them that it becomes a message send.  So that is slightly nasty.

In any case, the code is pretty clean and concise, and this ends the code on the Stage itself.  The rest of the algorithm is on the Disks.

[![](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d4.png)](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d4.png)



### Disks Variables


The Disks have access to all the Stage global variables.  In addition, each Sprite can have its own instance-private variables, which are either true instance variables ('disk-id', 'pole', 'previous-pole') or would be local variables if Scratch supported them ('can-move', 'am-on-top', etc.)

[![](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d5.png)](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d5.png)

Instance variables can be displayed on the Stage also, but this becomes overwhelming if all instances are turned on.




### Disk Initialization


Upon receiving the 'initialize' broadcast, all Disks set their 'disk-id' (which must be different for each disk), and then set themselves up properly in size, color, and move onto the correct pole.

[![](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d6.png)](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d6.png)




### Propose Next Disk


Scratch has certain deficiencies that rear their heads badly in the Hanoi code.  The concept of Propose Next Disk is as simple as:


    
    
    If 
       * I am on-top of a pole 
       * And I can move to a new pole (with a bigger disk on its top) 
       * And that new pole isn't the pole I just came from
    Then
       * Add myself to the movable-disks list
    



But because Scratch (or my knowledge of Scratch) is deficient in dynamic referencing of variables, the 'propose-next-disk' code needs to expand all the potential pole values from 1 through 3.  Fortunately this is only a three-valued expansion, but the code looks quite nasty for being as simple as the above.

[![](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d7.png)](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d7.png)



### Move Next Disk


The Move Next Disk code has (1) the filter on the broadcast (in this case by 'disk-id') to make it only go to a single object and (2) a ton of noise due to 'pole' expansion mentioned above.  But the code gets the job done and it looks fairly colorful :-)

[![](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d8.png)](http://chimu.files.wordpress.com/2008/09/atasteofscratch_d8.png)



## Conclusions


I was impressed that Scratch was able to get Tower of Hanoi to run properly.  Scratch has actively avoided certain features that the Scratch team finds are difficult to understand.  But the toolbox of event processing, broadcasts, lists, and Sprites (objects with Stage presence :-) ) are rich enough that a pre-sized Hanoi can be created and will run correctly.  A dynamic Hanoi would require Sprite cloning, which is not in Scratch 1.3.

The nastiest aspect to Scratch was its inability to dynamically reference the pole lists.  And a missing feature to Scratch are simple procedures (for example, the 'glide' code is repeated twice).  The missing recursion is clearly another missing feature, but in this case getting rid of that recursion made the algorithm somewhat nicer.

On the whole, Scratch is a very inspiring visual language and development environment, and it did a good job with this 'offbeat' test.



