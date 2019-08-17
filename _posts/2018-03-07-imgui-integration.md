---
layout: post
title: ImGui Integration
categories: [Open Sea]
tags: [C++, ImGui, debug, GUI]
date: 2018-03-07 17:31
---
To allow easy debugging of Open Sea at runtime, you need a good debug GUI.
One of the most important requirements is for it to be easy to expand.
You don't want the debug GUI to be a substantial part of development of a new module.
To that end I have decided to use [Dear ImGui](https://github.com/ocornut/imgui).
It is an immediate mode GUI that I first saw in a [GDC talk](https://youtu.be/W_okgL6HJX8?t=37m50s).
Module integrating this library with Open Sea is contained in `open_sea::imgui` (omitted from identifiers below).
The relevant pull request can be found [here](https://github.com/pilif0/open-sea/pull/5).

## Initializing
As with other modules, before you do anything you need to call `init()`.
This maps the relevant GLFW keys to ImGui, loads cursors and connects input slots.
After that you can start interacting with ImGui.
It is recommended that you make the GUI building and rendering every frame conditional on some flag, so that you can enable and disable (show and hide) the GUI.
When doing that, you also want to appropriately connect or disconnect ImGui's input slots so that it doesn't react to input when disabled.
An example of how to do this is in the Sample Game example.
I am working on a way to make this automatic, but haven't found an elegant solution yet.

## Building and Rendering
Every frame when you want to show the GUI, you need to call `new_frame()`.
This prepares ImGui for a new frame by making sure it has all the proper OpenGL objects prepared, updating the window size and sorting out some input flags.
After that you can use functions from `imgui.h` to build the actual GUI.
To find out how to do this, see the [ImGui GitHub repo](https://github.com/ocornut/imgui) and the Sample Game example.

Once you are done building the GUI, you can render it by calling `render()`, which takes care of getting all the data from ImGui and passing it to OpenGL for rendering.
This function saves and then restores any OpenGL state, therefore it will not affect other rendering in any way.

Once you are done with the main loop and no longer wish to use the GUI, call `clean_up()` to make sure all the allocated resources are correctly freed.

## Conclusion
This module is a very simple bridge between Open Sea and ImGui improving the debug capabilities of the engine.
I trust that it will become an integral part of all the various modules and allow for more rapid development in the future.
