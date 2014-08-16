---
date: 2014-08-14 12:00:00 PST
title: Using Postgres as a Document Store
tags: postgres, database, dulcimer
type: text/html
---

Lately, I've been working on [pgDOWN](https://github.com/fritzy/pgdown), which is a Postgres backend for levelup, so that I can support Postgres in [Dulcimer](https://github.com/fritzy/Dulcimer).
My motivation is to have a CP backend for Duclimer.
Currently leveldb (levelDOWN) works great for an embedded backend, and Riak for an AP backend, but I we have needs at [&yet](https://andyet.com) for a CP backend.

Postgres has some handy features, which can make it very nice as a document/key-value store;
Namely the JSON and HSTORE data types.

<script src="https://gist.github.com/fritzy/7f7716d11bb51a059d88.js?file=create_documentstore.sql"></script>

Generally key-stores have GET, PUT, and RANGE queries. Let's make some functions for those.

Here's a put function based on Postgres's lack of a REPLACE query.

<script src="https://gist.github.com/fritzy/7f7716d11bb51a059d88.js?file=documentstore_put.sql"></script>

We don't really need a function for GET by key, except it's nice to be injection safe.
but let's do one anyway for consistency.

<script src="https://gist.github.com/fritzy/7f7716d11bb51a059d88.js?file=documentstore_get_by_key.sql"></script>

The RANGE query is very similar.

<script src="https://gist.github.com/fritzy/7f7716d11bb51a059d88.js?file=documentstore_key_range.sql"></script>

But what if we wanted to query by values in the JSON itself?
Well, that'd be pretty slow, UNLESS we made an index for the values we wanted to query by.

<script src="https://gist.github.com/fritzy/7f7716d11bb51a059d88.js?file=documentstore_create_index.sql"></script>

If you want to prevent duplicate values, you could use the `UNIQUE` restriction on the index.

<script src="https://gist.github.com/fritzy/7f7716d11bb51a059d88.js?file=documentstore_create_unique_index.sql"></script>

Now we can query by the `lastname` field of the JSON without it having to scan the entire table!
`value->>'lastname'` generates an index for the literal, unescaped TEXT value of the field.
You should do this for every field that you plan on escaping.
It does incur a small cost on writes (as any atomic index system does).

<script src="https://gist.github.com/fritzy/7f7716d11bb51a059d88.js?file=documentstore_get_by_field.sql"></script>

And again, a field RANGE function would be handy.

<script src="https://gist.github.com/fritzy/7f7716d11bb51a059d88.js?file=documentstore_range_by_field.sql"></script>

You probably want to set up different tables for different object types to keep indexes from being full of junk.
As such, you probably want to add a `tablename` field into the above functions and escape it with `quote_ident` in your `EXECUTE` statements.

Using these functions are as easy as `SELECT`ing them.

```sql
SELECT * FROM documentstore_get_by_field('lastname', 'myusers', 'Fritz', 10);
```

Let me now on [@fritzy](http://twitter.com/fritzy) Twitter if you have any questions, fixes, or comments.
I currently have availability for consulting work through [&yet](https://andyet.com).
