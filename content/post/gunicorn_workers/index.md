---
title: "Guide to worker types in gunicorn"
description: "Why are there different worker types in gunicorn and when to use which type of worker." 
date: 2024-12-22T11:41:14Z
toc: true # Controls if a table of contents should be generated for first-level links automatically.
image: 
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - Python
---

# Python is single threaded

- Python by design is single threaded. It can only run one thread at once due to GIL.
- This makes making web server which serves a lot requests at once a bit complicated.
- Thread based concurrency is good enough for most of the applications but its control block is heavier than traditional async coroutines.
- This led to necessity of eventloop based concurrency which can share almost eveything of main python thread except the request data.
- This would make coroutines very light with fast context switches.
  
# Async in python before asyncio

- To handle thousands of request at once, there were a few implementations that made python work somewhat like async runtime.
- Gevent and eventlets being the major one.
- There was also a framework called tornado which had its own implementation of coroutine based server which is not that popular now.
- First eventlet was released to tackle this problem with its pure python implementation but later gevent came to picture with its blazing fast speed as it was implemented in cython itself.
- They work by monkeypatching specific functions in python that does I/O operations. They replace those functions with their own version so that they emit some information to the eventloop manager before going to I/O wait state. This allowed for another function to run till it entered same kind of I/O wait state.
- They were built upon the concept that a python process should never be sitting idle for I/O wait.

# Role of Gunicorn <-> workers

- Gunicorn is just a process manager, that makes sure that "worker" process never stop processing the requests.
- It has two distinct section of architecture; one being "arbiter" and the other being "worker".
- Arbiter does the job of manging worker processes while the worker process does the job of communicating with arbiter along with starting/monitoring the application execution context that processes request as per worker type defined.
- The worker type defines how the server application gets run. The default sync worker process one request at a single time per process.
- The workers that comes packaged in gunicorn are process-worker, threads, gevent, eventlet and tornado. But you will be able to find other third party workers like uvicorn workers that supports fastapi like async frameworks too.

## Sync vs Async workers in gunicorn

- Sync workers are those workers which can handle requests sychronously i.e. one request at a time.
```
# both the gunicorn instances below can process 2 requests at once. The first one uses threads to do so while the second one uses python processes.
gunicorn test:app --threads 2 --timeout 5
gunicorn test:app --workers 2 --timeout 5
```
- Async workers, as the name suggests, can handle multiple requests at once. So there will theoritically be good resource utilization for i/o bound task.
- Gunicorn provides us with gevent, eventlet and tornado workers out of the box.
- Gevent is the most widely used due to its compatibility and speed as compared to tornado and eventlet.

# When to use which workers

- **When to use sync default workers**
- If the endpoints that your server serves is resource heavy and can fully utilize the cpu of the application in each request then sync workers is the only option for you.
- If your system is mutltithreaded, then you will benefit from running multiple workers that fully tanks your cpu. If your application is very resource intensive, number_of_workers=number_of_cpu_threads can perform well.
- Assuming you have around half of the application which is cpu intensive and the other half is somewhat io intensive you should be okay with the golden rule floating around, num_workers=(2*num_cpu_cores)+1
- As increasing the number of workers directly increases the python processes running your application code, it has the highest memory overhead.
  
- **When to use thread workers**
- If your endpoints are I/O intensive that can somewhat benefit from python's inbuilt gil, then go for threaded workers.
- This can be used as default for most of the new flask or WSGI applications.
- The number of threads will depend on your workload.
- This is a bit lighter than worker processes, as not the whole python application is loaded in memory as soon as a new thread gets spawned.
- Threaded worker is the only option for a few cases where gevent support is not there for application code.
- As this shares same python process, use of any global state that any endpoint modifies or uses is prohibited. If done, undefined behaviour might occur.

- **When to use Gevent workers**
- If your endpoints are almost exclusively I/O bound this should be the best type of worker for you.
- This will ensure highest throughput for your application with linear resource scaling for I/O bound tasks.
- This also prohibits any global state being shared across requests.
- The issue with this worker is not all the applications and library support it.

# True async workers vs gevent

- The asyncio library of python is superior in portability and operability. But when it comes to sheer number of coroutines that an application can handle if the coroutine is totally network bound, gevents can also sometimes be better.
- Gevents don't support context switching in all the internal I/O apis like file operations, so file operation in gevent worker will stop the whole program from responding.
- Fastapi and starlette are built around asyncio framework of python, thus are better at handling real world workloads and would certainly be better performing in medium to high loads as compared to gevent.
- Pallet Projects, the parent project of flask has another flask like framework that supports true async worker called Quart. But the community for fastapi and starlette is very large as compared to those.
- Hence my personal recommendation would be to use fastapi or such for web application in python.