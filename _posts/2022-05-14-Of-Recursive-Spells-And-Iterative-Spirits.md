---
title: Of Recursive Spells And Iterative Spirits
date: 2022-05-14
categories: [Programming]
tags: [programming, functional, lisp, scheme, magic, witchcraft]
---

`Programs` are like magic spells, they conjure the spirits that roam the
digital world (the Matrix). We formally know these spirits by the name
`processes`. I mostly think of my spells as a sequence of steps that a spirit
needs to carry out but this is not 100% true. The spirits don't exactly follow
line by line what I ask of them. They merely take my instructions as guidelines
to what tune they dance. Of course, this is mostly all down to the mediums who
are responsible for translating my spells into a language the spirits
understand. The mediums sometimes do optimise my instructions for some selfish
reasons (saving time and space I am told) and sometimes there just isn't a one
to one translation of my instructions into the language spoken within the matrix.


## Out of the Recursive Spell Came the Recursive Spirit

Of particular interest to me at this point in time is a class of spells
that are called recursive. The general property of recursive spells is that
they somehow directly or indirectly invoke themselves. Consider the popular
spell 'factorial' that is presented in almost every recursion 101 lesson.
What this spell does is: given a positive integer N, multiply all numbers
from 1 to up to N. One of the ways of achieving this in a spell is to simply
start from N then multiply that N by the result of invoking the spell again
on N - 1 until N bottoms down to 1. This simple spell can be specified in the
language of the old patriarchs (achina
[Sussman](http://catb.org/jargon/html/koans.html#id3141241)) as follows:

```scheme
(define (factorial n)
  (if (= n 0) 1 (* (factorial (- n 1))))
 (factorial 5))
```

> The spell above is specified in a language called Scheme (a dialect
of Lisp). Went with Scheme here because it is very close to the language
the Spirits understand (I am lying through my teeth but work with me here,
you will soon see what I am on about). A Scheme spell is basically just
a list of lists which when pushed into the matrix turns to a spirit whose
purpose in life is to decay into a single scalar value by continuously
applying the first element of every list (the
[CAR](https://en.wikipedia.org/wiki/CAR_and_CDR)) onto the rest of list
(the [CDR](https://en.wikipedia.org/wiki/CAR_and_CDR) - pronounced Couldar)
to produce a scalar value or another list (that is more elementary
compared to the original - process repeats until this list becomes a single
atomic value).
{: .prompt-info }

Now, when the spell above is pushed into the matrix through a medium like
[s9](https://github.com/reflectionalist/S9fES), it produces a spirit that
whose life goes as follows:

```scheme
=> (factorial 5)
(* 5 (factorial 4))
(* 5 (* 4 (factorial 3)))
(* 5 (* 4 (* 3 (factorial 2))))
(* 5 (* 4 (* 3 (* 2 (factorial 1)))))
(* 5 (* 4 (* 3 (* 2 (* 1 (factorial 0))))))
(* 5 (* 4 (* 3 (* 2 (* 1 1)))))
(* 5 (* 4 (* 3 (* 2 1))))
(* 5 (* 4 (* 3 2)))
(* 5 (* 4 6))
(* 5 24)
=> 120
```

The thing to note about the spirit above is that overtime it grows in
space (ie the depth of the list grows first before it starts
to reduce). That's essentially what makes a recursive spirit (process).
We started with a recursive spell and we ended up with a recursive spell.

> In the real world, think C or similar languages, recursive processes
have this growth in space from using more and more of the stack. Every
recursive invocation pushes the stack down to make space for the local
variables of this new function invocation.
{: .prompt-tip }

## An Iterative Spirit from a Recursive Spell, you say?

Let's rewrite our recursive spell in a slightly different way:

```scheme
(define (factorial2 n product)
  (if (= n 0) product (factorial2 (- n 1) (* n product)))
 (factorial2 5 1))
```

This new factorial will decay as follows:

```scheme
=> (factorial2 5 1)
(factorial2 (- 5 1) (* 1 5))
(factorial2 (- 4 1) (* 5 4))
(factorial2 (- 3 1) (* 20 3))
(factorial2 (- 2 1) (* 60 2))
(factorial2 (- 1 1) (* 120 1))
(factorial2 0 120)
=> 120
```

Notice that in this iteration of the factorial spell, the original list we
started with did not grow in depth between recursive calls. This lack of
growth in space is characteristic of what is called is an iterative spirit.
We started with what is obviously a recursive spell but we somehow ended up
with an iterative spirit instead of a recursive one. We are able to generate
an iterative spirit from a recursive spell by specifying our spell in a style
that is called [tail call recursion](https://en.wikipedia.org/wiki/Tail_call).
You can loosely think of tail call recursion as a recusion in which the very last
operation in the spell is an invocation of the spell. Comparing our two
spells, we will notice that factorial's last operation is to apply
the * operator whilst factorial2's last operation is a call to itself.

> Not all mediums (compilers and interpretors) are smart enough to detect
a tail call recursion and optimise it into an iterative spirit. Most mediums
that are able to do this, reside in the functional world. Tail call
optimisation is necessary within the functional because the majority of
languages in that world have no looping constructs like for-statements.
All looping is done through recursion (even where a language provides
something akin to a for-statement, it's usually just a macro that turns
into a recursive call).
{: .prompt-info }

## Parting words

I am not really good at drawing conclusions but if anything the thing
you should take from this is:

1. Programs generate processes
2. Recursive programs usually generate recursive processes
3. Recursive programs written in a tail call style generate iterative processes
   if the compiler/interpretor in use supports it.
4. Iterative processes are more efficient in their use of space

## References

- [Structure and Interpretation of Computer Programs](https://en.wikipedia.org/wiki/Structure_and_Interpretation_of_Computer_Programs)
    > Been a long time since I last read this but I definitely do recommend
- [Sketchy Lisp](http://community.schemewiki.org/?Sketchy-LISP)
- [Wikipedia](http://wikipedia.org)
