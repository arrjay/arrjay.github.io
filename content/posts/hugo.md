---
title: "Why I chose Hugo for setting up this blog"
date: 2018-06-22T14:56:08Z
draft: true
---

So this site is set up using [Hugo](https://gohugo.io) and I figure I'd detail a little bit of why.

# Static generator with a live updater

One of the easier ways to do hosting have a simple collection of files to upload to a webserver,
rather than building a server environment to run a blog engine. This also gives you the capability to more easily move your site between
providers.

The tradeoff is you need to create your post content as input files for the generator - writing those as files somewhere and running the
processor. Hugo helps here with their live server mode, where you can edit the input files for hugo to render on-the-fly and serve via a
built-in HTTP server.
