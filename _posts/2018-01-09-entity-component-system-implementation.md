---
layout: post
title: Entity Component System Implementation
categories: [Open Desert]
tags: [entity system, components, implementation]
date: 2018-01-09 15:40
---
During the implementation of the Entity Component System, some things had to be changed from the design described in the previous post.
In this post I will summarise these changes and the reasons for them.
A lot of the reasons stem from my desire to keep the code simple and modular.
As this article goes out, the changes will be merged from their feature branch into the `master` branch of that [project repository](https://github.com/pilif0/open-desert).

# Game Object
When implementing the game object, I have noticed that the handle is used much less than the design suggested.
In the current state, it is only used in one log message.
One problem I have noticed with the handle as it was designed, was when implementing the game object serialization.
That originally saved the handle as well, but that would have been impractical.
It would mean I would have to maintain a list of used handles to make sure a newly constructed game object didn’t get assigned a handle of a game object that was deserialized.
This would make the game object creation and deserialization much more complex than it needs to be and therefore I decided to scrap that idea, at least for now.
I might, at a later date, reintroduce it if I come up with an elegant solution, but for now the handle is not preserved through serialization and is not widely used.

# Component
Component implementation has changed a lot from the design, mostly towards being less restrictive.
Components are now classes that implement a fairly simple `Component` interface, and a lot of what was designed to be handled by the system is now left to the specific implementation of the component (access control, default values, field overriding, …).

Fields are no longer declared in the components file, as this would introduce a lot of extra work in processing, validating and using components and templates.
Moreover the information on the fields is already contained in the associated class itself.

Templates now state what fields of the component they wish to override, that data gets passed to the component instance after its construction and the component itself decides what to do with it.
Therefore the whole system allows more freedom to the component.

Components also utilise events even more that the design intended.
During the implementation it became obvious just how much that helped keep all components separate.
The best example would be, that none of the `position`, `rotation`, or `scale` components need to know about the existence of the `world_matrix` component, they just send out events through the game object whenever their values change.

# Worker
The closest to a worker, as the design describes them, would be the rendering of game objects.
That is currently handled by a `SpriteRenderer` class with a function that, given a projection matrix and a game object, renders that game object.
The game objects are gathered from the world tree by a condition on having the `sprite` component and then passed to this function along with the camera’s projection matrix.

The `sprite` component contains all the data that the rendering requires except the projection and world matrices (provided by the camera and a separate component respectively).

# Template
The main difference between the design and implementation of templates is that not all fields of a component need to be overridden in a template.
If a field is not overridden, it will just keep the default value specified in the component implementation.
This makes templates shorter, easier to write, and also increases their backward compatibility.
When a new field is added, it is not necessary to change all the templates that use the component unless the default value is not suitable.

# Files
As mentioned above, fields were dropped from the components files and the handle was dropped from the gameobjects files.
For illustration, below are examples of files from the implementation.

```
#numbers.template
name: numbers
description: Sprite test (numbers from 0 to 7)
parent: none
components:
    - position:
        position: 800, 600
    - rotation
    - scale
    - world_matrix
    - sprite:
        atlas: textures/atlas.png
        dimensions: 64, 64
    - keyboard_sensitive
    - test_sprite_control:
        increment: left
        decrement: right
```

```
#main.components
- name: position
  description: Position
  class: net.pilif0.open_desert.components.PositionComponent
  required: none

- name: rotation
  description: Rotation
  class: net.pilif0.open_desert.components.RotationComponent
  required: none

- name: scale
  description: Scale
  class: net.pilif0.open_desert.components.ScaleComponent
  required: none
...
```

```
#root.gameobjects
- template: texture_test
  components:
    - position: {position: '238.738922, 242.719513'}
...
```
