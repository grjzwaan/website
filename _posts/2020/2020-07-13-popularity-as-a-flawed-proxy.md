---
title: "Popularity as a flawed proxy"
description: "Popularity as proxy for success is probably a bad idea."
date: 2020-07-31T10:02:14.499+02:00
draft: false
tags: [Meta]
layout: post
---
Python is the most popular programming language for machine learning. So sayeth the internet... So you should pick Python to learn over other languages. But, is it? Should you?

## Bla bla bla
Most of these articles have no basis and combine some opinions on how Python code is _clean_, it's popular (notice the circular reference?) and that it has many libraries. Every update of an index (for example [TIOBE](https://www.tiobe.com/tiobe-index//)) spawns derivative articles how Python went up a notch, or Julia is posed to overtake it.

I think it's good to take a step back and ask yourself:

* How did they measure popularity? 
* Is this representative for my field?
* Do I actually care about popularity?

The aforementioned TIOBE index takes the number of results in search engines as input. Seems reasonable, but more is not always better. In my bubble you sometimes think that everyone has a blog, but there is a big world of enterprise software. In this setting there is less blogging and tweeting, but there are great engineers.

## Fundamentals, not popularity
Popularity might seem like a good proxy to choose a language to learn, but it's hard to gauge the real popularity. There are many echo chambers/bubles in which the gospel of X is spread. 

I argue that you should go _fundamentals first_. Yes, being fluid in a language helps, but being shitty at the basics results in shitty code. Enjoying the coding is also important, so if you enjoy Julia, R, Java or Swift more than Python: by all means, start with that.

## The future
Many reasons that these articles mention why Python is great might be moot. Pretty soon we'll get compiler infrastructure for GPU's and all the hand-tuned libraries in Pytorch and Keras will be available for all languages that target LLVM (even [Python does this through Numba](http://numba.pydata.org/)).

Swift allows you to [mix Python code into your Swift project](https://www.tensorflow.org/swift/tutorials/python_interoperability), so you don't even need to choose.

So pick Python because it's fun, beautiful or convenient because [a great course on ML](https://www.fast.ai/) is using it. Learn fundamentals. Then, also learn other languages a little to understand the differences.

## CV
All advice above is moot for applying for jobs. For this I would actually advice to learn all popular tools and have sample projets. You know what the recruiter/HR will ask ;)