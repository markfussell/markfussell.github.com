---
comments: true
date: 2008-09-17 20:28
layout: post
slug: scratch-textual-language
title: Scratch Textual Language
wordpress_id: 195
categories:
- Scratch
---

[Scratch](http://scratch.mit.edu/) is a visual programming language, but it might usefully have a textual version as well.
Scratch in it's graphical form can verify syntax intuitively & automatically, and the resulting graphics are definitely beautiful, but it took quite some effort to produce
[these diagrams](/blog/a-taste-of-scratch)
when trying to document a program, and it is even harder to reference sections of the diagram.

<!-- more -->

This activity came out of producing a Scratch Player in Flash, and I wanted to see whether the parse of the file produced the correct program.  A textual dump of the parse was an obvious solution, but then this 'dump' might as well be the same as a complete program declaration so it was clearly 'complete' and I wasn't missing anything.




## Scratch Textual


The core concepts in this proposed textual language for Scratch are:




  * It should match Scratch as closely as possible


  * It should be a readable and easily writeable textual language (vs. XML as an interchange format)


  * It should be translatable to other human languages for the blocks (as Scratch is)


  * It should be really simple to produce and parse


  * Given Scratch is graphical, visual indentation is reasonable


  * It is not a complete 'dump' of the Scratch project -- no media resources are included.



The above concepts produced the following initial choices:


  * Different sections start with simple keywords: "stage:" and "sprite:" for the main objects, "vars:" and 'listVars:" for declaring variables, "comment:" for a comment section, and "script:" for declaring a sequence of blocks.


  * Variables simply define their name and an initial value (if desired... otherwise it would be '0')


  * Each block is written out textually with its 'spec' -- a human readable description of the block and its parameters


  * The parameters to a block (if any) come immediately after, and are preferably indented [say 4 spaces] from the block they are parameters too.  Note that the indentation is not necessary because you can tell the number of parameters from the block 'spec', but indenting makes it more readable.


  * If a block can contain other blocks, contained blocks _must be_ indented [say 4 spaces] so you can tell they are inside the block vs. after the block.  This is in a fashion similar to Python.


  * There should be no blank line within a 'script:' section.  The end of a script: section is the first blank (non-comment) line.


  * Comments within the textual blocks use '#' to match with Python.  Multi-line comments do not exist.  Note that Textual Comments are not parsed at all (and do not round-trip), so they should be minimal and merely augment parsing the textual source -- use 'comment:' for real documentation.


  * 'script:' and 'comment:' could have position information after them using the 'loc' property (e.g. "loc = 500@250") but this is not likely to be useful since any edits would cause positions to be wrong





### An Example


The following is a complete "normal expanded format" version.  See further below for a possible optimization to make things more vertically compressed.


```python

#===========================
#=== Morph: Stage 
#===========================
stage: Stage

#============
#=== Vars 
#============

vars:
talest-pole-height = 6
previous-moved-disk = 0
potential-disk = 1
num-disks = 5
next-disk = 0
base-disk = 0

listVars:
pole-1 = undefined
pole-2 = undefined
moveable-disks = undefined
pole-3 = undefined

#============
#=== Script 
#============
script:

when 'find-next-disk'
set next-disk to %n
    0
broadcast %e and wait
    propose-next-disk
repeat until %b  
    %n > %n
        next-disk  
        0
    set potential-disk to %n
    if %b  
        not %b
            %n = %n
                potential-disk  
                previous-moved-disk  
        set next-disk to %n
            potential-disk  
broadcast %e and wait
    move-next-disk

#============
#=== Script 
#============
script:

when 'Scratch-StartClicked'
set num-disks to %n
    5
set base-disk to %n
    %n + %n
        num-disks  
        1
set next-disk to %n
    num-disks  
repeat until %b  
    %n < %n
        next-disk  
        1
    set next-disk to %n
        %n - %n
            next-disk  
            1
broadcast %e and wait
    initialize
set previous-moved-disk to %n
    0
set talest-pole-height to %n
    0
repeat until %b  
    %n > %n
        talest-pole-height  
        num-disks  
    broadcast %e and wait
        find-next-disk
    set talest-pole-height to %n
    if %b  
        %n > %n
            talest-pole-height  
        set talest-pole-height to %n


#===========================
#=== Morph: Sprite1 
#===========================
sprite: Sprite1

#============
#=== Vars 
#============

vars:
can-move = 3
am-on-top = 1
disk-id = 1
pole = 3
height = 6
previous-pole = 1
pole-at-start = 1

#============
#=== Script 
#============
script:

when 'move-next-disk'
if %b  
    %n = %n
        disk-id  
        next-disk  
    set previous-moved-disk to %n
        disk-id  
    set pole-at-start to %n
        pole  
    if %b  
        %b and %b
            not %b
                %b or %b
                    %n = %n
                        previous-pole  
                        1
                    %n = %n
                        pole  
                        1
            %n < %n
                disk-id  
        set pole to %n
            1
    if %b  
        %b and %b
            not %b
                %b or %b
                    %n = %n
                        previous-pole  
                        2
                    %n = %n
                        pole  
                        2
            %n < %n
                disk-id  
        set pole to %n
            2
    if %b  
        %b and %b
            not %b
                %b or %b
                    %n = %n
                        previous-pole  
                        3
                    %n = %n
                        pole  
                        3
            %n < %n
                disk-id  
        set pole to %n
            3
    set previous-pole to %n
        pole-at-start  
    if %b  
        %n = %n
            previous-pole  
            1
    if %b  
        %n = %n
            previous-pole  
            2
    if %b  
        %n = %n
            previous-pole  
            3
    if %b  
        %n = %n
            pole  
            1
        set height to %n
    if %b  
        %n = %n
            pole  
            2
        set height to %n
    if %b  
        %n = %n
            pole  
            3
        set height to %n

#============
#=== Script 
#============
script:

when 'propose-next-disk'
set am-on-top to %n
    0
set can-move to %n
    0
if %b  
    %b and %b
        %n = %n
            pole  
            1
        %n = %n
            disk-id  
    set am-on-top to %n
        1
if %b  
    %b and %b
        %n = %n
            pole  
            2
        %n = %n
            disk-id  
    set am-on-top to %n
        1
if %b  
    %b and %b
        %n = %n
            pole  
            3
        %n = %n
            disk-id  
    set am-on-top to %n
        1
if %b  
    %n > %n
        am-on-top  
        0
    if %b  
        %b and %b
            not %b
                %b or %b
                    %n = %n
                        previous-pole  
                        1
                    %n = %n
                        pole  
                        1
            %n < %n
                disk-id  
        set can-move to %n
            1
    if %b  
        %b and %b
            not %b
                %b or %b
                    %n = %n
                        previous-pole  
                        2
                    %n = %n
                        pole  
                        2
            %n < %n
                disk-id  
        set can-move to %n
            2
    if %b  
        %b and %b
            not %b
                %b or %b
                    %n = %n
                        previous-pole  
                        3
                    %n = %n
                        pole  
                        3
            %n < %n
                disk-id  
        set can-move to %n
            3
    if %b  
        %n > %n
            can-move  
            0

#============
#=== Script 
#============
script:

when 'initialize'
set disk-id to %n
    1
set pole to %n
    1
set previous-pole to %n
    0
set can-move to %n
    0
set am-on-top to %n
    0
set height to %n
    %n + %n
        %n - %n
            base-disk  
            disk-id  
        1
```



### Vertical Compression


The format above has every block on its own line.  This is readable and informative (you can see the argument types) but is vertically expansive.  Assuming parentheses or some other paired delimiter are not used for anything else in names, we could replace the argument placeholders with parentheses like below.  This gives a more lisp-like feel, at least for the inner evaluations.  If you leave one or more placeholders off, they would come from the next lines in 'missing' order, kind of like a macro.


```python
script:

when 'find-next-disk'
set next-disk to (0)
broadcast (propose-next-disk) and wait
repeat until ((next-disk) > (0))
    set potential-disk to (0)
    if (not ( (potential-disk) = (previous-moved-disk)) )
        set next-disk to (potential-disk)
broadcast (move-next-disk) and wait

```
