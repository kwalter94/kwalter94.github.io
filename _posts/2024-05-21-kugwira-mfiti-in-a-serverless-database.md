---
title: Kugwidwa Ufiti in a Serverless Database
date: 2024-05-21 16:00:00
categories: [Databases, Data Engineering]
tags: [snowflake, databases, data engineering, data warehouses]
---

In late 2022, we started exploring Snowflake as an alternative to our existing
data warehouse solution at my usual place of work. Before then we were a
strictly Microsoft SQL server house. Sequel Server was adopted in the first
place because the organisation already had a lot of expertise around it.
Adopting it as a data warehouse was rather natural. It did serve the organisation
quite well for a number of years. Overtime the number of users of the warehouse
grew. At the same time, we were also producing more data than before. Total
number of datasets was closing in to over a thousand and some of these datasets
grew quite large, hitting hundreds of millions of rows.

Some of the problems we experienced included tables locking up because of
data pipelines trying to update the tables, whilst a number of users were trying
to access those same tables at the same time. Also the server would just crawl
to a halt because of too much, that was too heavy, happening.
We needed a change... From our exploration we ended up landing on
[Snowflake](https://snowflake.org). Snowflake could quite easily address all
the problems that we had. We did eventually migrate to it and I can tell you
that performance hasn't been a problem at all, again. Snowflake is blazingly
fast for analytics. We are able to do far more than we were able to do on SQL
Server with very little effort. The closest we had to performance problems was
this one time when we found ourselves executing a lot of queries (thousands) at
the same time, so Snowflake had to queue up some of these queries. The fix was
rather easy; with a click of a button we were able to scale up our warehouse to
handle the load.

Snowflake did solve our performance problems but it did come with its own
cruft. The costs of running Snowflake were rapidly rising by the day with
more and more adoption within the organisation. This increase in costs did
not necessarily come from pushing Snowflake to the max but rather inefficient
use of it. We basically just did a lift a shift and operation of our work
and how we worked on SQL Server to Snowflake. That ended up unnecessarily
costing us heavilly.

## Moving to a Serverless Database requires a Change of Behaviour

  > "Modern solutions require modern problems" (Anonymous Dude on the Internet)

Snowflake is a serverless data warehouse. There is no server for us to
manage in a sense. Snowflake just gives us a choice on how much compute
we need. We don't need to be experts on the internal workings of Snowflake
like we were required with SQL Server. We don't need to tune servers to
get the most out of them. I haven't even thought about where to put an
index in over a year now. It's brilliant, however there is a catch...
Costs in a serverless don't work the same as they do in an environment
where you manage your own servers.

When we pay for a server on [Cloud Africa](https://cloudafrica.net) for
example, what we get is exclusive ownership of the server for the term of
the contract we have gotten into. Serverless, on the other hand, doesn't
work quite like this. We pay for compute time, not a server. In other
words, we kind of pay for how much we do. When we own a server, we
can do as we please, so long as the server can handle it. On serverless,
we have to cough up more money to do as we please. And those costs do
rise rapidly.

On Snowflake, we buy credits which are time-bound. Credits can only
be used within a given time period. Credits translate to compute hours.
One credit is consumed for an hour of compute on the smallest warehouse
possible (instead of servers you think of warehouses on Snowflake). The
bigger the warehouse, the more credits consumed per hour. Here is the
catch, one hour of compute time does not necessarily mean that you have
done an hour of work. That just means the warehouse was up/available for
an hour. We can do more than an hour of work (multi-tasking) in that time
or less. We can end up paying for an hour of compute when we have rightly
performed less than a minute of work and this was what was happening to
us.

Here is what happens... When you send a query to Snowflake, a warehouse
powers up to handle your query. When done, the warehouse is done with the
query, it doesn't power down immediately. It waits for some time in case
more queries come in. We had this timeout set to 10 minutes initially.
The warehouse would then most times wait for a number of minutes for a
query that may or may not come, and it when it does, the 10 minutes gets
extended. In effect, we would could have a warehouse up hours on end,
doing nothing. We were paying money to literally do nothing!!! We did
eventually come to reducing the timeout time to the minimum possible
1 minute. This improved things for a while but time did catch up with
us again.

The root cause of the problem we had stemmed from the fact that we did
not change our behaviour moving from SQL server to Snowflake. When we
were working on SQL server, we would schedule our pipelines to run
whenever without thinking about it too much. For example, I would come in
and configure my pipeline to run every hour starting from now (time is
18:23 now). Somebody else comes in schedules their pipeline to run every
30 minutes starting from 18:31 and so on. Scale this behaviour to dozens
of individuals (we have data engineers, analysts, and scientists all
scheduling their ETL and analytical pipelines). Soon, we find ourselves
in a situation where the warehouse doesn't get to power down. On average,
we find that each of these individual is doing very little work. Maybe
just a query that checks if there is new data and it takes less than
a second to run but in effect costed us a minute of compute. To make
matters worse we had people building APIs and using Snowflake as their
database. There are two problems with APIs: 1) their traffic is not
patterned (akin to the scheduling of pipelines we were doing) and 2)
healthchecks (these can come in every second).

