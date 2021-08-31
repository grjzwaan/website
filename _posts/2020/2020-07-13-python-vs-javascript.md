---
title: "Python vs Javascript for neural networks"
description: "Comparing Python to Javascript for neural networks based on features."
date: 2020-07-13T16:20:52.516+02:00
draft: false
tags: [Machine Learning]
layout: post
---

In this article I describe why I would prefer Python to JavaScript _specifically for neural networks_. 

This post is only based on actual features of the language and its design, not hand waving of 'popularity'.  Both Python and JavaScript are general programming languages. Both can accomplish any task and probably in comparable speed. So let's focus on the specific domain: neural networks. 

Through some simple programs you can see why Python could be appealing. This is based on my observations to program [auto differentiation](https://observablehq.com/@grjzwaan/building-autodiff-from-scratch) in JavaScript. 

The conclusions could be extended to other machine learning techniques. This is left as an exercise for the reader&trade;.

## The Premise
Programming languages bridge the gap between the real world and computers. Through the language we encode our ideas and hope it works as expected. If the code resembles how we think about a topic it makes it much easier. For example, anything can be made unintuitive if programmed in assembly. For many domains there are _domain specific languages_. Either full-blown programming languages or embedded. Haskell  and Swift are particularly great at this.

This will be the biggest difference between Python and JavaScript. Python is quite amenable to this process, and JavaScript is not - for logical historical reasons, I might add.

## Auto differentiation
Auto differentiation is a core component of neural networks and how to build them efficiently. To perform back propagation you need to (approximately) calculate the gradient.

The language of differentiation is _functions_ and _mathematical operations_. For example $f(x)=(x+6)*7$. All operations in neural networks are basic mathematical operations, most on tensors. Let's write code to represent a function in Python and Javascript.

## Operator overloading
In both languages we want an object of a class `Value` that represents the function for a particular input. As a building block we need to write down the computations we want to do. Visually the Python code looks nicer.

Python:
```python
x = Value(5)
y = (x + 6) * 7
```
In the code above `y` is actually a `Value`! Furthermore, we get the whole computation tree of intermediate values.

JavaScript:
```javascript
let x = new Value(5)
let y = mult(add(x, 6, 7)) 
```

This is because Python allows us to extend arithmetic on objects with custom functions by defining `__add__` on the object as follows: 
```python
class Value:

    def __init__(self, x, _deps=[]):
        self.data = x

    def __add__(self, other):
        other = other if isinstance(other, Value) else Value(other)
        return Value(self.data + other.data, _deps=[self, other])

    def __mul__(self, other):
        other = other if isinstance(other, Value) else Value(other)
        return Value(self.data * other.data, _deps=[self, other])

    def __radd__(self, other):
        return self + other

    def __rmul__(self, other):
        return self * other
```

## Objects as functions
Every neuron in the neural network consists of _weights_, _biases_ and an _activation function_:
```
neuron(input) = relu(weight * input + bias)
```

In most approaches to perform back-propagation its handy to also have some mutable state in the neuron. So it's both a function and an object.

In Python we can use an object as a function which results in this code:
```python
neuron = Neuron(5)
output = neuron([1,2,3,4,5])
```

For JavaScript we would need to 
```javascript
let neuron = Neuron(5)
let output = neuron.eval([1,2,3,4,5])
```


## List comprehensions
List comprehensions and generators are two features of Python that I really miss in other languages.

In Python we can multiply two arrays `A` and `B` by doing:

```python
result = [a*b for a,b in zip(A, B)]
```

In JavaScript you would need to do implement the `zip` function and do:
```javascript
result = zip(A, B).forEach(t => t[0]*t[1])
```

However, the `zip` function actively constructs the array of tuples instead of lazily generating it during the iteration. JavaScript actually supports generators but it requires more coding, because there is no natural way to map over them:

```javascript
function* zip(A, B) {
    let i = 0;
    while(i < Math.min(A.length, B.length) ) {
        yield [A[i], B[i]];
        i++;
    }
    return [A[i], B[i]];
}

let result = [];
for(let t of zip(A, B)) {
    result.push(t[0] * t[1])
}
```

## Wrap-up
Obviously you _can_ build neural networks in JavaScript. I hope that through these examples you see how much closer to reality you can get in Python and why people would prefer it.

However, there is a small downside to the Python code: to a beginner it appears as _magical_ and it's harder to find the place where the code is defined. For JavaScript it's explicit. 

