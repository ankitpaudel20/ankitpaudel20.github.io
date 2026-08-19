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

Hi, I'm **Ankit Paudel** 👋

I'm a Senior SRE / Platform Engineer at [Docsumo](https://www.docsumo.com), a document-AI company, where I build and run infrastructure that processes millions of documents a month. I work across the stack — Kubernetes platforms, Python/Go backend services, and the developer tooling in between — from Kathmandu, Nepal.

{{< button href="/cv/ankitpaudel-cv.pdf" target="_blank" >}}
Download CV
{{< /button >}}

## What I do now

At Docsumo I own large parts of the platform:

- Architected and run a **distributed document-processing pipeline** that reliably handles millions of documents a month.
- Led the design of a **fault-tolerant workflow system on Temporal**, now serving 90% of clients.
- Built automation for **on-demand ephemeral development environments** inside a single Kubernetes cluster — [I wrote about how](/posts/ephemeral_env/).
- Migrated live production traffic from **Ingress to Gateway API with zero downtime**.
- Run **HA observability with Prometheus + Thanos** ([post](/posts/prom-thanos/)) and **HA persistent storage with Longhorn** ([post](/posts/rwx-in-k8s-longhorn/)).
- Operate a self-hosted **Redis Sentinel cluster** with next-to-zero downtime through node failures and upgrades.
- Built a **feature-flag server on the OpenFeature spec** enabling blue-green deployments.
- Provisioned **secure sandboxes for executing arbitrary Python code**, and made credential rotation routine across cloud-native workloads with secret/parameter management services.
- Define **SLIs and keep SLAs honest** across critical production systems; on-call incident responder.

## AI in my workflow

I treat AI tooling as part of the engineering toolchain, not a novelty:

- **Agentic coding tools are my daily driver** — CLI agents (Claude Code and similar) for writing code, debugging, and infrastructure work, wired into my terminal workflow with guardrails around anything production-facing.
- **LLM-assisted operations** — incident debugging, log analysis, and generating one-off automation during ops work.
- Before the LLM era I completed a **Machine Learning micro-degree** (FuseMachines) and NAAMI's **Winter School in AI**, so I'm comfortable one level below the API too.

## Experience

**Docsumo** — Remote / Singapore
- *Senior SRE / DevOps / Platform Engineer* · Jan 2025 – present
- *DevOps / SRE Engineer* · Jan 2023 – Jan 2025 — multi-tenant Kubernetes ownership, GitLab→GitHub migration cutting operational costs by half, 30%+ database-latency reduction through data-model rework, Keda/HPA autoscaling, CI/CD for cross-platform builds and heterogeneous deploy targets.

**Yasok Systems** — Kathmandu, Nepal
- *Software Engineer* · Feb 2021 – Dec 2022 — site-wide search with Python + Meilisearch, Node.js ERP backend, 50%+ performance gains via Redis caching layers, async notification system on AWS SQS + Lambda, search clusters across Elasticsearch, Meilisearch, and Azure.

**FuseMachines** — Kathmandu, Nepal
- *AI Fellow* · Sep 2021 – Dec 2021 — Machine Learning and Deep Learning micro-degree; trained and evaluated ML models on real datasets.

## Projects

- [**3D renderer from scratch**](https://github.com/ankitpaudel20/simple3drenderer) (C++, OpenGL) — renders arbitrary `.obj` models; includes a software rasterizer implementing the primitive triangle-rendering algorithms low-level graphics drivers use.
- [**Sorting & pathfinding visualizer**](https://github.com/ankitpaudel20/Algorithms-Visualizer) (C++, SDL2) — teaching aid visualizing sorting, pathfinding, and spanning-tree algorithms.
- [**Realtime text similarity**](https://github.com/ankitpaudel20/realtime-text-similarity-frontend) (Python, Flask, scikit-learn) — sentence-transformer-based similarity detection built to help professors avoid repeating exam questions.

## Skills

**Cloud / DevOps / Platform:** Kubernetes, Gateway API, Helm, Docker, Nix, Terraform, Prometheus, Thanos, GitHub Actions, GitLab CI, GCP, AWS, Azure DevOps, secret management, Redis, Linux administration, Bash

**Backend:** Python (Flask, FastAPI), Go, API design, microservices, distributed systems, Temporal, Celery, C/C++, SQL, PostgreSQL, MongoDB, JavaScript

**Foundations:** algorithms & data structures, operating systems, networking, graphics programming & shaders, object-oriented design

## Education & certifications

**BE Computer Engineering** — Tribhuvan University, IOE Pulchowk Campus (2018 – 2023).
Instructor for a one-month software fellowship, designing and running the Docker and cloud-tooling classes.

Certifications: AWS Cloud Foundations · FuseMachines ML micro-degree · NAAMI Winter School in AI · IBM Quantum "Qbit by Qbit" intro to quantum computing

## Get in touch

Fastest way is email: [ankitpaudel20000@gmail.com](mailto:ankitpaudel20000@gmail.com). I'm also on [GitHub](https://github.com/ankitpaudel20), [LinkedIn](https://www.linkedin.com/in/ankitpaudel20), and [Twitter/X](https://twitter.com/ankitpaudel20000).
