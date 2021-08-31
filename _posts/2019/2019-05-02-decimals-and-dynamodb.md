---
title: Decimals and DynamoDB
date: 2019-05-02
description: "A story on how I ended up adding ducktape code and putting it in production. It still hurts."
tags: ['AWS']
layout: post
---

A short note on [DynamoDB](https://aws.amazon.com/dynamodb/) of AWS and how it suprisingly gives `Decimals` when you extract records. And forced me to write ducktape-code.

## The story
DynamoDB implicitely converts integers to decimals. Decimals cause some problems in Python code since it is not a basic type. For example, `json.dumps()` doesn't work anymore.

1. You put a Python datastructure with lists and dictionairies into DynamoDB with strings and integers.
2. DynamoDB converts and stores the datastructure
3. You load an entry from DynamoDB
4. ...it suddenly contains `Decimals` instead of integers
5. ...and `json.dumps()` crashes (among others)

## The 'solution'
I now simply convert decimals back to `float` and `int`. The alternative of storing the object as a string (convert to JSON) I dislike. Figuring out how to configure DynamoDB...I couldn't find a decent way within half an hour. I did find a [discussion](https://github.com/boto/boto3/issues/369) on a feature request to disable decimals and/or find some sane way to work with it, but no solutions.

So I wrote a little function to convert a dictionary:

```python
def convert_decimal(dictionary):
    """ Converts decimals to float and int. """

    for key in dictionary.keys():
        value = dictionairy[key]
        if isinstance(value, Decimal):
            if v % 1 == 0:
                dictionary[key] = int(value)
            else:
                dictionary[key] = float(value)

    return dictionary
```
