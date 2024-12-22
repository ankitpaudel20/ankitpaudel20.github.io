---
title: "Introduction to WSGI and gunicorn"
description: "Tldr and intro to gunicorn so that you can know what actually is going on when something strange happens in production" 
date: 2024-12-22T11:41:14Z
toc: true # Controls if a table of contents should be generated for first-level links automatically.
image: 
math: 
license: 
hidden: false
comments: true
# draft: true
categories:
  - Python
---
<!-- 
Outline:

# Webserver in Python
* How does python serve requests.
* Sockets api form c
# What is WSGI
* why was wsgi needed while other programming language does not need anything much to deploy in production.
* A simple wsgi application.
## Flask and other frameworks
* WSGI, the commonground between flask, django and other sync web frameworks.
* What makes flask wsgi application.
## Gunicorn
* What it is and why is it needed? its a wsgi server with other features like processmanagement.
* Design philosophy: what are workers? how it can be extended? Some default worker types. -->


# Webserver in Python

- Originally, python was not made to develop webservers, but due to its simplicity and very easy learning curve, along time a few web frameworks started popping.
- Different frameworks/tools had their own serving method that tried to alleviate the fact that python was a single threaded application and io calls would block the application from receiving any further requests.
- Solution: use threads, multiple python processes and combination.
- This issue led to development of some standard interface that all frameworks can adhere to so that frameworks don't have to worry about how to manage concurrency and be best on what they do, acting as an easy and fast tool to create a httpserver.

# What is WSGI

- WSGI is the interface that most of the python web frameworks provide for their application to be served without worrying about the specifics of concurrency and load management in high rps environments. At the core, its just set of documents that describe how a compatible web application should be structured, process the request and respond accordingly.
- A simple hello world WSGI application looks like this:

```
def application(environ, start_response):
    method = environ['REQUEST_METHOD']
    print(method)
    path = environ['PATH_INFO']
    print(path)
    status = '200 OK'
    response_headers = [('Content-Type', 'text/plain; charset=utf-8')]
    start_response(status, response_headers)
    return [b'Hello World!']

```

- This application can be ran using gunicorn which is a wsgi server designed to serve these kind of applications. The simplest command to do so would be `gunicorn app:application`.
- So all the flask or Django applications can be accessed as a simple function above. All the ifs are abstracted away in different ways in different frameworks, decorators in flask and functions/methods in django.

# Gunicorn

- As per gunicorn's documentation, gunicorn is inspired from ruby's unicorn project which used to manage multiple ruby web-server processes.
- It essentially is a process manager that kills/restarts hung processes that is supposed to listen to requests.
- When gunicorn starts, it starts its main process called "arbiter" which listens to some port. Then it gets forked into multiple "worker" processes that inherit the socket that was opened during the startup of arbiter thus, multiple processes would be listening to same os port.
- The "worker" that I mentioned previously is a python process that runs the WSGI compatible web server.
- As multiple workers are listening to a single port, the role of load-balancing is handed over to kernel. The kernel manages the resource contention between multiple "worker" processes for the opportunity to handle the request.
- This disproves the fact that would be plaguing people's mind assuming that arbiter acts as some sort of reverse proxy like nginx.
- The "management" of "worker" processes by arbiter includes things like restarting workers when workers gets hung up.
    - "hang up" in this context means that the worker is not able to message arbiter saying its okay for some period of time. This is somewhat like linux's "watchdog" concept.
    - Async workers can send periodic heartbeats but sync workers can't do that if they are stuck in some I/O operations. In this case, arbiter relies in messages in specific occations like acknowleding the request, returning response and likes.
- Sends SIGKILL to timed out workers which gets logged as log somewhat like this: `[CRITICAL] WORKER TIMEOUT`. This might be due to anything, If your request is too slow that it took more than the timeout specified to response.
- There can be different worker types in gunicorn. Gunicorn is very simple, it just does not let worker live without it processing a request.
  - The workers can be put into two distinct buckets that explain how they process python requests. Sync and Async workers. Sync workers can handle one request at a time while async ones can handle multiple requests at the same time.

## Timeout in gunicorn

- As gunicorn is a process manager(not request handler), the timeout you give during the gunicorn's startup is timeout of a worker process.
- This timeout is equal to the timeout of request when you are using sync workers but can be totally different if you are using async workers.
  - **Value in `--timeout` does not mean the max time a client can wait for response.**
- Following simple flask program demonstrates this behaviour of gunicorn:
  
```
import time
from flask import Flask

app=Flask("__name__")

@app.get("/slow")
def test():
    time.sleep(10)
    return "slept for 10s"

@app.get("/fast")
def test1():
    time.sleep(1)
    return "return super fast, in 1 second"
```

- When you run the program using this gunicorn command: `gunicorn test:app --threads 2 --timeout 5` which spawns multipel execution context per worker process. Then you send a request to the `/slow` endpoint, you get your response after 10 secs. The other thread was alive thus the worker process was able to prove that its alive during the whole time.
- Contrary to above, if you set the thread count to 1 or use sync worker, you will get the worker timeout log and 5xx would be returned to the client.
- The responsibility of timeout is soly given to the application side where the developer must set proper timeout for each request.
  - This can be done in various ways I am giving only specific outlines to a few ways because this topic in itself is worthy of a whole article.
    - Setting every socket connection's alive time globally in each process. This can be very easy fix but might not work in scenarios like http/2 because they use single connection for multiple http requests.
    - Making a timeout decorator and using them in each of the route handler. This is very common in flask like application where you define the route handler using decorators.
    - Use Middleware. This can be applicable in `express-like` frameworks or django where middlewares can ensure that the request can timeout after certain seconds.
    - Use of reverse proxy like nginx. As they create a new http connection between themself and the server application, nginx can track time taken by each request and close connection to server after some time. Nginx will normally return 504 to the clients if it can't reach the application server within the time defined.