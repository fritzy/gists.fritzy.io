---
date: 2014-05-13 12:00:00 PST
title: Generating Reverse Sort Indexes for Riak in Node
tags: node.js, level, riak
type: text/html
---

While developing the prototype for what became [@quitlahok's](https://twitter.com/quitlahok) [riakdown](https://github.com/nlf/riakdown),
I quickly realized that while Riak can give keys back in lexical order of indexes, it cannot do the same in reverse.
We needed reverse support in order to support the [levelup](https://github.com/rvagg/levelup) interface.

Our solution was to generate extra indexes that would be the lexical mirror of each byte for each index given, as well as the key.
This creates double the indexes, but for our purpose was important.

In order to mirror the byte, I did a NOT operation on it, which is not straight forward in JS.
For each byte, we need to get the ascii value, and NOT that, but the ascii value is returned as a JS Number.
When we perform a NOT on a Number, we get an unexpected result, due to its signed nature.
Really, what we want is the last byte, so we mask it with AND 0xFF.

If there is a more direct way to do this in JavaScript, please [let me know](https://twitter.com/intent/tweet?screen_name=fritzy).

<script src="https://gist.github.com/fritzy/236b3c727264ded700aa.js?file=riak_reverse_put.js" type="text/javascript"></script>

## Update:

[@brycebaril](https://twitter.com/brycebaril) points out that things get weird with unicode characters,
and that we can reduce the number of operations by doing 255 - value, rather than NOT+AND.

<script src="https://gist.github.com/fritzy/236b3c727264ded700aa.js?file=buffer_reversestring.js" type="text/javascript"></script>
