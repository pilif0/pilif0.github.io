---
layout: post
title: Input Module
categories: [Open Sea]
tags: [C++, Boost.Signals2, input]
date: 2018-02-28 11:59
---
Another new module for Open Sea is the Input module.
It builds on top of [GLFW](http://www.glfw.org/)'s input using the [Boost.Signals2](http://www.boost.org/doc/libs/1_66_0/doc/html/signals2.html) library.
It allows easy access to signals for input of the global window defined in the Window module.
This module is contained in the `open_sea::inut` namespace (omitted from identifiers below).
The relevant pull request can be found [here](https://github.com/pilif0/open-sea/pull/3).

## Initializing
Before the input system can be used, it needs to be initialized using the `init()` function.
This instantiates the actual signals and tries to attach them to the current global window using GLFW callbacks.
If the window has not been created yet (the pointer is `null`), it doesn't attach anything and the signals will not fire.
Once a window is created (initially or after having been destroyed), you need to call `reattach()` to attach the signals to the new window.

## Signals
There are signals for all the main input methods except for joysticks (to be added at a later date).
That is, keyboard, cursor entrance, mouse buttons, scrolling and characters.
The specific description of these different input methods can be found in the [GLFW input guide](http://www.glfw.org/docs/latest/input_guide.html).

Functions can be connected to the signals using the `connect_***(slot)` functions, where `***` is the relevant input method.
This returns the connection instance which can then be used to disconnect from the signal.
The function types for each signal can be found by accessing the `slot_type` field of the relevant signal type.

## Querying
Apart from listening to signals, you can also directly query some input.
This applies to cursor position, keys and mouse buttons.
Cursor position is returned as a 2D vector of doubles of screen coordinates with origin in the top left corner of the window.
Keys and mouse buttons need to be supplied as GLFW key names (either the `GLFW_KEY_***` constants or directly their values, which can be found [here](http://www.glfw.org/docs/latest/group__keys.html)).
The limitation of querying keys instead of listening for signals is that querying only returns whether the key is pressed or not.
It doesn't distinguish between pressing down, holding and releasing.

You can also use the `key_name(key, scancode)` function to retrieve the localised name of a printable key.
The key argument is the GLFW key name and scancode is the system-specific key code.
The scancode is only used if the key has value of `unknown_key` (which is `GLFW_KEY_UNKNOWN`).
If a printable name is not found, it returns the string "undefined".
Note that this functionality is not supported in Wayland and returns "undefined" for all keys.

## Conclusion
This module allows simple and quick interaction with most frequent forms of input.
You should keep in mid that a lot of functions can listen for signals and some signals fire often (e.g. a key being held down).
This means that the listening functions should not contain any complex calculations, rather just setting flags that are used in more infrequent calculations.