Our problem was not serverless per se, our problem was that we operated
in a serverless environment as if we were in a self-managed environment.
In a self-managed environment we can afford to be wasteful because we
don't really need to think about costs relative to what we were doing.
What we were doing only mattered when we tripped on each other. I am
slowing down the database for every because of a heavy that I am running
for example.

## Measuring User/Role/Query Impact on a Serverless database

When we figured out that our behaviour was the problem and the next step
was to be able to detect when someone was behaving like a savage. For this
we ended up using tools we already had in our utility belt but we ended
up applying them differently. Again as the behaviour thing, we couldn't
simply carry what we knew from SQL server over to this.

If something was going wrong on SQL server, we would look for the slowest
queries or look for a query that was running too many times in a day.
Looking at these two things independently worked on SQL Server but it
turned out to be useless on Snowflake for the most part. A query that
runs continuosly for 3 hours in a day could end up costing us less than
a query that runs for a total of 30 seconds in a day. A query that runs
once a day could cost us more than a query that runs 10,000 times in
a day. The runtime and scheduling of a query both mattered. It was never
an either - or situation. Sometimes our biggest cost would come from
the 3 hour query and sometimes it could be the query that runs 10,000
times depending on how it was scheduled. A query could run 10,000 times
within the same minute. The cost of that is less than that of a query
that runs continuosly for 5 minutes. At the same time if the 10,000
runs are spread out throughout a day, the cost would tilt the other
way. We needed a measure that could capture both of these situations. The
measure also had to be applicable to various elements like roles,
users, and queries.

We ended up on a measure we called relative cost (for lack of a better
name - English is not our first language, Chidzungu chinabwera pa boti
ichi). It combines both the number of times of a query is executed and
how long the query takes on each run. The scale of this measure is roughly
between 0 and 1 (it can exceed 1 but we don't care about that excess -
it adds no more meaning). The closer to 0 the value is then the lower the
impact on the data warehouse and the closer to 1 it is, then the user, role,
or query in question kept the warehouse up for about 100% of the time.

The logic behind the measure goes something like this:

- Walk every minute of an hour
- For each minute check if there is a query started by a particular
  user or role
- Sum up the each of these occurrences and total execution time
- Divide the total by 60
- NOTE: If a query executes in every minute for less than a second
  then after the summing up you end up with a value that's about 1.
  Same as if you only have a single query that runs for the entire
  hour.

The query we were running for this looks something like:

```sql
WITH roles_relative_cost (
  SELECT
    warehouse_name,
    role_name,
    user_name,
    TO_CHAR(start_time, 'YYYY-MM-DD HH:MI:00')::DATETIME AS minute,
    (DATEDIFF(MINUTES, MIN(start_time), MAX(end_time)) + 1) / 60 AS relative_cost,
    COUNT(query_hash) AS num_queries,
    SUM(execution_time) AS execution_time
  FROM snowflake.account_usage.query_history
  WHERE
    start_time BETWEEN DATEADD(DAY, -1, CURRENT_DATE) AND CURRENT_DATE
  GROUP BY ALL
)
SELECT
  TO_CHAR(minute, 'YYYY-MM-DD')::DATE AS date,
  COALESCE(warehouse, 'Unknown') AS warehouse_name,
  role_name,
  user_name,
  SUM(relative_cost) / 24 AS relative_cost,
  SUM(num_queries) AS num_queries,
  SUM(execution_time) / 60000 AS execution_time -- Express in minutes
FROM roles_relative_cost
GROUP BY ALL
ORDER BY 5 DESC, 6 DESC, 4 DESC;
```

The query above gives each role/user combo's relative cost. Onse with
a cost above 0.5 ndi athakati. By multiplying the cost to 24 we can
get a rough estimate of how long they kept the warehouse on. We have
this query and a couple of variations running in [n8n](https://n8n.io)
every morning to produce a reports showing the most expensive queries
and other information.

Jah man...
