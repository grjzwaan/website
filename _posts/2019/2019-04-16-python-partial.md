---
title: "Python Partial"
date: 2019-04-16T09:00:33+02:00
description: "A short note on Python partials and why it's awsome."
tags: ["Python"]
layout: post
---
This is a short note on the Python function `partial`.

<!--more-->

Partials are very common in functional languages such as [Haskell](https://www.haskell.org/) and [Elm](https://elm-lang.org/).

## Functional language
Suppose you need a function to color a line, it would map the position on the line (ranging from 0 to 1) to a RGB color like so
_(for convenience I noted the tuple as `RGB` instead of `(Int, Int, Int)`_):

```haskell
color :: Position -> RGB
```

But we do not want to define a function for, say, all gradients:

```haskell
blue_green :: Position -> RGB
blue_green p = (0, p * 255, (1 - p) * 255) + (0, (1 - p) * 255, p * 255)
```

We can now define a function:

```haskell
gradient :: RGB -> RGB  -> Position -> RGB
gradient l r p = (1 - p) * l + p * r
```

However we're in a bit of a pickle since our line coloring expects a function that maps positions to RGB. Functional languages allow is to simply only pass the first two arguments:

```haskell
color_function = gradient (0, 255, 50) (50, 0, 100)
```

This yields a function that maps positions to RGB and the functionality does not need to know how it was constructed.

## Python
Python has the `partial` function to do this.

```python
# Target signature and behaviour:
def color (position):
  return (R, G, B)

# Gradient coloring function
def gradient(p, left=(0, 0, 0), right=(255, 255, 255)):
  return tuple((1 - p) * left[i] + p * right[i] for i in range(3))

# Now get a blue-green coloring function
blue_green = partial(gradient, left=(0, 0, 255), right=(0, 255, 0))
```

Alternatively:
```python
# Target signature and behaviour:
def color (position):
  return (R, G, B)

# Gradient coloring function
def gradient(left, right, p):
  return tuple((1 - p) * left[i] + p * right[i] for i in range(3))

# Now get a blue-green coloring function
blue_green = partial(gradient, (0, 0, 255), (0, 255, 0))
```
