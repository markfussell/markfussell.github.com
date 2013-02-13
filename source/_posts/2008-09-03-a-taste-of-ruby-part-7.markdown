---
comments: true
date: 2008-09-03 02:28
layout: post
slug: a-taste-of-ruby-part-7
title: A Taste of Ruby (Part 7)
wordpress_id: 140
categories:
- Rails
- Ruby
---


This is a multi-part series.  The first in the series is [here](/blog/a-taste-of-ruby).



## Tower of Hanoi &mdash; Rules on Rails


In part 6 we changed the algorithm of Tower Of Hanoi to work without requiring the stack -- it instead "more naturally" put sufficient state and logic into the model that the model could start and stop and continue as needed.  I believe with Ruby 1.8.x, this is required to have Rails be able to work with the Tower of Hanoi.  So we should try it out and see how well it works :-)

<!-- more -->


```ruby
require 'hanoi_7'

class HanoiController < ApplicationController
  def say
    @hanoi = session[:hanoi]

    if (@hanoi == nil) then
      @hanoi = createHanoi
      session[:count] = 0
      session[:consoleLog] = ""
    end


    @hanoi.initApp(self)
    @isDone = @hanoi.doNextMove_IsDone()

    #clear the app because otherwise it will be part of the attempted serialization
    #could also override the marshaling code, but this is simpler
    @hanoi.initApp(nil)

    #clear out the session variable, but @hanoi is still valid through
    #the rest of this interaction
    session[:hanoi] = @isDone ? nil : @hanoi

    @count = session[:count] ||= 0
    @count += 1
    session[:count] = @count

    if (@isDone) then
      render(:action => 'say_done')
    end
  end

  def appendLog(outString)
    logger.info("Append to Log: "+outString)
    @consoleLog = session[:consoleLog] ||= "\n"
    @consoleLog = @consoleLog + outString + "\n"
    session[:consoleLog] = @consoleLog
  end

  def noteChange

  end

  def createHanoi
    hanoi = RulesTowerOfHanoi.new
    hanoi.initHeight(5)
    hanoi.setup_disks

    return hanoi
  end

  def getAsciiTowers
    result = ""
    lines = Array.new
    @hanoi.stacks.each_with_index do | eachStack, i |
      0.upto(10) do | j |
        eachDisk = eachStack[j]
        if (eachDisk == nil) then
          newLine = " "*12
        else
          newLine = eachDisk.to_s
        end

        newLine << " "*(12 - newLine.length)

        currentLine = lines[j] ||= ""
        lines[j] = currentLine+" "+newLine
      end
    end
    lines.reverse.each {|line| result += line + "\n"}

    return result
  end

end
```


```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
<head>
<title>Foo <%= @page_title %></title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
<meta http-equiv="refresh" content="1">
<%= javascript_include_tag :defaults %>
</head>

<body<% if @page_class %> class="<%= @page_class %>"<% end %>>

<h1>Tower of Hanoi (Move <%= @count %>) </h1>

It's <%= Time.now %>.
<br/>
Done = <%= @isDone %>
<br/>
Render =
<pre>
<%= controller.getAsciiTowers %>
</pre>
Log =
<pre>
<%= @consoleLog %>
</pre>

</body>
```
