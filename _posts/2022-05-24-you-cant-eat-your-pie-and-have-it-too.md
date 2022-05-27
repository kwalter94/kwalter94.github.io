---
title: You Can't Eat Your Pie And Have it Too
date: 2022-05-24 19:00:00
categories: [Databases]
tags: [databases, distributed systems]
---

## Get your CAP on and take a walk with me

If you have worked long enough with databases or distributed systems, chances are that
you are familiar with the [CAP](https://en.wikipedia.org/wiki/CAP_theorem) theorem.
CAP is short for the following terms:

1. Consistency
    - All reads reflect the most recent writes regardless of which of partition of
      a system was written to and where the read has occured. NOTE: This is different
      from consistency in [ACID](https://en.wikipedia.org/wiki/ACID) of database
      transactions.
2. Availability
    - All reads are guaranteed to have data but they are not guaranteed to be consistent
      as above (that is, the data you get may not be of the most recent writes the
      system).
3. Partition tolerance
    - A system remains operational regardless of delays or errors in routing of messages
      between partitions. Partitioning here refers to
      [Network Partitioning](https://en.wikipedia.org/wiki/Network_partition) in case
      you are having trouble with the way this term is being used.

For more details refer to [this](https://en.wikipedia.org/wiki/CAP_theorem) Wikipedia
article. Basically, CAP theorem tells us that it's not possible to have a system that has
all three properties at the same time. You can only combine two of these properties in
any system. For example, in a distributed system you are forced to make one of two
choices: (1) you can only have Consistency and Partition Tolerance (CP) or (2) you can
only have Availability and Partition tolerance (AP). This is due to the fact that by
their very nature distributed systems have to be resilient to
[network partitioning](https://en.wikipedia.org/wiki/Network_partition).
The moment you go for a distributed system, you have implicitly made the choice of having
partition tolerance hence you are left with picking one thing between availability
and consistency.

There are however limitations to the application of CAP in distributed systems
especially in the world of databases. Availability is a
[very hard target](https://yokota.blog/2017/02/17/dont-settle-for-eventual-consistency/)
to hit even though the theory tells us otherwise. In theory, you can have a system
that's always available but in practice humans are highly error prone. They create bugs
like nobody's business and misconfigure systems for breakfast whilst existing in a world
where [ESCOM](http://www.escom.mw/load-shedding-updates.php) is running the show. These
factors are the main culprits when it comes to system downtime regardless of any
availability guarantees coming from your architectural choices.
In addition, CAP doesn't really reflect all the decisions people make when choosing
a (NoSQL) database. To a large extent, people look at what their usage patterns on the
database will be. For example, you may be thinking about things like "will I primarily be
doing reads on this thing and do I need an immediate response everytime?" It's that kind
of stuff that largely dominates these decisions. CAP in this case is not enough.

## Take a seat, remove your CAP (show some respect) and have this PIE with me

From the previous section, we know that CAP theorem may not be the best guiding principle
when making the choice of what NoSQL database technology to use. I didn't provide any
information as to why this applies to NoSQL database, let me explain a tiny bit (sorry
future self you used to be quite lazy - I suspect you still are). The existence of NoSQL
databases is to some extent from a need to enable easy horizontal scaling of storage. This
is in contrast to the traditional relational databases which mostly scale well vertically.
If you need more performance or more space, get a bigger box. You don't really have much
leeway in terms of adding more boxes. Thus CAP is something you will hear a lot of in the
NoSQL world.

An alternative theory called PIE was proposed in order deal with the shortcomings of
CAP. You could think of it as something that supercedes CAP but I prefer to look at
it as more of a complement. Now, let's look at why PIE may matter more than CAP...

1. Pattern flexibility
    - You can throw any kind of question to a database and you should be able to pull
      out an answer without modifying your data model. This is something you will
      typically see in a well normalised relational database. Databases with this
      property are well suited for both
      [transactional](https://en.wikipedia.org/wiki/Online_transaction_processing) and
      [analytical](https://en.wikipedia.org/wiki/Online_analytical_processing) uses.
2. Infinite Scaling
    - Need more storage and performance? Add more boxes to your cluster without limit
      (in theory). Your traditional relational database can't really do this or they
      can but the limits to this aren't that far away. You will mostly see databases
      with this property in the data lake/warehouse space.
3. Efficiency
    - A second is too long a wait for your query responses. You need your queries to
      be fast. Typical of transactional workloads and not too much of an in issue in
      analytical workloads. A report that takes 5 minutes can be acceptable, however
      having your users wait for 10 seconds just to fetch a list of their friends on
      your social network is a big NO.

With PIE as well, you have to make a decision of picking two out of those three
properties. Let's go through the various combinations of these properties.

### Pattern flexibility && Efficiency (PE)

PE is in my opinion the most common combination (at least it should be for most use cases
which are quite small scale). You want the fastest possible queries and at the same time
you want to be in a position to ask questions that you didn't have when you first came
up with your data model without making any changes to it. Typically, for you to achieve
this you will model your data in a relational way (I don't think we have much else better
than this) and even if you don't you will end up with a very general data model that's
packed in such a way that allows you to efficiently combine the various elements of your
data model in arbitrary ways. However, this will present a challenge when you try to go
past one box. Think of a situation where you need to perform something akin to an SQL
join on two datasets that span multiple computers. This is not easy, and it will come at
the cost of efficiency when done violating your objective in the first place.

Workflows that take advantage of this are mostly transactional and analytical in the
basic [Business Intelligence](https://en.wikipedia.org/wiki/Business_intelligence)(BI)
sense. The BI workflows here utilise
[OLAP cube-like](https://en.wikipedia.org/wiki/OLAP_cube) data models (eg
[Star](https://en.wikipedia.org/wiki/Star_schema)
and [Snowflake](https://en.wikipedia.org/wiki/Snowflake_schema) schemas.

### Pattern flexibility && Infinite scaling (PI)

From the section above, you have seen how difficult it is to scale beyond one box and
retain something resembling efficiency. However, in some cases you will be willing to give
up some efficiency to gain the ability to ask adhoc questions of your databases. This is
something that you will typically see in some kinds of analytical workloads. Data lakes
mostly fall under this combination. Data in data lakes is not organised in a way to allow
for efficient access. You may even find data in a data lake in different formats, some
in CSV files and some just raw JSON objects stored in some object storage. To get answers
out of this data you are pretty much looking at distributed computing tools like Spark
and Hadoop.

### Infinite Scaling && Efficiency (IE - not Internet Explorer, please)

To get IE, objects in the database have to be packed in such a way as to minimise the
need for complex computations when retrieving data. You in some way end up with a single
dataset unlike in the extreme case of relational databases where you have multiple
datasets (think of a table as a dataset). This enables you to partition this data across
multiple machines and have these machine independently operate on the parts of the data
they are storing. This structuring of data does however come at a cost. You may not be
able to easily answer certain kinds of questions without modifying your data model.
Workloads that take advantage of IE are mostly transactional in nature. Think of when
you have an application making a lot of writes but you also need to read back that data
as fast as possible. You would probably use something like MongoDB.

## I am tired now, I must end this

Choosing a database for your application isn't a simple thing. There are lots of things
you need to take into account. I have tried to explain some of these things in the
context of CAP theorem and PIE theory. These notes mostly are on NoSQL databases but there
are some drops of wisdom here and there on relational databases as well. So long...

## References

* [AWS re:Invent 2018: Building with AWS Databases: Match Your Workload to the Right Database (DAT301)](https://www.youtube.com/watch?v=hwnNbLXN4vA&list=WL&index=2)
    - I definitely do recommend this video, please watch if you want to learn a lot more
      on this subject.
* [Don't settle for eventual consitency](https://yokota.blog/2017/02/17/dont-settle-for-eventual-consistency/)
* [Why PIE theorem is more relevant than CAP theorem](https://www.alexdebrie.com/posts/choosing-a-database-with-pie)
* [CAP Theorem for Databases: Consistency, Availability & Partition Tolerance](https://www.bmc.com/blogs/cap-theorem/)
* [PIE Theorem Notes By Nikola Kovačević](https://nk-learning-notes.netlify.app/databases/theory/pie-theorem/)
    - More people should be doing this, publish your notes please...
* The billion Wikipedia links I have left in the article

