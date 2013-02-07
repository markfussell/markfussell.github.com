---
comments: true
date: 2008-08-28 18:00
layout: post
slug: a-taste-of-ruby
title: A Taste of Ruby
wordpress_id: 41
categories:
- Dynamic
- Ruby
- Smalltalk
---

One of my favored books in the 1980s was 'A Taste of Smalltalk'.  This was a very short book that gave you the flavor of Smalltalk programming (the language and IDE) as compared to Pascal, C, and Lisp.  I believe it is always important to show multiple programming languages for concepts and this book followed that rule.  The goal of this post-series is to take 'A Taste of Smalltalk' and apply it to Ruby.

<!-- more -->


You can find the original book here:



   
  * [A Taste of Smalltalk](http://www.iam.unibe.ch/~ducasse/FreeBooks/Taste/)
  * [Free Smalltalk Books](http://stephane.ducasse.free.fr/FreeBooks.html)




Given the original book is available online, I will not repeat it.

You should have a Ruby interpreter, which you can find at [http://www.ruby-lang.org/](http://www.ruby-lang.org/) or it may already be on your computer.  The code examples should work pasted directly into 'irb'.

For playing with Smalltalk, you can pick up Squeak from [http://squeak.org/](http://squeak.org/).  Squeak's UI is a bit different from Smalltalk-80 (in the book), but if you go through a quick Squeak tutorial, you should be able to connect the two.




## Tower of Hanoi -- Procedural


The first example in 'A Taste' is in multiple languages (Chapter 1).  This is Tower of Hanoi done totally procedurally.  Converting to Ruby is very simple and clean:



### Tower of Hanoi -- Procedural -- Ruby


```ruby
# A Taste of Ruby.  Based on a Taste of Smalltalk (Kaehler and Patterson)
# Tower of Hanoi -- Variation 1

# Recursive procedure to move the disk at a height
# from one pin to another pin using a third pin
def move_tower(height, fromPin, toPin, usingPin)
  if (height>0) then
    move_tower(height-1, fromPin, usingPin, toPin)
    move_disk(fromPin, toPin)
    move_tower(height-1, usingPin, toPin, fromPin)
  end
end

# Actually move the disk between the pins
def move_disk(fromPin,toPin)
  print fromPin.to_s, "->", toPin.to_s, "\n"
end

# Do the tower
move_tower(3,1,3,2)
```

You could simply paste this into irb to see it run.



### Smalltalk without the IDE


Some people (many people) consider Smalltalk to mandatorily include an IDE and that you _must _ use an IDE to create Smalltalk code and to run a Smalltalk application.  This is clearly not true and there are many examples of Smalltalk running without any IDE at all.  For example, GNU Smalltalk, Little Smalltalk, GemStone, and 'Headless Smalltalk' (when you deploy VisualWorks you frequently strip the IDE code out) are all valid Smalltalks that do not have an IDE in themselves and if you wanted them to run a program, you would just 'File it in' from a file.  I mention this because the next chapter of "A Taste of Smalltalk" shows the Smalltalk-80 IDE as the way to create the Smalltalk code.  If you download [Squeak](squeak.org) you can follow fairly similar  steps to the one in the book.

To make inter-language comparisons easier, I will instead use the 'File In' format when talking about Smalltalk programs.  Since that is missing from the book, here it is.  Note that the lines starting with '!' are not Smalltalk language syntax but are part of the standard library for being able to 'file in' changes.  For most programming languages the two would be equivalent, but Smalltalk allows multiple ways of creating a program so the coupling is softer.



### Tower of Hanoi -- Procedural -- Smalltalk



    
```smalltalk

    !Object methodsFor: 'hanoi' stamp: 'MLF 8/28/2008 10:31'!
    moveTower: height from: fromPin to: toPin using: usingPin
        (height > 0) ifTrue: [
    		self moveTower: (height-1) from: fromPin to: usingPin using: toPin.
    		self moveDisk: fromPin to: toPin.
    		self moveTower: (height-1) from: usingPin to: toPin using: fromPin
    	]
    !
    
    moveDisk: fromPin to: toPin
    	Transcript show: (fromPin printString, ' -> ', toPin printString); cr.
    
    ! !
    
    "Do the tower
    (Object new) moveTower: 3 from: 1 to: 3 using: 2
    !
    
```




Now an interesting thing about the Smalltalk version is that we had to create an object to execute our code.  Smalltalk does not have functions or procedures: it only has objects and messages that get sent to objects.  So we need some object to respond to the message.  There could be an implicit object associated with the 'File In' format -- but there isn't.  A second interesting thing about the Smalltalk version is that we modified Object itself.  So any of:



	
  * 'Foo' moveTower: 3 from: 1 to: 3 using: 2

	
  * 3 moveTower: 3 from: 1 to: 3 using: 2

	
  * false moveTower: 3 from: 1 to: 3 using: 2

	
  * nil moveTower: 3 from: 1 to: 3 using: 2




would work.  So because we modified Object and there are well known objects of type Object (and descendants) in the system, we didn't have to actually create an Object at all.



### Ruby is cleaner


In this particular case, Ruby is:



	
  * Doing a much cleaner thing by default (for this file-in approach)
  * But can do exactly the same thing if told to




The "cleaner thing by default" is that Ruby statements (commands) when being executed are always talking to some object, and the default object is an object called 'main'.  So statements without an explicit object _receiving_ the message send those messages to 'main' -- and the message "move_tower" is sent to main.  And although Ruby doesn't describe itself quite like this, effectively 'def' could be viewed as a message and it is also sent to 'main'.  'def' creates a method on a Class -- either the eigen-class of the receiving object or on a Class object.  So Ruby behaves almost identically to Smalltalk except it:



	
  * Has a default object for 'File In' format
  * Creates methods on the eigen-class if a non-class is the target of the def 'message'



Probably the simplest way for Smalltalk to match this cleaner scripting approach is for a Smalltalk 'terminal' (called IRS :-) to define 'self' as being itself and then the method definitions would automatically happen on itself.  So you would have lines like:

	
  * '!self methodsFor: 'hanoi' ...
  * self moveTower: 3 from: 1 ...




and applying to Object would no longer be necessary.  This 'scripting' need has not been important to the Smalltalk community in general although someone may have something very like this.



### Ruby being a bit more Smalltalkish


The second point above was that Ruby can do exactly what the Smalltalk code does.  This isn't to show that this is a better approach (in this case it is not), but to show the incredible similarity between the two languages.

```ruby
# A Taste of Ruby.  Based on a Taste of Smalltalk (Kaehler and Patterson)
# Tower of Hanoi -- Variation 1b

class Object

# Recursive procedure to move the disk at a height
# from one pin to another pin using a third pin
def move_tower(height, fromPin, toPin, usingPin)
  if (height>0) then
    move_tower(height-1, fromPin, usingPin, toPin)
    move_disk(fromPin, toPin)
    move_tower(height-1, usingPin, toPin, fromPin)
  end
end

# Actually move the disk between the pins
def move_disk(fromPin,toPin)
  print fromPin.to_s, "->", toPin.to_s, "\n"
end

end

# Do the tower
(Object.new).move_tower(3,1,3,2)
puts "---"
"HiThere".move_tower(3,1,3,2)
puts "---"
nil.move_tower(3,1,3,2)
```

This similarity is way beyond casual similarity -- enabling 'nil' to have a method by augmenting Object with almost exactly the same incantation makes for a very similar DNA.  Basically it seems like the Ruby language is a pure syntactic mutation on Smalltalk.



### Fun with Ducks


The term "Duck Typing" was not around in the 80s (AFAIK), but instead people talked about 'Dynamic Typing' where the Duckness was hidden under that Dynamic-ness.  There are certainly other terms too (I used the term 'protocol' where protocol was just obeying the required messages and semantics properly -- i.e. acting like a Duck).  This Hanoi example has a nice Duck property -- We don't care what the stacks are as long as they can be converted to Strings:


    
    
    #Bizarre Hanoi Showing Duck Typing
    move_tower(3,"Tower-1", nil, Proc.new {|| "Tower" })
    







### Tower of Hanoi -- Procedural -- Java


For completeness, we should do the example in Java as well.  Given Java execution starts at some static 'main' method/procedure, it seems like the most natural version of this Procedural code is to stay on that static side.


```java
class Hanoi {
   static public void main(String[] args) {
      moveTower(3,1,3,2);
   }

   static public void moveTower(int height, Object fromPin, Object toPin, Object usingPin) {
      if (height > 0) {
         moveTower(height-1, fromPin, usingPin, toPin);
         moveDisk(fromPin, toPin);
         moveTower(height-1, usingPin, toPin, fromPin);
      }
   }

   static public void moveDisk(Object fromPin, Object toPin) {
      System.out.println(""+fromPin+"->"+toPin);
   }
}

```




## Comments


This example from "A Taste of Smalltalk" is the highly procedural version.  It is not meant to be a particularly good program, but it is a simple and good introductory example.  Because Ruby is an interpreter-oriented (scripting-oriented) language, it does quite well in making the example look simple and clean.  But even better (IMO) is that the developer has unknowingly shifted into a much more powerful programming model without any pain-of-entry.  I have never found Smalltalk to be hard but many people do and any barrier that prevents people from entering a community can harm both sides.

I believe the syntactic mutation in Ruby that hides an implicit object ('main') actually helps tremendously with accessibility of the language.  It would only be a problem if the syntactic mutation later penalized the experienced developer.



### Logo, Turtles, and 'main'


I recently decided my daughters (4 and 7) might as well learn how to program, so I fired up a version of [Logo](http://en.wikipedia.org/wiki/Logo_(programming_language)) for them to play with.  Logo has a pleasing characteristic where you are talking to a graphical Turtle by default -- you can program the turtle by telling it what to do directly and can then start working your way up in programming sophistication.  The Ruby 'main' seems to lend itself to the same thing.  You are by default talking to a (more boring :-) ) turtle, which makes the first level of accessibility easier.

At one point I wanted to test something quickly in Smalltalk and actually forgot what object I had to talk to to do a simple 'print'.  Likewise with Java, 'System.out' is extremely obscure.  The 'Turtle' approach seems to help alleviate these kinds of issues a bit.



### Where Next?


Next we should go to Chapter 3 -- Page 36.  The Second Example.
