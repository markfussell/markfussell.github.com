---
layout: post
title: "Scratch on Flash"
date: 2008-09-19 18:00
comments: true
categories: Scratch
---

The original version of this posting is on the [MIT Scratch Forum](http://scratch.mit.edu/forums/viewtopic.php?id=9982) and the
current location of the repository is [https://github.com/markfussell/scratchonflash](https://github.com/markfussell/scratchonflash)

In the spirit of release early and often...

I have made a full bottom-to-top pass at implementing Scratch in Flash.  It can:

   * Read a scratch project file
   * Create the project model (Stage, Sprites, Blocks, etc.)
   * Run that model (the core variable assignments, loops, etc.)
   * And do a couple of the other kinds of Sprite blocks in the 'motion' and 'looks' categories

<!-- more -->

It most definitely does not:

   * Respond to mouse or key events yet (but that is trivial to finish...)
   * Do all the blocks... this is going to take work to map them all into Flash and especially the Sound area could be problematic
   * Draw things properly -- for one, the center of the stage is not 0,0 and the y axis is 'upside-down' [I said this was 'early']
   * Run the model correctly for all control blocks.  The core concepts are there and the blocks I have tested work right [and I picked interesting ones], but I have not tested beyond a couple projects
   * Read the Sound media at all
   * Show any of the Watchers -- Either variables or lists


But given those caveats and all that I have forgotten, there is a demo at Scratch on Flash

You put a project URL into the text area and click 'Open'.  Then click 'Go!' and if your project responds to 'Go!' things may or may not start happening :-)    Note that the URL is smart about shared projects on this site (and locates the '.sb' file automatically if you give it the project's url) but if you use a local file system URL or some other web site, you will need to have a full path to the '.sb' file.

The window has a console at the bottom, a textual dump of the program above that, and has a 'Step Processor' above that.  By default the program is running one step every 30th of a second, but you can turn that off or make more steps occur every 30th of a second.

=======================

It could easily be that only the one project 'Tower of Hanoi' does anything interesting... probably within a short time some other projects would work too.  By default, I would probably start picking and choosing among Tutorials or the included 'Projects' (is there a Gallery on the site for all the Included projects?).

I will release source in some format in the next release -- things are pretty clean and somewhat commented, but I think I should get a little further and make sure I am not going to rename or repackage anything.  And I will need to strip out the few 'Flex' references from within the core Scratch model so a smaller SWF download is possible.

If anyone is interested in working on this -- doing Blocks and verifying that functionality is working, please let me know.  If people are interested then I will put source into a public RCS like Subversion or Git sooner than later.
