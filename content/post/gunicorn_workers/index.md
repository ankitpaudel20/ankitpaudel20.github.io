---
title: "Guide to worker types in gunicorn"
description: "Which type of worker to use in which kind of scenario" 
date: 2024-12-22T11:41:14Z
toc: true # Controls if a table of contents should be generated for first-level links automatically.
image: 
math: 
license: 
hidden: false
comments: true
draft: true
categories:
  - Python
---
Outline:

## Python is single threaded
* Python by design does not used to allow event loop based async before asyncio
* thread based concurrency not good enough for large workloads.
* Eventloop based concurrency required.
* memory constraints
* Async before asyncio in python.
  * Greenlets and eventlets
  * How do they work.
# Role of Gunicorn <-> workers
* Separation of concern and worker management
* workers needed for concurrency and multiple connection handling capacity
# Sync vs Async workers
* Sync are sync and async are async
* Workers are independent. they can be made custom while the arbiter is main part of guncorn
# Sync vs Async workers
* Process vs threaded workers.
* Process worker no brainer, thread worker needs global state free application.
* Basic gunicorn commands for them.
* Async workers, greenlets.
NOTE: CHECK IF UVICORN WORKER WORKERS WORK IN FLASK APPLICATIONS AND THEIR PERFORMANCE IS IMPROVED
* How do these async workers work.
* Limiting async workers, max connections and backlogs.

## When to use which workers:
* cpu intensive, sync workers,
* a bit io intensive and does not support greenlets, threads.
* Heavy io intensive: greenlets.
  * Greenlets have catch not everything can work in them.

## True async workers vs greenlets/eventlets
* greenlets are limited support and context switch only on network call
* better performance in async based frameworks like fastapi, starlette or quart.
* Recommended fastapi.
