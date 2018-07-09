---
title: "Objectgraph Tag Hack"
date: 2018-07-09T02:58:57-04:00
tags:
  - meta
categories:
  - software
  - blogging
ogimage: "/reference/seriously.png"
---
# Hacking in Facebook Object Graph Image tags for hugo posts

Just detailing that as of [this commit](https://github.com/arrjay/arrjay.github.io/commit/c3568fa29855a6e94c750cac4388fcd26d1b09f0) I added support for setting Facebook Object Graph preview-image settings in my Hugo templates. It was
really to add in the front matter, actually, once I realized you can refer to _anything_ in front matter under Params. The main tricky thing was FB wanted absolute URIs, so I had to use `absURL` (which I did, in the next commit).

And for no good reason, a silly QR code!

![are you seriously reading this](/reference/seriously.png "are you seriously reading this")
