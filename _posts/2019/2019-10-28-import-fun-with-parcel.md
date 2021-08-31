---
author: "Ruben van der Zwaan"
title: "Import fun with Parcel"
date: 2019-10-28
tags: ['NodeJS']
layout: post
description: "Bundling for NodeJS has ways to go before being comparable to compiled languages."
---
Today I was having fun with Parcel and generating a bundle for NodeJS. Very convenient, only importing seemed weird. Some imports were 'successful' in the sense that both Parcel and NodeJS could resolve it, but the import was `{default: {}}`. Hmmm.

<!-- more -->

I also write my thoughts on bundlers I used. Depending on your standards it might or might not be a review.

# Bundling

> A JavaScript bundler is a tool that puts your code and all its dependencies together in one JavaScript file.

There are many reasons you want this. One of the main reasons is if you develop a client side Javascript library. You actually need to serve all your Javascript and cannot rely on the client to have dependencies installed.

Further, most of these bundlers transpile sane Javascript to insane Javascript that is backwards compatible, and minify it. Notice that sane is relative here.

If you're stumped by these bundlers and why it seems a big deal, you're probably programming in Go or Java where the compiler actually makes bundles for you.

For NodeJS backend applications you could rely on the package information and get the dependencies when deploying or include them in your Docker image. But I do not like that.

Even if you're meticulous in pinning exact versions of your dependencies other could forget to update version numbers. Or, pull their packages from NPM and all your deployments/builds fail.

You argue that I should not be so cynical. Ok, I hope you at least stop pulling dependencies during deployment, but I'll go with it.

It's also wasteful. You're pulling huge amounts of dependencies and most likely you're only using a small fraction. For me it's no exception to have 50-100mb of modules but the compiled Javascript file is ~20kb. You can make deployment much faster and it's self-contained.

# Bundlers

I've used webpack, Rollup and Parcel. Webpack feels like making art with spaghetti and glue. It's about the process, not the result and only fun if you're not cleaning it up. I'm not biased against spaghetti and glue, but rather not have them in my code.

Rollup was pretty nice to use and I started using it because of Sapper and Svelte. Support for NodeJS bundles is lacking and Node Addons are not supported. I tried to make my own plugin for this but...one hour of weird errors and I bailed. Also because I seemed to include many plugins to make it work for NodeJS.

Parcel was working within 5 minutes and got me a nice NodeJS bundle, including the binaries in a Node Addon. Add a nice code generator and we're good to go! Or not. One weird error got my stumped.

Consider two imports of files with special characters:
```javascript
import * as route_1 from ('./routes/a/b/c/[slug].js')
import * as route_2 from ('./routes/a/b/c/<slug>.js')
```
The imports resolve and, yes, both files really, really, really exist.

Normally this works fine, but Parcel does something with it. The imports resulted in:
```javascript
route_1 = {default: {}}
route_2 = {get: [AsyncFunction: get], post: [AsyncFunction: post] } }
```

Huh.

I did find out what the underlying issue is (probably some regex stuff in imports), but even when escaping the square brackets it did not work. The point brackets worked, so I consider it solved.
