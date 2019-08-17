---
layout: post
title: Delta Time
categories: [Open Sea]
tags: [C++, delta time, FPS]
date: 2018-03-09 12:32
---
Many parts of an engine require knowledge of how long the last iteration took.
This then allows scaling of all changes in proportion to time elapsed and makes the experience more consistent with variable update times.
To provide this, I have added the Delta Time module, contained in `open_sea::time`.
The relevant pull request is available [here](https://github.com/pilif0/open-sea/pull/6).

## Initialization
This module has very simple initialization, as it only keeps track of time and every once in a while (~ every second) calculates the average FPS.
To initialize the module, call `start_delta()`, which resets all the counters and clears the history.
This has to be done after GLFW initialization, because the system uses `glfwGetTime()` to get the system time.
It should also be done directly before the start of the main loop to avoid any delay between the initialization and the actual start of the loop.
For now, the delta time starts at 0 for the first iteration.
If I discover that this breaks some functionality in the future (e.g. because of division by zero), I will change the starting value to assure that delta time is always positive.

## Updating
At the end of each iteration, call `update()` to update the delta time.
This computes the delta time, updates all the counters and history, and computes the average FPS if required.

## Time and FPS
To get the actual delta time value, call `get_delta()` which returns the time elapsed since the last update or start of tracking.
The units are seconds, therefore usually it is significantly less than one.

To get the immediate FPS, call `get_FPS_immediate()` which returns the frames per second at the instant it is called.
That is the multiplicative inverse of the delta time.
This value changes every update, therefore to get a more stable average FPS, call `get_FPS_average()`.
The average FPS is counted over about one second (the first update past one second), therefore it should be more representative of the preformance, because it smooths out momentary hiccups.

## Debug
You can call `debug_widget()` to draw a set of ImGui debug features describing the state of the module.
This draws a plot of the delta time history over the last 1000 updates, the current delta time in milliseconds, and the immediate and average FPS.
Note that this is not drawn as its own window (as with Window and Input debug), but has to be drawn into some other window.
See the Sample Game example for a demonstration.
