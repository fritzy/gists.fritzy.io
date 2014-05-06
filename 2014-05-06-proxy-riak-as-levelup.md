---
date: 2014-05-06 12:00:00 PST
title: Create A Levelup Proxy to Riak
tags: node.js, level, riak
type: text/html
---

[Riak](http://basho.com/riak/) is a clustered key-value database that applies "AP" from the [CAP Theorem](http://en.wikipedia.org/wiki/CAP_theorem).
It has HTTP and Protocol Buffers APIs, but can be a little bit cumbersome to browse keys for debugging.
I've been getting used to [hij1nx's lev](https://github.com/hij1nx/lev) for debugging [levelup](https://github.com/rvagg/node-levelup) and thought it would be handy to use the same tool for Riak.

Since [Nathan LaFreniere](https://github.com/nlf) wrote [riakdown](https://github.com/nlf/riakdown), a Riak backend for level, it wasn't too hard. 
Essentially just mix [multilevel](https://github.com/juliangruber/multilevel) (for a network enabled levelup) with riakdown, and away you go!

<script src="https://gist.github.com/fritzy/ae8a8f3de86dbe842ce0.js"></script>

```sh
$ level2riak --bucket somebucket
Connected to riak://localhost:8087/somebucket
Listening for multilevel connection on: 8091
```

Now just load lev and save connection details to match the proxy.

```sh
$ lev
```

![lev connection config](https://i.cloudup.com/KJxW4Kn92E.png)

You can also use lev's other modes: cli and repl.

###But wait, there's more!

I turned this into an npm package.

Install:

```sh
npm install --global level2riak
```

You can then run it with:

```sh
level2riak --bucket somebucket
```

###Note

As of this writing, I've made an update to lev, but I'm waiting on hij1nx to publish to npm.
The manifest is no longer required, repl can handle remote connections, and I added --conns and --use for listing and using preconfigured connections.
