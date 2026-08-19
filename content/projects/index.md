---
title: "Projects"
description: "Things Ankit Paudel has built — graphics, algorithms, games, and tooling."
showDate: false
showDateUpdated: false
showReadingTime: false
showAuthor: false
showComments: false
showPagination: false
showTableOfContents: false
---

Things I've built outside of work — mostly to understand something from first principles. The cards below are live from GitHub.

## 3D renderer from scratch

C++ and OpenGL. Renders arbitrary `.obj` models, and includes a pure-software rasterizer that implements the primitive triangle-drawing algorithms low-level graphics drivers use — written to understand what the GPU is actually doing for you.

{{< github repo="ankitpaudel20/simple3drenderer" >}}

## Algorithms visualizer

C++ and SDL2. A teaching aid that animates sorting, pathfinding, and spanning-tree algorithms step by step.

{{< github repo="ankitpaudel20/Algorithms-Visualizer" >}}

Later I rewrote parts of it in Go, as my first excuse to learn the language properly:

{{< github repo="ankitpaudel20/algoviz-go" >}}

## Realtime text similarity

Python, Flask, and scikit-learn — my college final project. Uses sentence transformers (plus simpler methods like smooth inverse frequency) to score how similar two pieces of text are, built to help professors avoid repeating exam questions.

{{< github repo="ankitpaudel20/realtime-text-similarity-frontend" >}}

## Pocket Tanks clone

C++, SFML, and Box2D. A physics-based artillery game clone — projectile arcs, destructible terrain, the works.

{{< github repo="ankitpaudel20/Pocket-Tanks-Clone-in-SFML" >}}

## This machine, declared

My entire home environment — shell, editor, tools — as reproducible Nix configuration. It's also how this site's dev shell works.

{{< github repo="ankitpaudel20/dotfiles" >}}

---

More experiments live on [my GitHub profile](https://github.com/ankitpaudel20).
