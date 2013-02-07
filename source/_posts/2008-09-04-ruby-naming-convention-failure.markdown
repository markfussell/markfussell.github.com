---
comments: true
date: 2008-09-04 01:20
layout: post
slug: ruby-naming-convention-failure
title: Ruby Naming Convention Failure
wordpress_id: 144
categories:
- Dylan
- Java
- Ruby
- Smalltalk
---

Ruby has a strong recommendation against using CamelCase for method and variable names and to use only underscores instead.  There are lots of arguments out there on different naming conventions, and whatever side I pick in isolation is fairly irrelevant since either the whole community or the individual team has to chose the best approach.  

<!-- more -->

In case people care, in isolation of other issues, I would pick this order as being the most natural:



	
  1. Mark-Fussell (or mark-fussell)

	
  2. MarkFussell

	
  3. Mark_Fussell (or mark_fussell)


where the underscore goes third mostly because it can be hard to see [URLs and other things put underlines _on top_ of that underline like thus _Mark_Fussell_], is harder to type, and it over spaces things visually.  So I (and lots of people actually) believe programming languages _should_ use hyphens, but because a bunch of programming languages want 'a-b' to be interpreted as 'a - b', they cop out and prevent the hyphen from being part of a name.  Ruby has that same cop out, so we can't use hyphens in Ruby :-(



## Missing Phrase Delimiter


So I said my opinion is fairly irrelevant, but I also want to say that people have missed an important aspect to this argument.  I may have a small preference for CamelCase over underlines as a single word delimiter, but I have a _huge_ preference to having both a 'word' and a 'phrase' delimiter.  And by insisting on just one delimiter in this naming standard, Ruby has significantly interfered with expressiveness.  Maybe I am unusual, but I have long-time argued that we should have both words and phrases in methods and frequently even in classes.  Back in the 1990s, I documented this as part of my [JavaStandards](http://www.chimu.com/publications/javaStandards2/part0006.html#E11E13) and this was based on Smalltalk naturally having both of these pieces.  It was clearly more readable and you were much less likely to produce bad method names.

For method naming, the quick summary is:



	
  * When naming a method

	
  * Start with a verb phrase (what you are asking the object to do)

	
  * Put an underscore where a parameter is expected (up to the first two at least)

	
  * Describe what that parameter is for right before the underscore, if it is not obvious from the verb


this way, the reader of the code can immediately know how the parameters (in the parentheses) map onto their role in the method itself.  And the whole method name becomes a readable phrase... not just a weird long word strung together (by whatever convention you want).



### Array::insert


So an example is:
   
    
    Array::insert(index, obj)


This is a method name that is ambiguous in usage:
   
    
    anArray.insert(3,2)


It is not clear what the code is going to do and I would actually intuitively expect it to be the other way around (except I know about getting burned so would then have to look it up).

A better name would be:
   
    
    Array::insert_at(obj,index)


so reading:
   
    
    anArray.insert_at(3,2)


is clearly inserting the '3' not inserting the '2'. 

Or if people like the order the other way:

    
    
       Array::insertAt_value(index,obj)
    
       anArray.insertAt_value(2,3)
    



And in the Ruby case where any number of values could follow, we should have:
  
    
    
       Array::insertAt_values(index,obj...)
    
       anArray.insertAt_values(2,3)
       anArray.insertAt_values(2,5,3)
    


and everything reads naturally something like this "anArray insertAt: 2 values: 5 and 3".



### forPolesOtherThan_do(disk)


Related to a recent post on the Tower of Hanoi, I believe the resulting choice of:
   
    
    forPolesOtherThan_do(disk) [block]


is much clearer in both communicating what the passed in 'disk' is for and that it requires a block as a second (semi-hidden) argument than if you have only a word delimiter and have any of:

    
    
       for_poles_other_than
       for_poles_other_than_do
       forPolesOtherThan
       forPolesOtherThanDo
    





## What to do?


Given Ruby wants to use underscore to separate words... I can't separate the phrases at all.  I tried to wrestle my mind around using something different, but Ruby surprisingly (for a modern and internationally-created) language does not have any other punctuation that is allowed and could plausibly work.  So maybe use two underscores for the phrase point?  That seems seriously ugly and also two underscores don't really look different from one underscore:

    
    
       insert_at__values
       for_poles_other_than__do
    



Using capitals for this purpose just comes off weird (plausible but weird):

    
    
       insert_at_Values
       for_poles_other_than_Do
    



It is also exactly the opposite of Smalltalk, and I try to be able to work across languages without having to flip upside down all the time.  Same regarding Java and Flex (two other languages I actively work in).  For reference, the Smalltalk version of these examples are:

    
    
       insertAt: values:
       forPolesOtherThan: do:
    






### Arguments about non-English speakers


One argument I (so far) find specious is that using capitals prevents non-native speakers from understanding the code.  I could maybe believe this is true in rare cases -- but (1) nobody has a study referenced that shows that, and (2) if that were true, someone simply has to learn the language better.  You can't argue that some people's inability to use a language is a reason to not use a feature of that language.  That would argue that almost all syntax in languages should be stripped, and few languages (and especially computer languages) can survive that.  Actually only Lisp survives that (give me just a pair of delimiters, and I can rule the world :-) )

Also given Ruby uses CamelCase for classes, you already have a requirement to understand this kind of syntax aspect.  So Ruby is presupposing the reader can read the syntax it is arguing the reader can't read.



### The standard, in different languages


A more interesting question is whether the "word and phrase standard" survives human-language change.  Obviously the standard does not work "as written" if the language used is not a capital-capable language.  And an obviously important language for that test is written Chinese (traditional or simplified).  But actually the 'underscores for phrases and parameters' degrades with Chinese better than 'underscores for words'.  Chinese doesn't require spaces for words to be apparent, and does not use spaces at all for that purpose (normally) -- it is obvious where each word starts and ends without spacing because a word is only a couple characters long.  So given words are apparent, the only thing left is to have phrases.  Although unnatural to use underscores for phrases and parameters, it at least seems plausibly useful.

Admittedly I have not tried in depth to program in Chinese (Smalltalk/Agents by QKS had the ability way back but I only played with it a little) and have not found a study on it...  so I can't say for sure whether the underscore for parameters would make sense.  But it is at least plausible it would make sense and it would make much more sense than underscore to separate (already separated) words.

Another plausible language to test against would be Hindi, but (I believe) most writers of Hindi in the computer field are also excellent English writers, so that is harder to argue for.



### Could be hyphens


Note that the convention is totally happy with using 'hyphens' as word separators if the environment supports it.


    
    
       insert-at_values
       for-poles-other-than_do
    





## Summary: Ruby has it wrong -- We need CamelWords or hyphen-words


So I believe Ruby has it wrong and am very unhappy a modern language started 20 years after Smalltalk and several years after Dylan would get this kind of thing wrong.  And I recommend moving the Ruby standard to the CamelWord_UnderPhrase standard, or allowing the hyphen and use the hyphen-word_under-phrase standard.




