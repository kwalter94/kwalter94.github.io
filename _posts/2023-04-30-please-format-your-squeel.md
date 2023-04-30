---
title: Please Format your Squeel!!!
date: 2023-04-30 19:00:00
categories: [Programming]
tags: [sequel, sql]
---

This might be short... Mostly a rant on the lack of etiquette most developers
have when it comes to writing SQL.

There are two things that irritate me without end when it comes to reading
other peoples SQL:

1. Abuse of *ALIAS*
2. Omitting of a field's source table in the `SELECT`ed fields
2. Lack of or inconsistent formatting of code

## 1. Abuse of *ALIAS*

What do I mean by abuse of *ALIAS*? The majority of developers tend to use
`ALIAS` to shorten names of database objects to one letter names. I do not
fully understand why they do this but as far as I can tell, they do this
to save a few keystrokes. For example for a table named people, some
efficiency expert will alias the table name to __p__ to save himself the
trouble of hitting 5 letters on his keyboard. I do sort of understand this
reasoning, however I think the cost of doing this far outweighs the
percieved efficiency gains.

By doing this crazy aliasing you make the query harder to understand. Let's
look at an example to help me make my point:

```sql
SELECT
  p.name,
  p.dob,
  p.gender,
  a.city,
  a.village,
  a.district
FROM people AS p
LEFT JOIN address AS a
  ON a.id = p.person_id
```

When reading the query above, you start from the top. You look at the SELECT
followed by the list of fields being selected and so on. Now the first time
you get to the list of selected fields, you will not be able to make any
sense of what you are seeing without having any prior information. In the
query what exactly is `p.name` supposed to be? A name of a pharmacy or a name
of pants??? Of course, from the above you can guess that this is a name of
person but still my point stands. For you to make sense of what __p__ is,
you must scan ahead to the FROM part of the query to see what __p__ is.
From that point on you need to maintain in your head that `p = people`.
If there are a couple of more tables being joined to, then you have to
maintain a mapping of all those aliases and must continuously substitute
those aliases throughout the query. The query in itself might be complex and
for some super self serving reason you decide to add unnecessary complexity
to the query.

Imagine if we did this in every program we write (granted Go developers
do it, but those guys are nuts). Every variable is aliased to some short
one letter name. Reading code would become such a laborious task. We can
agree that most of us enjoy writing code not reading it. So let's try to
avoid making an already boring task even more painful than it already is.

### So where do I think aliases should be used?

I think aliases should be used to add more context to a query
or remove unnecessary context.

Adding more context to a query... Let's look at the following query:

```sql
SELECT
  employees.name,
  employees.dob,
  employees.gender
FROM people AS employees
WHERE employees.type = 'Employee'
```

In the above query, I am trying to select all people that are employees.
So by aliasing people to employees, I am effectively adding a bit more
context to every reference I have of people in my query. A reader of
the query sees employees and they understand what that is immediately,
without needing to read ahead and track back.

Another reason for aliasing is to remove unnecessary context. On this
reason, I am kind of on the fences but there are places where I think
it doesn't really do any harm. Consider the following query:

```sql
SELECT
  people.name,
  people.dob,
  people.gender
FROM human_resource__people AS people
```

In the query above, there is this funny namespacing of tables
that's going on. If you have used something like Django then you have
probably run into this. Basically a Django project is an umbrella
project to which you add a bunch of small applications (kind of bounded
contexts in Domain Driven Design). Each application is namespaced in the
database. So multiple applications can have a domain model with the same name.
Now in the query above we are simply interested in people from the
human_resources context. Continuously repeating that fact in the query
does not really add much value. If you are looking at that query in
an application then that fact must be implicit in the name given to that
query.

## 2. Omitting of a field's source in `SELECT` blocks

It's quite common for people to omit table names when referencing field
names in a query. An example:

```sql
SELECT
  name,
  dob,
  gender,
  city,
  village,
  district
FROM people
LEFT JOIN address
  ON address.id = people.person_id
WHERE
  primary = TRUE
```

The query above is selecting fields from two different tables. You can't
simply look at that query and easily tell where any of the fields are
coming from. Is city from people or from address? And to make matters worse
there is a field called `primary` that is being reference in the `WHERE` part
of the query. Where the hell is that field coming from? Need I say more?

My general recommendation on this is to never omit the source table names
whenever a query directly references two or more tables. If you have
subqueries in conditions and that kind of stuff, this really doesn't apply.
The only time you are free to omit the table names is when you only have
a `FROM` that references one table without any `JOINs`. The context in such
cases is unambiguous, no need to remind me of it.

## 3. Lack of or inconsistent formatting

This one bugs me a lot... Everytime I see it I tend to question the sanity
of the person that was writing the query. It's a clear lack of organisation
that can only come from being brain damaged. Let's start with an example:

```sql
select
people.name
,people.dob
,people.gender, address.city, address.village,
address.district from people left join address on address.id = person.person_id
left join identifiers
on identifiers.person_id = people.id
where people.type = 'Employee'
```

Reading code generally relies on visual cues. It's rare for anyone that has
been programming for a while to read code line by line. You do go line
by line when you want to zone in on a particular section of the code but
for the rest it's mostly scanning of the code and quickly identifying patterns
in the code (this is why idioms matter in programming). Going to the query
above it's very difficult to identify different sections of the query as
it is. Say for example you want to add a new `JOIN` to the query, you can't
simply jump to a specific part of the query and make the change. You need
to read every single line of that query to identify where you need to make
the change (granted you can just search for `join` and add your thing right
before it). Combine this with the other two problems mentioned in this
article, you have a serious problem on your hands.


Now let's format the query and see how it improves things:

```sql
SELECT
  people.name,
  people.dob,
  people.gender,
  address.city,
  address.village,
  address.district
FROM people
LEFT JOIN address
  ON address.id = person.person_id
LEFT JOIN identifiers
  ON identifiers.person_id = people.id
WHERE
  people.type = 'Employee'
```

Better, isn't it? You can easily identify different blocks in the code.
Even if you aren't familiar with this style, you can easily jump to
a specific part of the code and make changes. That's the style I use
but you are free to go with one that makes the most sense to you rather
than nothing at all.

A bit of an explanation of the style I go with:

- Use indentation to indicate that a block directly supports the
statement above. The block is more of a qualifier for the statement above.
So if you aren't interested in the statement above, you can happily ignore
the entire indented block that follows it.
- Use CamelCase or snake_case for identifiers
- Uppercase all SQL keywords to make them stand out from identifiers
- In `JOIN` conditions always lead with the identifiers from the table being
joined to. That is:

```sql
LEFT JOIN address
  address.value = some_other_table.value

-- Not

LEFT JOIN address
  some_other_table.value = address.value
```

I have more rules that I follow in my queries but these are the primary
ones and they help me easily make sense of my queries. As a bonus,
most times when I deviate from this style it indicates to me that I am
doing something wrong.
