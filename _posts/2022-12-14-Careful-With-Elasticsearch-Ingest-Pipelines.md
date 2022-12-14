---
title: Careful With Them Elasticsearch Ingest Pipelines
categories: [Databases]
tags: [databases,elasticsearch,couchbase,notes]
date: 2022-12-14 17:40:00
---

About a week ago I run into some mind boggling problem working
with [Elasticsearch](https://www.elastic.co/elasticsearch/).
Looking back at it now, it seems quite obvious but back then it was
not at all straight forward to figure out.

The problem stemmed from this: I have data in
[Couchbase](https://www.couchbase.com/products/server) that I need to get
replicated to Elasticsearch but unfortunately for me that data has some
fields like `_id` that Elasticsearch does not accept. So for a quick fix,
I decided to throw in an
[Elasticsearch Ingest Pipeline](https://www.elastic.co/guide/en/elasticsearch/reference/master/ingest.html)
which would remove the offending fields as the documents are getting
written to Elasticsearch. So I set up my
[replication agent](https://github.com/couchbase/couchbase-elasticsearch-connector)
with the ingest pipeline in place. A quick glance at the data led me to
believe that everything was okay. All the data had been replicated
and all new records coming from Couchbase were getting created in
Elasticsearch, LG. Time passes, and I realise that I have
significantly more documents in Elasticsearch than I have in Couchbase.
That's when all the fun began. I was completely lost trying to figure
out the cause of this disparity.

I sat down and looked deep into the data I had. First thing I
discovered was that the document `_id`s in Elasticsearch
were not matching the
[document keys](https://developer.couchbase.com/tutorial-document-key-design)
in Couchbase. To be frank, I really didn't know that they are supposed
to match, I just had a hunch. Secondly, I had duplicate documents.
On closer inspection of the duplicate documents, I discovered that
these documents although the same had some fields that were different.
Essentially, there was one original document and a number of documents
following that, that were just updates.

At this point it was becoming clear that the source of the problem was the
document key from Couchbase. Somehow Couchbase was not passing the
document key to Elasticsearch, hence Elasticsearch was not doing any
updates when it received an already existing document. There is no `_id`
specified thus Elasticsearch just creates a new record. Underneath, the
Couchbase - Elasticsearch replicator was sending bulk requests to
Elasticsearch. Those look something like this:

```
POST /_bulk?pipeline=<ingest-pipeline-name>

{"index": {"_id": "<document-key>", "_index": "<index-name>"}}
{ "_id": "some-value", "field1": "value", ... "fieldN": "value"}
{"index": {"_id": "<another-document-key>", "_index": "<index-name>"}}
{ "_id": "some-value", "field1": "value", ... "fieldN": "value"}
```
The ingest pipeline I had in place, was targeting the `_id` in
the body of the document and not the `_id` in the index command
(that's what I thought at least). If the `_id` in index command
is missing, that's when Elasticsearch creates a new document.
Went on a wild goose chase through out the
couchbase - elasticsearch connector code trying to find if
there was any chance of the document key getting omitted when
building the bulk request sent to Elasticsearch. Found nothing
of that sort, it was quite obvious the connector was not messing
up at all (points for Open Source Software - if the connector
was proprietary I would have probably convinced myself that the
connector was the problem).

Following the Sherlock Holmesian fallacy "when you have eliminated
the impossible, whatever remains, however improbable, must be the
truth", I came to the conclusion that the Elasticsearch Ingest
Pipeline was the problem. So, I set up some test documents in
Couchbase with no `_id` field and started replicating
them to Elasticsearch, with and without the Ingest Pipeline
in place. Sure enough, the Ingest Pipeline turned out to be the
problem. Lucky for me, the owners of the Couchbase data were okay
with removing the offending `_id` field. The field was a remnant of the
past that was no longer in use. Removing it would not result in death.
Happy ending in the end for all of us.

Still, I had one thing lingering on my mind that was unresolved. Why
was the Ingest Pipeline not working correctly? Turns out, the
Elasticsearch team
[have made it possible to access a document's metadata fields](https://www.elastic.co/guide/en/elasticsearch/reference/master/ingest.html#access-metadata-fields)
as part of the document itself within an Ingest Pipeline. For a field
named `foobar`, I can access it by just referring to `foobar` in
an Ingest Pipeline. With this feature, I can do the same with metadata
fields like `_id`. So, when the data was coming into the Ingest
Pipeline, the `_id` I had as part of the document, was getting
overriden by the document key I had as part of the `index` request.
In addition, modifying these metadata does affect the document's
actual metadata. So if I change an `_index` field's value, that
would effectively change the index the document is written to.
Strangely, although I had access to the source document through
the `_source` field in the Ingest Pipeline, modifying the
`_id` through the source document had a net effect that was exactly
the same as modifying the global `_id` (this looks to me like a bug
in the Ingest Pipeline - I should at the very least be able to
access the raw source document through `_source`).

Moral of the story, I think that would be
Read The Whole Fucking Manual (RTWFM). But even if I did, I am sure
I still would have had trouble figuring out what was going on.
I would have simply done something like
`rename _source._id to something_else` which still wouldn't have
worked. There is no easy work around for this as far as I know.

ASIDE: Writing this has made me realise that having a no-op Ingest
Pipeline would have worked. I am not sure if Elasticsearch allows this
though. Think about it...
