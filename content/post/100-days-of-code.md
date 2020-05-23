+++
author = "Chris Suttles"
categories = ["Python", "golang", "Java", "C", "automation"]
date = 2018-01-15T08:53:00Z
description = ""
draft = false
cover = "/images/2018/01/Screen-Shot-2018-01-14-at-4.52.19-PM.png"
slug = "100-days-of-code"
tags = ["Python", "golang", "Java", "C", "automation"]
title = "100 Days of Code"

+++


Recently, I started doing the [100 days of code challenge](http://www.100daysofcode.com/). I'm off to a slow start, since I am juggling more than usual in my personal life. If you're curious about my progress, you can see my [100 days of code repository on GitHub](https://github.com/csuttles/100-days-of-code).

### First steps

I'm revisiting Java and C fundamentals at the same time. I hope to improve my skills in both, as well as gaining some insight at a meta level on just "programming". So far I'm seeing a lot of similarities that I didn't see previously, even though the languages are different. Both languages share very similar primitives. The data types and flow control mechanisms are *very* similar. They are both strongly typed languages, and both can end up rendering negative numbers accidentally if you're not careful with your types. In both cases, this is because of signed variables being stored using [two's compliment](https://www.cs.cornell.edu/~tomf/notes/cps104/twoscomp.html). When a variable is accidentally stored in a type that is too small for the value, the the value can overflow the type, anding in a position that is using two's compliment to denote it as negative.

Here's a couple examples from the fountain of collective knowledge known as Stack Overflow:

* [two's compliment and negative integers in Java](https://stackoverflow.com/questions/13422259/how-are-integers-internally-represented-at-a-bit-level-in-java)
* [two's compliment and negative integers in C](https://stackoverflow.com/questions/3952123/representation-of-negative-numbers-in-c)

This is not a new thing, but it's a correlation I've never made before. In the past when working with a programming language, I would read stuff like this and move on, since it's usually not super relevant to the task at hand. In this way, my efforts in tackling a refresh on more than one language for [#100DaysOfCode](http://www.100daysofcode.com/) is already paying dividends.

### What About Twitter?

Part of the challenge is to tweet every day about what you are working on. I don't tweet very much. I decided when making the commitment to do this that I would not tweet about it every day. Using my time more productively and becoming a better developer aligns with my goals. More time on social media, especially *posting* to social media, has little return value for me, and doesn't align with my goals, so I'm skipping that part, or at least participating in that part more selectively.

### So Far, So Good

Despite my progress being slower than I had hoped and interruptions more frequent, I am making progress. I haven't written anything incredible yet, but I have been writing a lot more code, and putting in a lot more time, so I'll take that as a win.

So far I've written simple programs in C and Java, added some functionality to a Python program I borrowed from an old example, and then re-implemented the Python example using Go (although without the extensions I wrote). I also read a good portion of a C programming book that has been living on a shelf for far too long, despite my best intentions. I still have a long way to go, but I'm excited about what I will learn and the things I will build. If nothing else, it's been a reason to lean into personal projects and write more code, and a reason to turn off the TV a little sooner and spend my time more constructively.

