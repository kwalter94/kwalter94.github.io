---
title: Don't Settle Too Early [DRAFT]
date: 2023-11-08 18:00:00
categories: [Programming]
tags: [programming]
---

A couple of days ago there was a debate on programming languages (technology in general).
in one of our local developer groups. The initial proposition that was made was kind of
something like PHP is a dead language. Everyone should switch to Python instead, well
... because AI (I don't agree with 99% of the arguments that were made in support of this,
story for another day though...). The debate progressed into a different topic; something
to do with specialisation. It was more around the idea of specialising in one or two
programming languages and more or less ignoring everything else. You don't need to learn
Rust or Go or [insert some language/framework/tool here] because:

  1. There is no demand for those technologies in Malawi...
  2. The kinds of problems we have in Malawi don't really need those technologies,
     you can easily solve our problems with PHP
  3. Your users don't care about the technology used in developing your service
  4. People only learn/use these technologies to show off to other developers

Whilst I do understand these arguments, I think this kind of reasoning could hamper a
person's growth as a software developer especially if the person is just starting out.
I also believe it's not good for our developer industry in general. Before proceeding, I
need to drop a disclaimer (people these days are easily offended - too many snowflakes
that take opposing views as a personal attack):

  1. I am not against specialisation... I am against the thinking that specialisation
     means ignoring everything not within the immediate scope of one's area of
     specialisation (e.g. Python is more than enough, I don't need to learn PHP...).
  2. Everything in this article are just my personal opinions. Most of these are coming
     from my experience and the arm-chair programming philosophy I do from time to time.

## A different language can make you a better developer at your language of choice

>> "LISP is worth learning for a different reason — the profound enlightenment experience
you will have when you finally get it. That experience will make you a better
programmer for the rest of your days, even if you never actually use LISP
itself a lot."
Eric S. Raymond - [How To Become A Hacker](https://www.catb.org/~esr/faqs/hacker-howto.html)

When I was starting my journey into programming, one of the first articles I went through
was [ESR](https:://www.catb.org/esr/)'s Hacker How-To. One thing I learnt from it was
that there are tools out there that I will learn for the sake of learning. I might never
use the tools themselves in any real life projects but the lessons I learn from those
tools will help me be better at whatever tool I stick to. This idea is summed up in the
quote above. You may agree with it or not but there is an element of truth in it.
There are things out there whose sole purpose in our lives is to teach us how to make
better use of the things we already have.

Primarily, I am a Python developer. I started programming in this language in 2010. Over
the years there are certain concepts that I now use that only became clearer when I looked
at them from a different programming language. The easiest example I can give (and I usually
do give) is on use of interfaces. For a time, interfaces were a concept that I had heard of
and could explain to an extent but I never really thought through as I was programming.
Interfaces didn't really factor into the decisions I made when structuring my programs.
I then learnt a bit of Java in something like 2012 or earlier. I started learning
Java mostly because I wanted to start contributing to Zangaphee Chimombo's
[jBawo](https://sourceforge.net/projects/bawo/) (I was passionate about board games, free
software, and programming - these were some of the best days of my life). Unfortunately,
I never got to contribute to jBawo beyond bug submissions. I didn't learn enough Java to
be writing it competently. Swing, AWT, and ma threads anandigudubuza sizibwana. I did
learn about interfaces though and I started to think about them a little more in the
design of programs in Python.
[Here](https://github.com/kwalter94/pybawo/blob/ff65f949a3625231e3c3e612373e8d8abe2cf018/baoAgent.py#L10)
(and [here too](https://github.com/kwalter94/pybawo/blob/ff65f949a3625231e3c3e612373e8d8abe2cf018/player.py#L17))
are examples of how that usage looked like. You can also see it below:

  ```python
  class BaoAgent(object):
      '''An interface used to communicate with a bao arbiter or player.'''
      def new_game(self, field, **kwargs):
          pass

      def message(self, message, from_=None):
          '''Send message to agent.

          from_ is the id of the original sender, if None then the message was
          sent by the arbiter
          '''
          pass

      def move(self, m):
          '''Send move made to agent.'''
          pass

  # And then I would later come up with implementations like:
  class Player(BaoAgent):
    def new_game(self, **args, **kwargs):
      pass

    def message(self, *args, **kwargs):
    ...

  class Malume(BaoAgent):
    """AI player"""
    ...

  class NetworkedPlayer(BaoAgent):
    ...
  ```

Whether this was good practice is debatable ... Python now has
[ABCs](https://docs.python.org/3/library/abc.html) and
[protocols](https://peps.python.org/pep-0544/) that kind of provide native support
for the same thing. Learning about how to properly use interfaces in Java gave me
an extra tool that I used in designing my Python programs. So, was learning Java a waste
of time since I never got to use it much? I don't think so... In my opinion, learning
Java made me a better programmer overall.

Later on I picked up Ruby, mainly out of peer pressure from madolo who were working at
BHT back then. These people made sure we know how beautiful Ruby was. I dove into it to
see this beauty that they spoke of and I can report back that it truly is there.
If you are just curious about programming languages, check it out sometime. One of the
things I learnt in Ruby was this concept called duck typing. Basically, duck typing can
be summarised by the phrase "if it quacks then it's a duck." Learning this made me
realise that I don't necessarily always need to be explicit about interfaces. I still
have to think about them but in a dynamic language like Ruby, I don't need to be super
explicit about them. I ended up going back to a style similar to what I was doing initially
in Python but with a lot more thought put into the structure of my programs.

> As I have grown older -- don't know if it's kukalamba chabe, I have come to prefer
statically typed languages. Even though I still use Python, I make heavy use of Python's
type annotations. I am a bit more explicit with my interfaces using
[protocols](https://peps.python.org/pep-0544/). Protocols provide
[structural subtyping](https://en.wikipedia.org/wiki/Structural_type_system#Description)
for Python type annotations. This to an extent is duck typing with some extra steps.
If you are familiar with Go's approach to interfaces, that's pretty much what protocols
are.
{: .prompt-info }

If there is one thing you can take from this is that you should not keep yourself from
learning more languages, frameworks, etc beyond what you primarily use. Some ideas are
easier to grok from a different perspective than the one your immediate tools give you.

## Exploring beyond different tools can help you see the limitations of your current tools

Most tools have communities around them. These communities tend to advocate different
approaches to problem solving. Take web programming for example: if I want to build
a web application in languages like Go, Erlang, or Clojure, chances are that I will
have to make choices about things like:

- What server should I use?
- What of a router?
- To ORM or SQL builder or just raw SQL?
- What HTML templating engine should I use?
- How am I running this application in production?

For I to be able to make competent decisions about any of these things, I need to
have an idea of what all those things are and why they exist. Having this knowledge
allows me to choose among various options and more importantly, I know what problems
these things are helping me avoid. I believe that the knowledge required for me to be
able to make such decisions makes me overall a better developer compared to how
I would be if I was never pushed to learn these things at some point.

Consider HTML templating engines, I know
that most templating engines help me avoid XSS issues in some way. Almost all
engines automatically escape HTML for me. To opt out of this I need to be explicit
about my intentions. The idea that XSS is a potential issue was only hammered down
into my brain because of my exposure to templating engines.

Of course, there are full blown frameworks in those languages that make these decisions
for you. However, you will mostly find that developers prefer making these decisions for
themselves. For

If you look at web development in Python on the other hand, you have to
choose a framework before you can do anything significant at all. Contrast all this with
web development in PHP. You don't necessarily need to make any of these decisions in PHP.
You don't need to choose what router you want or even pick a framework at all. Most people
do learn how to build web applications in PHP without ever touching on the items I have
listed above.

## Demand may simply reflect the lack of information not people's needs

... Ndimalizisabe ...
