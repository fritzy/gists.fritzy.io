---
date: 2013-11-05 12:00:00 PST
title: MapReduce for LevelUp.
tags: node, leveldb, levelup
type: text/html
---

This is a Map-Reduce implementation for LevelUp.

I really like the map-reduce pattern for querying keystores.
I'm a big fan of Riak even though the Map part of Map-Reduce is database key specific.
I wanted to build a Riak-like embedded database using LevelDB, but instead have a proper Map-Reduce.
My work in progress is [DeputyDB](https://github.com/fritzy/deputy). `npm install deputydb`

This is a simplified version of what is in DeputyDB.
DeputyDB's MapReduce can take a pre-filtered list of keys, does everything atomically despite Node.js's event loop, and has an optional batch step at the end.

<script src="https://gist.github.com/fritzy/7322802.js?file=levelreduce.js"></script>

What makes this different from Riak's Map-Reduce is the ability to return any number of key/value pairs from the map step (0+).
You could return key/values that have nothing to do with the key itself.

For example, it could return word-counts or any derived pairs.
Or you could simple return a single [{dbkey: value}] for every key that matched a pattern.
Maps could also convert formats.
Reduce is then called to reduce together any duplicate keys emitted from mapping.

Map-Reduce is an incredibly flexible pattern.
