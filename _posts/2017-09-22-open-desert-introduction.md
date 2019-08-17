---
layout: post
title: Open Desert Introduction
categories: [Open Desert]
tags: []
date: 2017-09-22 15:24
---
[Open Desert](https://github.com/pilif0/open-desert) is my OpenGL sandbox.
It is written in Java and uses the [LWJGL 3](https://www.lwjgl.org/) for bingings to the native libraries.
The aim of the project is to create a functioning game engine.
Currently it is strictly 2D, with the plan to add 3D capabilities at some point in the future.

# Goals
The main goal of the whole project is for me to learn concepts connected to game engines.
That includes computer graphics, basic physics simulation, and parallelisation.
I want to tackle common problems found when creating a game engine and come up with my own solutions to them, so as to learn as much as possible.
The aim is in no way to create a competitive commercial game engine, as that would be virtually impossible with no budget and just one person working on it.

Big emphasis is placed on modularity.
I try to make the different parts of the code as modular as possible, because that then allows me to easily upgrade one part of the system without breaking the rest.
It would also theoretically allow me to reuse parts of the systems without depending on the whole thing.

One more thing I am trying to learn from this project is how to write proper documentation for an open-source project.
From the project documentation such as the [Contributing Guideliness](https://github.com/pilif0/open-desert/blob/master/CONTRIBUTING.md) and the [README](https://github.com/pilif0/open-desert/blob/master/README.md), to the JavaDoc for each class, method and field.
I have in the past encountered libraries that had sub-par documentation and that meant extra time spent hunting down how some mothod worked.
Therefore, I try to provide as much documentation as I can, so that all the information is readily available.

# Contributing
If anyone wants to contribute something to the project, all the information required is in the [GitHub repository](https://github.com/pilif0/open-desert).
In short, I accept contributions in the form of Pull Requests connected to some Issues.
In the Issue you can describe what the problem or addition is about, and in the Pull Request you submit the actual code.
This has to be done from a fork of the original project repository.
The whole process is described in the [Contributing Guidelines](https://github.com/pilif0/open-desert/blob/master/CONTRIBUTING.md) file in the project repository itself.
