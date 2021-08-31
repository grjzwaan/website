---
title: "Clojure"
description: "Watching Rick Hickey and trying out Clojure."
date: 2020-02-07T08:17:15+01:00
draft: false
tags: [Learning]
layout: post
---
Through the labyrinth of the internet I somehow ended up watching [Rich Hickey](https://www.youtube.com/watch?v=rI8tNMsozo0),  the creator of [Clojure](https://clojure.org/). Now and then the stars align and I navigate Youtube, which I found to be badly sorted and organized. 

It's a great talk and it made me want to try out Clojure. My goal was to learn [Swift](https://swift.org), but, ..., procrastination happened? I did a dozen or so exercises on [Hackerrank](hackerrank.com).

One of the exercises was to reverse a list, without using the built-in function. Here it started to be weird.

My first thought was to use the `reduce` function where you walk through the list and built up a new result. `conj[oin]` is used to return a new collection with the element added. 

Let's skip reading the [documentation](https://clojuredocs.org/clojure.core/conj) completely of course :)

Doesn't work:
```clojure
(fn [list]
    (reduce conj [] lst)
)
```

Works:
```clojure
(fn [list]
    (reduce conj () lst)
)
```

There is a difference based on the type `()` is a list, `[]` is a vector. And, as is stated **clearly** in the documentation `conj` works differently for each datatype:

* For lists it adds at the end, the underlying implementation is a singly-linked list. So, makes sense.
* For vectors it adds at the beginning 

Probably the best option is to use sequences and use [`cons`](https://clojuredocs.org/clojure.core/cons). But how that would exactly work...a quick try in the editor of Hackerrank just gave me stack traces that my  function definition was wrong. Copy-pasting an example from the Clojure docs and the problem persisted.
