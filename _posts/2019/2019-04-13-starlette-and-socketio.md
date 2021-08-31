---
title: "Starlette and Socketio"
date: 2019-04-13
description:  "A recipy on how to get Starlette + SocketIO working."
tags: ['Python']
update: '//posts/2019/06/13/starlette-and-socketio-improvements/'
layout: post
---
A short note on how to use [Starlette](https://www.starlette.io/) together with [Python-SocketIO](https://github.com/miguelgrinberg/python-socketio/). For a DIY project I want to run a single thread process to control a wake-up light (among other things).

## Starlette
[Starlette](https://www.starlette.io/) is basically an asynchronous version on [Flask](http://flask.pocoo.org/), which are both web frameworks. Personally I use Flask quite often and like the philosophy and resulting code.

Starlette seems to take inspiration from Flask and improves on speed. Additionaly there are some nice quality-of-life improvements (Websocktes, GraphQL). The asynchrounous part gives must easier in-process background tasks, for example sending an e-mail.

In their own words:

> Starlette is a lightweight [ASGI](https://asgi.readthedocs.io/en/latest/) framework/toolkit, which is ideal for building high performance asyncio services.

## SocketIO (and Python-SocketIO)
[Socket.IO](https://socket.io/) is a library on top of websockets

Python-SocketIO is a Python library that enables you to work with socket.io in Python. The name is pretty descriptive:

> This projects implements Socket.IO clients and servers that can run standalone or integrated with a variety of Python web frameworks.

## The problem
I wanted the following things:

* Starlette with a REST API;
* SocketIO (through Python-SocketIO) for realtime streams;
* A background task that periodially streams the current information in the system.

Additionally I wanted to have it in a single thread, and no heavy extra libraries. Such as distributed task queues ([Celery](http://www.celeryproject.org/)) which then require other moving parts.

Part of the reason is that I have a single state in memory of some hardware components. It being a single-threaded async application makes it very easy to argue about updating the state and reading the state.

I ran into some issues when wanting to combine Starlette+Python-SocketIO+background tasks. Starlette runs on Uvicorn and the current way how Python-SocketIO hooks into the async loop was not working anymore.

After some trial and error I got it working:
```python
import logging
import asyncio
import uvicorn
from uvicorn.loops.uvloop import uvloop_setup
from starlette.applications import Starlette
from starlette.responses import JSONResponse
import socketio

# Set some basic logging
logging.basicConfig(
    level=2,
    format="%(asctime)-15s %(levelname)-8s %(message)s"
)

# Create a basic app
sio = socketio.AsyncServer(async_mode='asgi')
star_app = Starlette(debug=True)
app = socketio.ASGIApp(sio, star_app)


@star_app.route('/')
async def homepage(request):
    return JSONResponse({'hello': 'world'})


@sio.on('connect')
async def connect(sid, environ):
    logging.info(f"connect {sid}")


@sio.on('message')
async def message(sid, data):
    logging.info(f"message {data}")
    # await device.set(data)


@sio.on('disconnect')
async def disconnect(sid):
    logging.info(f'disconnect {sid}')


# Set up the event loop
async def start_background_tasks():
    while True:
        logging.info(f"Background tasks that ticks every 10s.")
        await sio.sleep(10.0)


async def start_uvicorn():
    uvicorn.run(app, host='0.0.0.0', port=8000)


async def main(loop):
    bg_task = loop.create_task(start_background_tasks())
    uv_task = loop.create_task(start_uvicorn())
    await asyncio.wait([bg_task, uv_task])

if __name__ == '__main__':
    uvloop_setup()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main(loop))
    loop.close()
```

Perhaps there are other people running into the same issue and I can save them some time.
