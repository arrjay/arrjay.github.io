---
title: "Why I chose Hugo for setting up this blog"
date: 2018-06-22T14:56:08Z
categories:
 - "reasoning"
 - "software"
 - "blogging"
tags:
 - "meta"
 - "pipeline"
---

So this site is set up using [Hugo](https://gohugo.io) and I figure I'd detail a little bit of why.

# Static generator with a live updater

One of the easier ways to do hosting is to have a simple collection of files to upload to a webserver,
rather than building a server environment to run a blog engine. This also gives you the capability
to more easily move your site between providers.

However, not having an engine has drawbacks too - you need to track all your HTML snippets for a
consistent experience, if you maintain indexes update _those_, and so on.

The tradeoff with a static generator is you need to create your post content as input files, write
those somewhere and then run the generator. Hugo helps here with their live server mode, where you
can edit the input files for hugo to render on-the-fly and serve via a built-in HTTP server.

# Trivially portable, self contained

Hugo's decision to be written in [golang](https://golang.org) let them target a variety of operating
systems - I can use their binaries on Linux, OS X, Windows and Android. This gives me the flexibility
to not have a need for an environment specifically to run Hugo, I can just try it out wherever I
happen to be. It also means everything Hugo needs to run is contained within their binary. There are
no additional frameworks, servers, or dependencies I need beyond the blog source files.
