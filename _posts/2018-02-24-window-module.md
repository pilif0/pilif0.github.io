---
layout: post
title: Window Module
categories: [Open Sea]
tags: [C++, GLFW]
date: 2018-02-24 13:50
---
Recently I finished a new module for Open Sea, the Window module.
This module is an abstraction over the [GLFW](https://www.glfw.org/) library and is contained in the `open_sea::window` namespace (omitted from identifiers below).
The relevant pull request is available [here](https://github.com/pilif0/open-sea/pull/2).

The module uses a few assumptions to simplify interaction with GLFW.
The main assumption is that there will only be one window.
Games rarely need more than one window, so this is a reasonable assumption.
It means that you now don't need to supply the window pointer to every method, because they now assume you are working with the global window stored in `window`.
There is also a set of properties of that window stored in `current`, which allow quick access to frequently used properties like size or state.
Note that using the window pointer in `window` you can still call normal GLFW functions, but that should not be necessary and all you need should be provided by this module.

# Creating a Window
Before a window can be created, GLFW needs to be initialized.
This is done by `init()`, which also sets an error callback that logs any GLFW errors.

Once GLFW is initialized, you are free to use one of `make_windowed(width, height)`, `make_borderless(monitor)`, `make_fullscreen(width, height, monitor)` to create a new window that is windowed, borderless or fullscreen respectively. The window becomes the global window and `window` is set to point to it.
Using any of these functions when the window already exists doesn't create a new one but only transforms the old one into the desired state.

The windows are shown at the end of creation.
If you wish to hide or show the window, you can use `hide()` and `show()` respectively.
The window can also be centred on the primary monitor using `center()`.
The window is not resizable by the used, that has to be done by either `set_size(width, height)` or one of the `make_***(...)` functions.

# Properties
As mentioned above, there is a struct that contains the frequently used properties of the window.
It contains its size, framebuffer size, title, monitor pointer, state (windowed, borderless, fullscreen) and vertical synchronization status.
The struct is defined in `window_properties` and the default values of its fields are defined in the `defaults` namespace under the same names as in the struct.

A copy of these properties can be retrieved using `current_properties()`.
To change properties, change the window using one of the other methods (`set_title()`, `enable_VSync()`, ...).
This way the properties always reflect the window and can't get any invalid values.

# Updating
At the end of each iteration of the main loop, the buffers are swapped and events are polled.
This can be done by calling `update()` which then calls the appropriate GLFW functions.

# Termination
The close flag of the window can be set using `close()`, which then results in `should_close()` returning true, which could be used to terminate the main loop.
There is also `clean_up()` that will destroy the window and reset the pointer to it.
This could be used to recreate the window (by using `make_***(...)` after), but the OpenGL context is destroyed with the window, so you would have to make sure it is not required in the meantime.

The function `terminate()` is a wrapper around `glfwTerminate()` and will result in destruction of all windows and termination of GLFW.
This should be used when closing the game after the main loop.

# Callbacks
There are three methods to set the most common window event callbacks &mdash; size, focus and close.
These take the appropriate GLFW callback function references.
Other callbacks can also be set by using GLFW directly and giving it `window` as the window pointer.

# Conclusion
This module should take care of all that is needed for windows.
The `window` pointer is provided to take care of any edge cases, as with it you can directly use GLFW to do anything it can do.
GLFW windows also function as the source of input for the game, but that will be taken care of in its own module, which will first require finding or creating an appropriate event system to use for it.
