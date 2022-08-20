---
title: Of Streams (Enumerators and Generators) in Ruby and Python
date: 2022-08-20 10:00
categories: [Programming]
tags: [programming, data engineering, python, ruby]
---

In a past life I was once tasked with coming up with a tool for extracting
and transforming datasets of various sizes into datasets in a different
format. The source datasets could hit sizes of up to a couple of Gigabytes
and this tool would have to be used in environments where memory
was quite limited. At the time, data engineering was not something that
I nor the rest of the team I was working with were into. And to make matters
worse, it would have been difficult to introduce a new set of tools and
have them deployed in remote areas with no internet connectivity. So for
the task, I ended up using our language of choice, [Ruby](https://ruby.org).

> Ruby was available on all the machines this tool was to be run. These
machines were shipped to these remote areas with Ruby preinstalled.
{:.prompt-info}

The tool was quite straightforward to build but the initial implementation
was somewhat naive. It loaded the entire dataset in memory and then
processed it from there on. This worked for the smallest of datasets but
it obviously did not scale well especially in an environment where
memory came at a premium. So I altered the part of the program that was
responsible for extracting the source datasets to paginate the data.
At the same time I managed to keep the rest of the program unchanged.
I did this by utilising Ruby's
[Enumerator](https://ruby-doc.org/core-2.6/Enumerator.html) as follows:

```ruby
def load_dataset
  Enumerator.new do |enum|
    (0..).each do |partition|
      dataset = fetch_dataset_partition(partition, size: PARTITION_SIZE)
      dateset.each { |document| enum.yield(document) }

      break if dataset.size < PARTITION_SIZE
    end
  end
end

def process_dataset(dataset)
  dataset.each do |document|
    transformed_document = transform_document(document)
    save_document(transformed_document)
  end
end
```

> Code has been heavily simplified for easy presentation
{:.prompt-info}

The `Enumerator` allowed me to break the source dataset into multiple
smaller chunks but maintain the illusion of having a single data stream
to the rest of program. I successfully managed to significantly cut down
on memory usage but still maintain the rest of the code base as is.

Recently, I found myself faced with a similar issue but within the context
of a different programming language, [Python](https://python.org). In this
case, I was looking at a dataset that was partitioned at source. However,
the output dataset had to be one (i.e. not partitioned). There were also
some memory constraints but they were not as stringent as what I faced
in the Ruby days (I can always negotiate for a bigger server).
Each partition was reasonably large but not too big as to be impossible
to have all in memory. On the other hand, I could not have all the
partitions in memory at the same time because well ... memory. The
partitions all together are just too big to hold in memory. Faced with
this, I went back to that familiar pattern from Ruby and implemented it
in Python. Now, Python does not have an `Enumerator` class like Ruby does
but it has something else under the name
[generators](https://wiki.python.org/moin/Generators).

> Generators are not to be confused with generator expressions. They
are not the same thing.
{:.prompt-tip}

Generators in Python are simply functions that return a lazy iterator
to a stream of indefinite size. Python provides some simple syntax
that allows one to generate a generator. Instead of using the `return`
statement, one uses the `yield` statement in a manner similar to
`Enumerator#yield` in the Ruby code above. For the problem I was
working on, I ended up with code that looks something like the
following:

```python
def load_dataset() -> Iterable[Any]:
    for partition in get_partitions():
        dataset = fetch_dataset_partition(partition)
        for document in dataset:
            yield document

def process_dataset(dataset: Iterable[Any]):
    dataframe: dict[str, list[Any]] = {'field_a': [], 'field_b': []}

    for i, document in enumerate(dataset):
        if i % MAX_DATAFRAME_SIZE:
            flush_dataframe(dataframe)
            dataframe_buffer = {'field_a': [], 'field_b': []}

        row = transform_document(document)
        append_to_dataframe(dataframe, row)

    flush_dataframe(dataframe)
```

> Again the code above is heavily simplified to keep things simple
{:.prompt-info}

From the code above, notice that I have taken multiple partitions of
a dataset and combined them into a single dataset without needing to
load all the partitions into memory. Another thing to note is that,
with the exception of the extraction code the rest of the code is
agnostic of the nature of the data source. It just sees a stream
of data coming in and not really care whether underlying the stream
is a single data partition or a billion of them.

## Parting words

What I have shown here is just one way of handling this sort of thing.
You can easily do this differently and have code that quite easy to
grok. I am only providing this to maybe give a different perspective
on how to handle this problem using a set of tools you maybe have neve
 considered to take a closer look at. In my case, I tend to use these
 "streaming-like" solutions when I am faced with:

1. A large data source that would best be handled by paginating it
to minimise memory usage
2. A naturally partitioned data source that I want to treat as one
3. A potentially infinite data source that I have to poll at some
interval

Zipitani!!!