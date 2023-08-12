---
title: Crystal Has Java-like Interfaces
date: 2022-11-14 19:46:00
categories: [Programming]
tags: [crystal, oop]
---

I love Crystal; I think it gets right pretty much everything that irks me
about Ruby. And that speaks volumes because Ruby to me is just brilliant but
it's not something I would happily use in a large project. Enough about that,
here is what I want to keep on record here. Crystal is damn near a complete
language to me but I have always felt that having interfaces as in Go or Java
would certainly make certain things nicer. If you go through the official
Crystal guides like I did, you will be convinced that Crystal does not have
interfaces. To my surprise I just learnt (a couple of minutes ago) that Crystal
does kind of have interfaces and they work in the Java sense of interface.
I learnt about this through a
[two year old thread](https://forum.crystal-lang.org/t/proposal-interfaces-for-more-generic-code/2103)
on the Crystal forum where someone was asking for interfaces in Crystal like
we have elsewhere in the world of OOP. Two years later the thread got a
response and here is what I have learnt:

- Crystal doesn't have an explicit interface keyword (already knew this)
- Crystal has abstract classes and methods (knew this too)
- Modules can have abstract methods (oh shit - I have never even considered that)
- When modules with abstract methods are included in a class, the class must
  provide an implementation of the abstract methods
- When a class `includes` a module, the class satisfies the module's interface.
  In other words the class can be used where ever the module is expected (I
  do recall trying something like but not working out for me, let's see...)

Here is how you could go about defining an interface in Crystal:

```crystal
# Define the interface as a module with abstract methods
module Foo
  abstract def name : String
end

# Define a class and include the module/interface
class FooBar
  include Foo # Shades of Java's implements, eh???

  getter name : String

  def initialize(@name); end
end

# Define methods that consumes or produces the interface/module
def print_foo_name(foo : Foo)
  puts foo.name
end

foobar = FooBar.new("J.Random")
print_foo_name(foobar) # prints "J.Random"
```

There you have it... Interfaces in Crystal!!!
