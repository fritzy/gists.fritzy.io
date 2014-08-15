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

```sql
CREATE TABLE document_store (key CHAR(100) PRIMARY KEY, bucket CHAR(100), value JSON);
```

Generally key-stores have GET, PUT, and RANGE queries. Let's make some functions for those.

Here's a put function based on Postgres's lack of a REPLACE query.

```sql
CREATE OR REPLACE FUNCTION documentstore_put(tname TEXT, bucket TEXT, key TEXT, value TEXT)
RETURNS VOID AS $$
BEGIN
    LOOP
        -- first try to update the key
        EXECUTE 'UPDATE document_store'
        || ' SET value = '
        || quote_nullable(value)
        || ' WHERE key = '
        || quote_literal(key)
        || ' AND bucket = '
        || quote_literal(bucket);
        IF found THEN
            RETURN;
        END IF;
        -- not there, so try to insert the key
        -- if someone else inserts the same key concurrently,
        -- we could get a unique-key failure
        BEGIN
            EXECUTE 'INSERT INTO document_store (value, key, bucket) values ( '
            || quote_nullable(value) || ', '
            || quote_literal(key) || ', '
            || quote_literal(bucket) || ')';
            RETURN;
        EXCEPTION WHEN unique_violation THEN
            -- Do nothing, and loop to try the UPDATE again.
            RETURN;
        END;
    END LOOP;
END;
$$
LANGUAGE plpgsql;
```

We don't really need a function for GET by key,
but let's do one anyway for consistency.

```sql
CREATE FUNCTION documentstore_get_by_key(key CHAR(100), bucket CHAR(100))
RETURNS TABLE(key TEXT, value TEXT) AS $$BODY$$
BEGIN
    RETURN query
    EXECUTE 'SELECT key::text, value::text FROM document_store '
    || 'WHERE key = ' || quote_literal(key);
    || ' AND bucket = ' || quote_literal(bucket);
END;
$$
LANGUAGE plpgsql;
```

The RANGE query is very similar.

```sql
CREATE FUNCTION documentstore_key_range(bucket CHAR(100), low CHAR(100), high CHAR(100), limit INTEGER)
RETURNS TABLE(key TEXT, value TEXT) AS $$BODY$$
BEGIN
    RETURN query
    EXECUTE 'SELECT key::text, value::text FROM document_store '
    || 'WHERE bucket = ' || quote_literal(bucket)
    || ' AND key >= ' || quote_literal(low)
    || ' AND key <= ' || quote_literal(high)
    || ' LIMIT ' || quote_literal(limit);
END;
$$
LANGUAGE plpgsql;
```

But what if we wanted to query by values in the JSON itself?
Well, that'd be pretty slow, UNLESS we made an index for the values we wanted to query by.

```sql
CREATE INDEX documentstore_value_lastname_index on document_store ((value->>'lastname'));
```

If you want to prevent duplicate values, you could use the `UNIQUE` restriction on the index.

```sql
CREATE UNIQUE INDEX documentstore_value_username_index on document_store ((value->>'username'));
```

Now we can query by the `lastname` field of the JSON without it having to scan the entire table!
`value->>'lastname'` generates an index for the literal, unescaped TEXT value of the field.
You should do this for every field that you plan on escaping.
It does incur a small cost on writes (as any atomic index system does).



```sql
CREATE FUNCTION documentstore_get_by_field(field TEXT, bucket CHAR(100), value TEXT, limit INTEGER)
RETURNS TABLE(key TEXT, value TEXT) AS $$BODY$$
BEGIN
    RETURN query
    EXECUTE 'SELECT key::text, value::text FROM document_store '
    || 'WHERE bucket = ' || quote_literal(bucket)
    || ' AND value->>(' || quote_literal(field) || ') = ' || quote_literal(value)
    || ' LIMIT ' || quote_literal(limit);
END;
$$
LANGUAGE plpgsql;
```

And again, a field RANGE function would be handy.

```sql
CREATE FUNCTION documentstore_range_by_field(field TEXT, bucket CHAR(100), low TEXT, high TEXT, limit INTEGER)
RETURNS TABLE(key TEXT, value TEXT) AS $$BODY$$
BEGIN
    RETURN query
    EXECUTE 'SELECT key::text, value::text FROM document_store '
    || 'WHERE bucket = ' || quote_literal(bucket)
    || ' AND value->>(' || quote_literal(field) || ') >= ' || quote_literal(low)
    || ' AND value->>(' || quote_literal(field) || ') <= ' || quote_literal(high)
    || ' LIMIT ' || quote_literal(limit);
END;
$$
LANGUAGE plpgsql;
```

You probably want to set up different tables for different object types to keep indexes from being full of junk.
As such, you probably want to add a `tablename` field into the above functions and escape it with `quote_ident` in your `EXECUTE` statements.

Using these functions are as easy as `SELECT`ing them.

```sql
SELECT documentstore_get_by_field('lastname', 'myusers', 'Fritz', 10);
```
