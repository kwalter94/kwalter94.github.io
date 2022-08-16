---
title: Why logger.debug() when you could just print()
categories: [Programming]
tags: [programming]
date: 2022-08-16 20:50
---

This is going to be a short one... `print()` statements and `logger.debug()`
statements achieve the same thing for the most part. They both print some
output in your programs to `stdout`. Using a `logger` does offer some
advantages over using bare `print()` statements.  With a logger you can
easily suppress some output based on a logging level and/or redirect the output
to a file instead of `stdout`, for example. This is all great but I want
to try and sell something else that's often not emphasised on using
a `logger`.

`print()` and `logger.debug()` may achieve the same end but
using each of those functions expresses some intent. A `print()` statement
in my opinion tells the reader of a program that this is output that the
program is built to produce. In other words, the output being produced at
that particular line, meets a functional requirement. A logging statement
coming from a logger on the other hand expresses that this is output that
is latent to the primary operations of this program. It is there not to meet
a functional requirement, rather it is meant to spill out the guts of the
program for diagnostic purposes. When I see a `print()` statement in code,
I zone onto it to see what it is printing out because I assume that the
output being produced by that statement is there to meet a functional
requirement. On the other hand, when I see a line starting with `logger`,
my brain happily ignores those lines as they are of no consequence.

I may have focused on logging here but this idea does apply to a number
of other things in programming in general. An example, A `while` and
`for` statement both do achieve the same thing but for certain kinds
of use cases, one may be better at expressing a programmer's intent
than the other. Keep that in mind... It is also a good idea to pick
up on various idioms in one's programming languages. Idioms are very
good at expressing intent to other seasoned programmers. An idiom makes
it easy for one to tell what a programmer's intentions are without
having to read every single line and grok the sum total of those lines.

PISS OUT!!!