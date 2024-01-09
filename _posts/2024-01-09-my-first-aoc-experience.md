---
title: My First AOC Experience [DRAFT]
date: 2024-01-09 12:00:00+02:00z
categories: [Programming]
tags: [programming, crystal]
---

Last year I decided to try out Advent of Code (AOC). I didn't get to finish the whole
thing. From what I managed to do though, I had fun and I believe that I levelled up a bit
in the language I used.

My initial goals for AOC 2023 were to build more familiarity with two languages:
Crystal and Scala. Crystal because that is what I have decided to make as my language of
choice for any personal experiments/work and Scala because ... well I professionally work
as a data engineer and Scala rules in data engineering to an extent. Both of these
languages I had some experience with but not enough to claim fluency. I was somewhat
skeptical that I would be able to meet this goal because of the thoughts I had about AOC.
I was expecting the time constrained and basic kind of questions that are mostly given
in a lot of online coding tests. In those type of tests, a short amount of time to solve
the problem is given and in most cases the problems themselves do not require coming up
with an overly elaborate solution (say one that requires some fancy data structure).
Under such constraints, I don't think there is enough leeway for me to explore what I can
do with a programming language. However, what I learnt from AOC is you can meet whatever
goals you have set out for yourself depending on how you approach it.

In AOC, you aren't time bound unless you go into it to compete. The problems can also be
complex enough to require some elaborate algorithms plus data structures. This gives a lot
of freedom for experimentation in whatever language you choose to go with. Anyways, I am
not trying to sell AOC to anyone, I just want to talk about some of the things I run into
that I thought were cool and maybe worth sharing (to a Malawian audience???).

> If you intend to do the AOC 2023 and don't want any spoilers, don't proceed.
{: .prompt-warning }

## Cycles ... Cycles ... Cycles ...

There were a couple of questions that involved some sort of cycle or loop.
You could be traversing some paths trying to find the shortest path among them. However,
some of them happen to be cycles. If you aren't careful you could find your program running
forever because it is locked in a cycle. Or you could have something like given these looping
paths of different lengths that have the same starting point, how many steps does it take for
all of them to converge on that starting point again. I am paraphrasing here but I hope you
get the point.

