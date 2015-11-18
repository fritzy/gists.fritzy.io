---
date: 2014-01-29 12:00:00 PST
title: Replacing Loading Images with Spinners Using Mutation Observers
tags: javascript, dom
type: text/html
---

Sometimes you'd rather images loading into your page don't show up until they're finished.
Sometimes you don't have direct control over when images get added to your page.

The [MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) is a handy API for reacting when DOM changes within an element.
It's commonly used in plugins, but sometimes it is handy for dealing with a central place for DOM changes, without having to mess with the script that actually changed the DOM.

In this case, it works pretty well for putting image placeholders in the place of loading images.

<script src="https://gist.github.com/fritzy/8700093.js"></script>

When I wrote my [StayDown](https://github.com/fritzy/staydown) library, it became apparent that images needed to handled as a special case when added to an overflow element that we're trying to keep scrolled down.
There was no event that fires when a loading image element changes size, so the library didn't realize it needed to scroll down until the image finished loading.
Loading a cached image in its place until loaded simplified the problem.
