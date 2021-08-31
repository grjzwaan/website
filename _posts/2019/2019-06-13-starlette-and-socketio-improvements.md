---
title: "Starlette and Socketio Improvements"
date: 2019-06-13T10:51:03+02:00
tags: ['Python']
layout: post
description: "A note how you can combine Starlette, Uvicorn and SocketIO in the same loop, respecting SIGINT."
---
This is a followup of the previous article on using  [Starlette](https://www.starlette.io/) together with [Python-SocketIO](https://github.com/miguelgrinberg/python-socketio/) and background processes.

I've made some improvements, especially to handle SIGINT (`ctrl`+`c`) properly. Additionally, I wanted to start background processes without waiting for a client to connect (this is a solution I saw online).

The core problem was that all async tasks should be on the same loop. The solution took some digging around uvicorn internals, but the following worked:

* Get the Uvicorn server as an awaitable;
* Join the awaitable with your background tasks and return when the first completes
* Run the composed application.

This properly responds to SIGINT and all processes start without waiting for outside interaction.

Background tasks are more like _workers_ here. One-off tasks fall in two categories:

* Startup or shutdown: Use Starlette hooks for startup and shutdown to handle work there;
* Based on interactions from outside: Just call tasks when an API is called.


```python
import logging
import uvicorn
from app import app
import asyncio
from uvicorn.loops.uvloop import uvloop_setup


logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)-15s %(levelname)-8s %(message)s"
)


def uvicorn_task():
    """ Returns running the app in uvicorn as an awaitable to join the main
        asyncio loop. """
    config = uvicorn.Config(app, host='0.0.0.0', port=8000)
    server = uvicorn.Server(config)

    return server.serve()


async def main(app):
    """ Joins unicorn together with background tasks defined in the app. """
    # Make the list with awaitables
    aws = [uvicorn_task(),
           *app.background_tasks()]
    # Run and return when the first completes or is cancelled
    await asyncio.wait(aws, return_when=asyncio.FIRST_COMPLETED)


if __name__ == '__main__':
    # Set up the loop
    uvloop_setup()
    loop = asyncio.get_event_loop()

    # Run the main loop
    asyncio.run(main(app))
```