I found the first kind of problem a bit straightforward to handle. I would build a directed
graph as I traverse the possible paths. Cycle detection was happening at the point of
adding a vertex to the graph. I would check if the vertex and its edge already exist in
the graph. You can see an example of this solution
[here](https://github.com/kwalter94/aoc/blob/6bd92cd4383874cbe635b477a196032153ee5e65/2023/16/part-2.cr#L142).
I was representing my graph as a map of vertices to edges.

```crystal
alias Point = Tuple(Int32, Int32)
graph = {} of Point => Array(Point)
```
The `Point` is a vertex and then I have it mapped to an array of other `Point`s. Those
other points are essentially edges. I am not maintaining any weights or any other
attributes for each edge hence why I just have the `Point`. Having the graph as a map
makes looking up vertices very easy. There are other ways to represent graphs
(e.g. tree-like structures) but if you are going to be randomly looking up vertices
in the graph then a map should be one of the first base data structures to reach out for.

> Shout out to
[Floyd's algorithm](https://en.wikipedia.org/wiki/Cycle_detection#Floyd's_tortoise_and_hare).
I didn't get to use it but I had it locked and loaded, ready for firing after I saw
that cycles and graphs were a thing. It can be used for cycle detection and it is more
efficient than what I was doing above.
{: .prompt-info }


The other place where a cycle was involved was around something like finding the
convergence point of multiple looping paths that have the same starting point. The naive
way of solving this problem is to simultaneously traverse all paths until all paths
converge at the starting point. For the input that was I provided the solution ended up being
21,366,921,060,721 steps. There was no way I was brute-forcing my way to that answer. Try
writing a program that just counts from one to a trillion and see how long it takes to
complete. If you are feeling lazy, let's do some quick maths... Assume that it takes a
millisecond to increment your counter by one. To get to a thousand, you need a second.
Within a minute you would have done sixty thousand. To get to a billion you would need
277 hours. To hit a trillion, well about 31 years if my math is correct.

Of course, I didn't know the scale of the problem I was facing. I didn't know that answer
would be that high so I tried the naive way and I gave up after an hour or
something with my computer running super hot that ndikanatha kumayiwotha ngati mbaula
bwinobwino. I was totally lost on how I could solve this. I humbled myself and looked up
discussions on this problem on
[r/adventofcode](https://www.reddit.com/r/adventofcode/search/?q=Day%208&restrict_sr=1).
Turns out that some really smart people quickly figured out that the solution is
Lowest Common Multiple (LCM). Seems obvious I know, probably that's because of how I have
phrased the problem here but the actual AOC problem, wasn't so clear. You had to put in
some work to first figure out that there were cycles. The sizes of each of those cycles had
to be determined and then calculate the LCM of the cycles. Since I already knew that there
were cycles involved, I just wrote some basic code to find the cycle size. You can see how
I implemented
that [here](https://github.com/kwalter94/aoc/blob/6bd92cd4383874cbe635b477a196032153ee5e65/2023/8/part-2.cr#L52).

If you are confused as to why LCM is a solution to this problem, you just have to remind
yourself of what an LCM is. An LCM is the lowest number that a bunch of other numbers
can all divide without a remainder. The LCM is the inital point of convergence for all
numbers involved.

## Gat damn these floods!!!

Say you have to implement something like selection of all point enclosed by a path
created by the path tool in Gimp, how would you go about it? In other words, given
a closed path, you have to identify all the pixels that are enclosed by that path.
This took me a while to solve and I was able to get through it only thanks to ChatGPT
for reminding me of a super simple solution to the problem.

After overthinking the problem too much I opted to implement a
[flood fill](https://en.wikipedia.org/wiki/Flood_fill). With flood fill you more or less
colour the region contained within the closed path a different colour from the region
outside. You start with a cell (pixel), give it a colour and then move onto the next one
and give it the same colour if it's not on the path and so on... My idea was that I just
count the number of cells (pixels) with the inside colour when done doing the colouring.
I run into some issues with this approach. First, it was near impossible for me to just
look at the input I was given and say this is the inside of the loop and that's the
outside. Still, with this solution I should have still got the answer. I would have had
two values, one for the inside and another for the outside. But then there was a secon
problem, there were some hard edge cases that I failed to handle with my flood fill.
Trying to add a fix for the edge cases proved to be too much of a challenge so I caved
and asked ChatGPT what algorithm it would use to determine whether a point is within
a closed curve. ChatGPT was like "point-in-polygon."

My immediate reaction was like, "The f#$k???" I asked for more details and ChatGPT
described pretty much described
[Ray Casting](https://en.wikipedia.org/wiki/Point_in_polygon#Ray_casting_algorithm).
I can only blame years of CRUD for wiping this simple algorithm from my brain. This
is something anyone with a basic experience with computer graphics should be familiar
with. Anyways, I went ahead and implemented the solution. Handled a couple of weird
edge cases and I was home and dry. With ray casting, what you are effectively doing
is drawing a straight line through the image to a particular cell (pixel). Then you
count how many points on the line intersect the closed path. If that number is odd
then you have a point on the inside, otherwise it's on the outside. You can refer
to the diagram below for a visual representation:

    ........|............
    ........|............
    ...************......
    ...*....|.....*......
    ...*....$.....*......
    ...*..........*......
    ...************......
    .....................
    .....................

From the image above, the point being checked is marked `$`. As you can see there
is a ray that's been cast from the top of the image straight to it. It crosses the
path demarcated by `*`s at one point only. One is odd thus `$` must be within the
closed path. My implementation
[here](https://github.com/kwalter94/aoc/blob/6bd92cd4383874cbe635b477a196032153ee5e65/2023/10/part-2.cr#L85)
goes through every point on the image and checks if a ray cast as above crosses
the path at an odd number of points. There was a hard edge case I had to handle
for points that lie below turns (corners). Consider the following:

    ..............|......
    ..............|......
    ...************......
    ...*..........*......
    ...*..........*......
    ...*..........*......
    ...************......
    ..............$......
    .....................

If you naively do a ray cast you might end up with a value of 5. The ray "crosses"
the path at 5 points therefore you may wrongly come to the conclusion that the point
is within the path. I worked around this problem by keeping track of the contiguous
parts of the path above the point and counting them. If a contiguous path forms a
turn (ie. it turns back) then that counts as two, else as one.

    ..............|......  .............|.........
    ..............|......  .......................
    ...************......  ...***********.........
    ...*..........*......  ...*.........*.........
    ...*..........*......  ...*.........*****.....
    ...*..........*......  ...*.........$...*.....
    ...************......  ...***************.....
    ..............$......  .......................
    .....................  .......................
    Forms a turn == two    Doesn't form a turn == one


## Don't force matters or matters will force you

I was supposed to talk about brute force here koma ndatopa. Message to carry home
here is that brute force sometimes isn't the way. It can cause you a whole lotta
pain. I have touched upon this to some extent under nkhani yama cycles. Mwina someday
ndizakamba zambiri.
