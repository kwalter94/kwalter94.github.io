---
title: FAM On That Crystal Meth
date: 2023-07-27 18:00:00
categories: [Programming]
tags: [crystal, c, linux]
---

One thing that got me into [Crystal](https://crystal-lang.org/) is how easy creating C
bindings is in the language. Here is a quick example:

```sh
$ crystal init app libc_bindings

$ cd libc_bindings

$ cat <<-EOF > src/libstdio.cr
lib LibStdio
  fun puts(Pointer(LibC::Char)) : LibC::Int
end
EOF

$ cat <<- EOF > src/libc_bindings.cr
require "./libstdio"

module LibcBindings
  subject = "world"
  LibStdio.puts("Hello #{subject}")
end
EOF

$ crystal build src/libc_bindings.cr

$ ./libc_bindings
=> Hello world
```

Simple, right? In case, this went over your head, this is what is going on:

1. Initialised a new Crystal application with the name *libc_bindings*
2. Created a file that contains a definition of the `puts` function from *libc*
(I have omitted some linker flags that are supposed to be passed to the `lib`
declaration as an annotation because Crystal links to libc by default)
3. Created a file containing your every day Crystal code that calls the `puts` function
4. Build application
5. Run it

## What's with this FAM business then

Some 7+ months ago, I was trying to create a binding for
[inotify(7)](https://www.man7.org/linux/man-pages/man7/inotify.7.html) in Crystal and
I run into an area that's somewhat undocumented in Crystal. So, I needed to bind to the
following C `struct`:

```c
struct inotify_event {
   int      wd;
   uint32_t mask;
   uint32_t cookie;
   uint32_t len;
   char     name[];
};
```

Notice that the last field in the `struct` is a Flexible Array Member (FAM). Let's take
a short detour for those that don't know what FAM is. Normally in C, `structs` have
a defined size at compile time. So if you need to embed a string or something else with
a size that's not known at compile time in a struct, you define a field with an open
ended size (can only be the last member of the struct). When allocating memory for the
`struct` you add the size of the string you want to include in the struct:

```c
int main() {
  char *name = getStringFromSomewhere();
  struct inotify_event *event_p = malloc(sizeof(inotify_event) + strlen(name) + 1); // Don't forget the \0

  ...
}
```

FAMs are quite easy to wrap your head around in C (although their use in code is 9/10
times questionable) but when it comes to other languages that bind to C, they are a pain
and usually not supported at all (Golang doesn't in case you wondering). Crystal happens
to not have first class support for these pieces of shit too. I initially gave up on
creating the *notify(7)* bindings after running into this block. Later on though, after
mizimu yakumidima itandinong'oneza I came back to it. It was painful to get it working
given that my knowledge of C bindings in Crystal was mostly surface level (still is) but
I survived. Here is how I went about it...

I first had to define the `struct` like this:

```crystal
lib Inotify
  struct Inotify_Event
    # Crystal converts the name above to inotify_event
    wd : LibC::Int
    mask : UInt32
    cookie : UInt32
    len : UInt32
    name : LibC::Char
  end

  type Event = Inotify_Event # Alias the name to something nicer
end
```

Notice that I have defined the FAM field *name* as a `char`. So the size of this object
in Crystal would be `sizeof(struct inotify_event) + sizeof(char)`. If I try to access
the *name* (assuming *name* is allocated), I will get back the first letter of the
*name*. That isn't very useful, however we can use the address of the name field to read
the entire string out. I ended up creating a helper function to extract the name.

```crystal
module LibInotifyHelpers
  def self.extract_name(event_p : Pointer(LibInotify::Event)) : Pointer(LibC::Char)
    offset = offsetof(LibInotify::Inotify_Event, @name)
    address = event_p.address + offset

    Pointer(LibC::Char).new(address)
  end
end
```

The function first of all gets the offset of the *name* field (`@name`) into
`struct inotify_event`. This offset is then added to the address of the `struct` in
memory coming up a with a new address. The new address is used to initialise a
`char *`. This effectively gives us a string. Here is how I ended up using this all:

```crystal
def watch
  loop do
    event_p = Pointer
                .malloc(sizeof(LibInotify::Event) + FILENAME_MAX_LENGTH + 1, UInt8)
                .as(Pointer(LibInotify::Event))
    status = LibInotify.read(@file_descriptor, event_p, sizeof(LibInotify::Event) + FILENAME_MAX_LENGTH + 1)
    raise "Failed to read inotify event: errno(#{Errno.value})" if status.negative?

    event_type = parse_event_type(event_p.value.mask)
    filename = LibInotifyHelpers.extract_name(event_p)

    yield event_type, String.new(filename)
  end
end
```

Cool stuff, I know... but highly unsafe... Good for educational purposes only in my
opinion.
