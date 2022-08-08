---
title: Data Engineering
date: 2022-08-08
categories: [Data Engineering]
tags: [data, data engineering, ETL, Streaming, backend]
---

I felt like writing something today but really had nothing interesting to write about.
To kill the urge I decided to write something about my current day job (well I do not
like it but it is unfortunately the only thing I think I smart enough to do). So, in
this article I am going to explain what I think data engineering is, maybe go on to
explain where it fits into software architecture, some of the techniques involved in
it, and whatever else comes to mind as I write.

## What is data engineering?

Data engineering can be summarised as an activity that involves extraction of data
from source systems and repackaging that data in such a way that makes it easy to
do all sorts of analytics on it. As a software developer (I am assuming you are)
you have probably built applications that did two things for your users. It solved
some particular transactional problem for them and then provided a way for the
users to extract some useful information from the application through reports.

Think of a stock management application, you have a transactional part to it that
is responsible for collecting information about what goes into inventory and maybe
overall facilitates the processes involved in moving items to and from the
inventory. These are clearly two separate use cases and they are formally called
[Online Transactional Processing (OLTP)](https://en.wikipedia.org/wiki/Online_transaction_processing)
and [Online Analytical Processsing (OLAP)](https://en.wikipedia.org/wiki/Online_analytical_processing)
respectively. When modelling the database for this stock management application you
will (unknowingly) optimise for one of these two use cases. For transactional
processing, you need your writes and reads to be fast. You do not want to keep
your users waiting for transactions to process. The transactions should at least
be as fast as the users physically move goods to and from inventory. In addition,
at this particular point in time users are not really interested in aggregate
information such as how fast is product X getting restocked/depleted. However, at the
end of the month the users might be interested in knowing how fast product X
is getting depleted/restocked so that they can come up with a decision on the
optimal time to restock the product. This is the analytical part and as you can
see, users are not looking for an immediate answer. You have about a month to
provide a response to this question (i.e. if you knew that the question was
going to be asked before hand). For a report, a user is willing to click a
button and wait for the system to process the data then provide an answer
after some reasonable amount of time (say 5 minutes...).

The nature of the queries made to the database for these two use cases are quite
different. For the transactional processing, the query is more row oriented.
Give me product X, its price and all other related data. On the other hand, for
the analytical processing you get queries that are more column oriented.
For product X, I need the total amount that was spent in purchasing it. That
question mostly focuses on aggregating a particular column over a number
of rows. Modelling data for optimal performance and ease of use for each of these two
use cases will lead you to two very different solutions. For efficient
transactions, you will likely end up with some highly normalised model that
makes extraction of rows from a database quite easy. And for the analytical
work, you will likely end up with something not very normalised that makes
column access quite efficient. This separation might even lead all the way
to the choice of database engine (column-store vs row-store). Of course,
for the simplest of applications, a single data model works. One feature of
highly normalised databases is that they offer a high degree of query
[pattern flexibility](https://www.alexdebrie.com/posts/choosing-a-database-with-pie/#introducing-the-pie-theorem)
which allows you to torture the data models with any kind of question. However,
what you get in flexibility is offset by a loss in either query performance or
database scalability. Inevitably at some point, as you get a lot of data or
the application gets significantly complex, you will need to have an additional
data model to specifically address your analytical needs.

Data engineering is a discipline that bridges the gap between the transactional
needs of an application and the analytical needs of the organisation the
application serves. Data engineers (this name is unfortunate, IMHO they should
be called data plumbers) are tasked with extracting the transactional data
and optionally combining with data from other sources to produce datasets that
can be easily analysed.

## Approaches to data extraction and processing

There are two general approaches that are followed in capturing and processing
of data from various source systems. The approaches are mostly informed by the
nature of the analytical needs and to some extent the architecture of the source
systems. Are you interested in realtime analytics (i.e. you need changes in the
source system to be immediately be propagated to your analytical systems or maybe
your analytics will be done at a later time)? Maybe you are interested in some
combination of the two? Is your source system built around an event driven
microservices architecture? The answer to these questions from a data engineering
is either streaming or traditional Extract-Transform-Load (ETL) or some combination
of the two.

### Streaming

In streaming, you basically attach to event stream and you process the events
as they come. Say, you are interested in reporting all error 500s that exceed
a certain threshold in a given amount of time. I mean you could say, we want
to know everytime a billion error 500s occur within a 5 minute period. In that
case you can attach to a log stream and check whether the total number of errors
that have occured in the last 5 minutes have hit a billion. You can then save
that data as a new dataset or simply trigger some reporting jobs.

There are a number of tools out there in the wild that allow you to do these
kinds of analytics. Of the top of my head I can think of
[Benthos](https://github.com/benthosdev/benthos),
[Spark Streaming](https://spark.apache.org/streaming/), and
[Flink](https://flink.apache.org/). Some of the features I look for when choosing
a streaming solution include windowing and sliding of those windows. A window is
just a period of time you want to focus on and a slide is all about how frequently
you need that window to be moved. For example, you could say I want to look
at data from the past 5 minutes (window) and I need this 5 minute window to
progress forward in time after 2 minutes (slide). So, what happens in that
case, is that the tool in use will buffer data for 5 minutes and make that
data available to you as a single dataset so that you can do your analytical
processing on it. These 5 minute windows can then be updated every two minutes
with new data (you essentially have a dataset with data ranging from about 2
to 7 minutes ago).

You were probably expecting to hear something to do with
[Kafka](https://https://kafka.apache.org/). Well, in my experience Kafka
makes for a good starting point for your streaming solutions. You can
think of it as a persistent log for your data streams. If you have
a continuous stream of data you would first have it write to Kafka then
you stream processing starts off from Kafka. In as much as you can directly
attach to a service bus of your event driven application, you probably will
want to buffer this data somewhere (that is where Kafka comes in).

### Traditional ETL

Forgive me for using the word "Traditional" here. Nowadays, the line
between ETL and streaming is kind of getting too blurred to warrant a
separate treatment of each of these techniques. Tools like Apache Flink
actually do handle both ETL and Streaming under the same framework.
Working in Flink makes you think "streaming-ish" even of you are working
with a data source that is not a stream. In ETL, you have a data source
that is not a stream for example an ordinary relational database.
So, you pretty much grab all the available in source then process that
data in a processing framework like [Spark](https://spark.apache.org).
You may join this data to other datasets from different sources and at
the end of all this you create new datasets.

This kind of workflow runs on a schedule. It could be something like,
every 3 hours grab all the data from the source system and then
process it somehow. Of course, you can include some techniques to
only capture and process the changes in the data. In general though,
you will be performing an extract, transform, and load at a given
schedule. Tooling used for this include ordinary cron and something
as sophisticated as [Airflow](https://airflow.apache.org/)

One of the main gotchas with ETL is that you could end up putting
a lot of stress on the source system with all the extraction work.
Data extraction is a resource intensive process and could result
in significant slowing down of the source system as the extraction
process hogs all resources. This can be mitigated to some extent
by setting replication to a different database and doing the extraction
from thereon. This does work to some extent but it is not always
the best solution. To work around this, it is quite common to resort
to reading the database transaction log directly and then processing
those logs in a style similar to stream processing. This reduces
the load on the source system and eliminates the need to perform any
sort of magic to determine what has changed in the data since the
last extraction process. There is some tooling that has been built
around this, examples include [Debezium](https://debezium.io) and
[Maxwell's Daemon](https://maxwells-daemon.io). I tend to separate
this from the other kind of ETL, thus I call the other kind of
ETL just "traditional" ETL to differentiate it from this. This is
still ETL but with a slightly different approach that is extremely
close to stream processing.

## Closing words

I have so much to say on this topic but I will leave it here now.
More will covered in other blog posts if I am up to it. Interesting
topics from here include data lakes, data warehouses, data lakehouses,
star schemas, OLAP cubes, and many more. I hope to cover these some
other day. What I have covered here should be enough to introduce
data engineering I believe.
