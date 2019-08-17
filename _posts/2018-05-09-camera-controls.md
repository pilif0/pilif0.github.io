---
layout: post
title: Camera Controls
categories: [Open Sea]
tags: [C++, input, camera, controls]
date: 2018-05-09 15:39
---
To control cameras, I chose to use the new ECS.
This feature is split into two parts: a system that makes cameras follow their designated entities, and system-like controls that move entities based on input.
The code is split among the `open_sea::ecs` and `open_sea::controls` namespaces.
The relevant pull request is available [here](https://github.com/pilif0/open-sea/pull/15).

# Camera Guiding
The first part of this feature takes care of transforming cameras to the transformation of their designated guide entity.
This relationship is represented by the entity's `CameraComponent`.
This component ties together entities and the cameras they guide.
There shouldn't be multiple records with the same camera, as you couldn't reliably know which entity the camera is going to follow.
But there can be multiple records with the same entity, which just means that all those cameras are to have the same transformation.

The following itself is done by the `CameraFollow` system, which just sets the camera's transformation matrix to the entity's (global) transformation matrix.
The system has two `transform` methods, one without arguments that applies all the entity-camera relationships in the `CameraComponent`, and one which accepts a set of cameras and applies only the relationships relevant to those cameras (if more than one entity is assigned to one camera, the system uses the first one it finds in the data array of the component).

# Controls
The second part of this feature takes care of transforming entities based on input.
This applies to any entity, not just camera guides, but the main use is to manipulate the camera.
Every controls object can have exactly one assigned entity (subject) that is transformed by it.
As usually only one entity is controlled by input, the controls are optimised for it.
But it is possible to instantiate multiple controls objects and assign each of them a different subject, although this could lead to weird results.

Each concrete controls class represents a control scheme with some assumptions.
There are three currently implemented: free controls (free 3D position and orientation control), FPS controls (free position control in the XZ plane, free yaw control, limited pitch control, and no roll and Y position control) and top down controls (free position control in XY plane, free roll control, and no pitch, yaw and Z position control).

Each controls class also has an associated configuration struct that contains its key bindings and factors (e.g. speeds, turn rates, ...).
This struct has to be passed to the constructor to initialise all the required configuration.
Any changes made to the values of the struct instance in a controls object will immediately take effect on the next `transform()` call.

To support both mouse and keyboard input for the controls, I have added a new type of input &mdash; unified input.
More about that below.

# Camera Changes
As mentioned above, cameras have been changed to no longer store transformation components, but directly accept a transformation matrix.
This was done to allow cameras to follow non-root entities, whose global transformation is only in their matrices.
You could still interact with the camera using the transformation components by wrapping it in another object that would just keep the three transformation components and compute and pass the transformation matrix to the camera with every change.

# Unified Input
The new input type &mdash; unified input &mdash; is used whenever you need binary input (i.e. pressed / released) and you don't care about the device.
Every key of the unified input is represented by a bit field with 4 bits for the device identifier (currently only keyboard and mouse) and 28 bits for the device-specific input code (scancode on the keyboard, button number on the mouse).
Whenever any compatible input is pressed, its unified input identifier gets added to a hash set, and whenever it is released that entry is removed.
That means that internally there is always a set of all currently held keyboard keys and mouse buttons.

The state of any unified input can be querried with the `open_sea::input::is_held(...)` function.
There is also a signal set up for unified input in the same way as for other inputs.
Therefore a key press now causes two signals to fire, the keyboard signal and the unified input signal.
There are also two convenience methods in the `open_sea::input::unified_input` struct, `keyboard(...)` and `mouse(...)`, that accept GLFW constants and return their unified input counterparts.

The goal of this system is to make it easy to use keyboard and mouse input interchangeably, which has proved very useful when designing the controls classes.

# Other Input Changes
To further support the controls, I have added cursor delta computations to the input module.
This keeps the change in cursor position since last update.
Cursor delta was needed in multiple controls' implementations, so I chose to implement it once in the input where it gets updated at the start of every frame.

# Conclusion
This feature allows me to now easily inspect the 3D scene, which will make implementing textures and lighting easier as I will have an easy way to test it.
It has also nicely shown how useful the ECS system is.
Using it has resulted in a loosely coupled system of highly reusable concepts.
That also helped with debugging, because the data paths are clear and simple.

While working on this feature, I have improved debug information for a couple modules and noticed that it has been falling behind in a lot of places.
I want to put some work into overhauling the whole debug GUI.
This will mean changing the layout of the whole GUI to put more tools there, as well as going through all the modules of the project and seeing where I can improve availability of debug information.
It will also pave the way for a feature I have been planning for a while, which is an in-system profiler.
That will allow me to see the performance of various parts of the system and hopefully lead to some optimisations.
