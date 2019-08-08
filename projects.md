---
layout: page
title: Projects
permalink: /projects/
---

# Open Sea
This project is my effort to learn C++ while also improving my skills with OpenGL.
The aim is to create a usable game engine supporting at least two, if possible then also three dimensions.
As the purpose is to practice C++, there will be a lot of iterative improvements and often substantial changes of how the system works in order to make it better.
I will try to document the decisions that go into development of this project in the dedicated category in my blog.

### Libraries
The libraries utilised in this project are:
- [OpenGL](https://www.opengl.org/) &mdash; for the rendering
- [glad](https://github.com/Dav1dde/glad) &mdash; to load OpenGL function pointers
- [GLFW](https://www.glfw.org/) &mdash; for window and input abstraction
- [Dear ImGui](https://github.com/ocornut/imgui) &mdash; for debug GUI
- [Boost](http://www.boost.org/) &mdash; for logging, event system and other utilities

Other tools also used are:
- [CMake](https://cmake.org/) &mdash; for building
- [Doxygen](http://www.stack.nl/~dimitri/doxygen/) &mdash; for documentation generation

### Links
- [GitHub repository](https://github.com/pilif0/open-sea)
- [Licence](https://github.com/pilif0/open-sea/blob/master/LICENCE.md)

---

# Basilisk
This project is my experiment with LLVM.
It is an LLVM for a very simple C-based language.
The aim is to learn concepts of compiler architecture and working with LLVM.
The first goal is to create a fully working LLVM frontend that can convert source files to object files, for the simplest possible working language.
After that, the plan is to slowly add further features to the language and build it up into a fully useable language.
After the initial goal is reached, I will try to document the development of the language and decisions taken in a dedicated category in my blog.

### Libraries
The libraries utilised in this project are:
- [LLVM](https://llvm.org/) &mdash; for optimizations and backend
- [Boost](http://www.boost.org/) &mdash; for unit testing

Other tools also used are:
- [CMake](https://cmake.org/) &mdash; for building
- [Doxygen](http://www.stack.nl/~dimitri/doxygen/) &mdash; for documentation generation

### Links
- [GitHub repository](https://github.com/pilif0/basilisk)
- [Licence](https://github.com/pilif0/basilisk/blob/master/LICENCE.md)
