---
title: "Projects" # Title of the blog post.
date: 2024-06-22T23:16:58+05:45 # Date of post creation.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
---

Here are a few projects that I have been involved in:

### [Intelligent Document Processing](https://www.easeannotate.com/)
* Takes in images of unstructured documents like invoices and outputs structured information like billing details and total to name a few.
* Involved in making the overall backend of the website and deployment side of things.
* University Major Project
* Uses fastapi as backend and React as Frontend.

### [Realtime text similarity Indentification](https://github.com/ankitpaudel20/realtime_text_similarity_backend)
* Gets list of potentially similar text as the user types in the frontend.
* Involved in rearching and testing different encoders for better accuracy and making the backend to support that.
* Mainly uses word to vec and bag of words to generate word embeddings
* Implemented a few methods like averaging, Smooth inverse frequency to transform word embeddings to sentence embeddings to compare them.
  * Smooth inverse frequency takes in account frequency of word in a sentence to form the overall embedding of the sentence. 
* Also used state of the art of models of that time like bert and Sentence Transformers by Google for increased accuracy 
* Used Annoy by spotify to generate a light weight vector cache for faster KNN retrival.
  * Vector databases were not too popular during the time.
* University Minor Project
* Uses Flask as the backend and React as the Frontend.

### [Priority Analyzer](https://github.com/ankitpaudel20/priority-analyzer) | [Alt](https://github.com/ankitpaudel20/priority_analyzer_go)
* Webapp to get probability of getting admission to different government engineering colleges based on your rank on IOE Entrance exam
* Uses past information about admissions to predict the probability.
* Involved in the development of backend.

### [Hardware Accelerated 3D Renderer](https://github.com/ankitpaudel20/simple3drenderer)
* Renders 3d objects in standard formats like .obj, .fbx and such.
* Solo side project to get better at C++.
* Uses modern opengl, supports diffuse, specular, ambient maps for phong shading.
  * Supports realtime shadows, multiple point lights and normal maps.

### [Software Accelerated 3D Renderer](https://github.com/ankitpaudel20/ComputerGraphicsProject)
* Same as Hardware Accelerated 3d renderer but instead uses software based rendering.
* Project for the Computer Graphics course at University
* Uses primitive graphics algorithms like Bresenham for lines, Barycentric interpolation to fill triangles.
* Implemented Texture mapping from scratch.
* Uses C++ and sdl

### [Algorithm Visualizer](https://github.com/ankitpaudel20/Algorithms-Visualizer)
* Project for Datastructures and Algorithm University course.
* Visualizes how different Sorting, MST and Pathfinding algorightms work.
  * Slowed down to show different operations like swap and comparision.
* Uses C++ and sdl

## [Tank Wars](https://github.com/ankitpaudel20/Pocket-Tanks-Clone-in-SFML)
* First year University Project.
* Strategic tower defense game inspired by a game named "Pocket Tanks".
* Implemented in C++ and SFML.
