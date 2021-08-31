---
author: "Ruben van der Zwaan"
title: "Starlette: A tale that was silently swallowed by uvicorns."
description: "Tracing weird bugs through the absence of stack traces."
date: 2019-08-13
tags: ['Python']
layout: post
---

Imagine the following bugreport of your Starlette application:

> Calling your API results 30% of the time in an error and 70% in correct behaviour.

and you **know** that these calls are time independent and should either always work or never work. Locally you cannot reproduce it, but the error can be traced to a datastructure being empty...which is filled during startup.

This is the offending code:

```python
@app.on_event('startup')
async def startup():
      logger.info("Executing the startup phase...")
      create_datastructure()
      logger.info('Ready to go!')
```
Locally you can never let `create_datastructure()` fail and it doesn't make any external calls (it reads a local file and processes it).

Going deeper: in production there were 4 workers and locally only 1, and 30% is suspiciously close to a quarter.

Turns out that errors during a startup hook are silently swallowed by Uvicorn and the worker _will still start_. Add to that unexplicable behaviours of AWS Elastic Beanstalk to randomly insert chaos during startup.

The annoying part is that AWS Elastic Beanstalk is very slow and the deployment (in my experience) is frail and prone to fail. It all takes quite a while and you need to do yet another deployment...

So, this is the new code to at least detect that AWS Elastic Beanstalk causes some error that Uvicorn than kindly sweeps under the rug. Now, it'll **fail**, as it should.

```python
@app.on_event('startup')
async def startup():
    try:
        logger.info("Executing the startup phase...")
        create_datastructure()
        logger.info('Ready to go!')
    except Exception as e:
        logger.error(str(e))
        raise e
```
