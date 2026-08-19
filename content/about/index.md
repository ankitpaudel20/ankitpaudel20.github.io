---
title: "About"
description: "Ankit Paudel — Senior SRE / Platform Engineer at Docsumo. Kubernetes, Python, Go, and AI-augmented infrastructure."
showDate: false
showDateUpdated: false
showReadingTime: false
showAuthor: false
showComments: false
showPagination: false
showTableOfContents: true
---

Hi, I'm **Ankit** 👋

I'm a platform / SRE engineer at [Docsumo](https://www.docsumo.com), a document-AI company, working from Kathmandu, Nepal. My day job is keeping a fleet of Kubernetes clusters and Python/Go services happy while they chew through millions of documents a month — and building the tooling so other engineers can ship without thinking about the plumbing underneath.

I like the messy middle of infrastructure: the part where "it works on my machine" meets "it has to work for everyone, at 3 a.m., during a node failure." Most of what I write on this blog comes from that place.

{{< button href="/cv/ankitpaudel-cv.pdf" target="_blank" >}}
Download CV
{{< /button >}}

## Things I've been up to at Docsumo

- Rebuilt our document-processing pipeline into a distributed system that comfortably moves millions of documents a month.
- Helped the team move critical workflows onto **Temporal** — most of our clients run on it now, and it fails far more gracefully than what it replaced.
- Built **on-demand ephemeral dev environments** that spin up a full product deployment when you need one and quietly tear it down when you don't — [I wrote about how](/posts/ephemeral_env/).
- Moved live production traffic from Ingress to the **Gateway API** without anyone noticing. Zero downtime — the good kind of boring.
- Look after observability (**Prometheus + Thanos** — [there's a post](/posts/prom-thanos/)) and storage (**Longhorn** — [that one too](/posts/rwx-in-k8s-longhorn/)).
- Run a self-hosted **Redis Sentinel** cluster that shrugs off node failures and upgrades.
- Built a **feature-flag service** on the OpenFeature spec so blue-green releases stopped being scary.
- Made credential rotation routine across our cloud workloads, and built **sandboxes that can safely run arbitrary Python**.
- Carry the pager, define the SLIs, keep the SLAs honest.

## How I use AI

I'm not training foundation models — I'm the person who wires AI into everyday engineering work and actually gets value out of it:

- **Agentic CLI tools** (Claude Code and friends) are my daily driver for writing code, debugging, and poking at infrastructure — with guardrails in place so nothing touches production without a human saying so.
- **LLMs during incidents** — log analysis, hypothesis generation, and the endless one-off scripts that ops work produces.
- The underlying tech isn't a black box to me either: I did a machine-learning micro-degree at FuseMachines and NAAMI's Winter School in AI, back before any of this was cool.

## Where I've worked

**Docsumo** · Remote / Singapore — joined in 2023 as a DevOps/SRE engineer taming multi-tenant Kubernetes; since 2025 I'm the senior platform/SRE engineer. Along the way: migrated the whole org from GitLab to GitHub (and halved the tooling bill), cut database latency by about a third through data-model rework, and set up autoscaling with Keda/HPA that actually tracks load.

**Yasok Systems** · Kathmandu — backend engineer from 2021 to 2022. Built site-wide search with Python + Meilisearch, made a Node.js ERP backend noticeably faster with Redis caching, and put together an async notification system on AWS SQS + Lambda.

**FuseMachines** · Kathmandu — AI fellow in 2021, where the ML micro-degree happened.

## Beyond work

Things I've built for fun — a 3D renderer, algorithm visualizers, a tank game — live on the [projects page](/projects/).

I studied Computer Engineering at IOE Pulchowk Campus (2018–2023). During that time I also taught Docker and cloud tooling in a one-month software fellowship — turns out explaining containers to a room full of people is the fastest way to properly understand them yourself.

Certifications, for the record: AWS Cloud Foundations · FuseMachines ML micro-degree · NAAMI Winter School in AI · IBM Quantum's "Qbit by Qbit" intro to quantum computing.

## Say hi

The inbox is always open: [ankitpaudel20000@gmail.com](mailto:ankitpaudel20000@gmail.com). I'm also around on [GitHub](https://github.com/ankitpaudel20), [LinkedIn](https://www.linkedin.com/in/ankitpaudel20), and [Twitter/X](https://twitter.com/ankitpaudel20000). And if you're hiring — the CV button up top has the formal version of this page.
