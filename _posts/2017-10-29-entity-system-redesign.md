---
layout: post
title: Entity System Redesign
categories: [Open Desert]
tags: [entity system, components, redesign]
date: 2017-10-29 12:41
---
As I have described in the [Components](https://smola.me/blog/2017/9/24/components) post, I have decided to completly redesign the current entity system.
The new solution will follow the [Entity Component System](https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system) pattern.
As this is an extensive change to the code which will also be the base for a lot of future code, it needs to be planned out first.
In planning this, I will use various articles, books, and talks available online.
Any significant sources are linked at the end of the post.

# Current State
The current system uses inheritance to specialise entities.
These are then, for example, directly connected to the rendering system, or have their position changed from outside.
This was an okay state for testing whether things render and move, but it is not sustainable in the long term.
It is not flexible enough to allow easy extension, and therefore replacing it was inevitable.
I have decided that it is better to replace it now, rather than later when more other systems depend on the current implementation.

# New Solution
The new system will be based on the Entity Component System pattern.
This will decouple the different aspects of the entities from eachother and allow them to be developed independently.
In this case, an entity will be called Game Object.

In a short summary, the whole system consists of Game Objects, that function as containers for Components, which contain data and behaviour that actually defines that specific Game Object.

## Game Object Composition
The Game Object will have the following responsibilities:

- Provide general Game Object data (handle, ...).
- Tie together Components.
- Support organisation into a quad tree representation of the world.

The aim is to keep the Game Object itself very simple, leaving as much of its content as possible to the Components.
The handle will be used for look-up and so that multiple views of the same world (for example in multiplayer) can agree on it being the same thing.
Therefore it will be unique in the world.
The Components will be kept in a List and access to them will mostly be done using Streams.
There will be convenience methods for adding, removing, retreiving, and checking the presence of Components.

Every Game Object will have the Position component to specify where in the world it is located.
This will work to make the world graph representation more effective.

There will only ever be allowed one instance of a Component type for each Game Object.
The deciding factor will be the name of the component.
More on that below.

There will also be a method to decide whether a Game Object matches a certain condition (combination of Components, position, ...).
This will allow more effective filtering of Game Objects, as for example checking for a set of Components can be done at once, not necessarily one by one.

## Component Composition
Components can have two types of elements:

- Fields — values that describe the Game Object state
- Events — things that have happened to the Game Object

Every Component will also be able to handle a variety of requests, which will facilitate other Components asking this one to do something (for example a health Component to respond to damage).
This will further decouple the Components, because if the same was to be done with direct access to fields, then the affecting Component would have to contain the logic that should belong to the affected Component (for example equation for taking resistances into account when calculating damage).

Access to all fields of the Component will go through getters and setters, because otherwise one could accidentaly avoid firing an event (for example position change event) or break a certain assumption about the value (for example it always being positive).

Every Component (type) will also have a name that will serve to reference to it, for example in the Templates (more on those later).
This name will be a part of the information stated in the Component declaration file.

Every Component will have to be declared in a file in order to be used.
That file will be used when initiating the game.
Each Component will have its name, description, associated class, any required Components, and fields stated in that file.
The fields will have information about their type and default value, and a short description of that field.
The fields stated in this declaration will be the only ones that can be overriden by a Template or Game Object file.

## World Representation
The world will be represented by a quad tree which will be ordered by position.
That is in fact the reason, why every Game Object has to have the Position Component.

Every segment will split once it contains a predetermined number of Game Objects (found by testing, exposed in settings).
The borders of the segments will always belong to the segment closer to the point `(+infinity, +infinity)`.
For example, the origin will belong to the top-right segment that has it as one of its corners.
It remains to be determined by testing whether the segments should merge back once the total number of Game Objects in all four is low enough.

Every node will have methods to query it for Game Objects by a variety of conditions.

## Worker Composition
One could also use Workers in addition to Components for some tasks.
Workers would be useful in cases where it is not optimal to update every Game Object on its own, but maybe in groups.
An example of this would be collision calculations being done in pairs.
In order to use these, you would leave the `update` method of the Component empty, and instead create a Worker for it.
Then you would attach an instance of the Worker to the root of the world tree.
The Worker would then query the world tree for all Game Objects that have the desired combination of Components, and operate on that set.

If a Worker could be run in parallel, then it would split after every update in which the number of updated Game Objects exceeded a certain limit.
The split would mean creating three new instances of that Worker, detaching self from the tree, and attaching each of the new instances and self to the four subnodes of the original node.
These Workers would then be run on separate Threads next update, thus hopefully improving performance.
This limit would have to be at least the limit for splitting the nodes, as otherwise one could try to split a Worker attached to a node with no subnodes.

## Templates
One more type of data to talk about is Templates.
Templates, as the name suggests, act as blueprints for Game Objects.
The template dictates some basic default information about the Game Object (name, description, ...)
and the set of Components that the Game Object has.
Every Game Object can then be specified by a Template and its difference to it.

For example, a particular chicken in a field will have a Template "chicken".
That Template will say that the Game Object has a particular model and texture, sound, AI, and so on in the Components responsible for those.
And the actual Game Object will then override the position to the actual position of the chicken, as well as maybe its health or AI.

When the world is loaded, every Game Object gets actual instances of its Components with the correct values.
The Templates will however be used when constructing new Game Objects, and when saving Game Objects.

One important feature of Templates will be inheritance.
Any template can declare that it extends another template and it will receive all its information by default.
Then if it declares that it has a Component that is already in the template it is extending, it can override some (or all) of the properties of that Component.
It can also add new Components, which then means it will have to provide values for all the properties of those Components.

For example that "chicken" Template will be extending an "animal" Template somewhere along the way, which will give it the health Component.
Every animal that then extends that Template can set its own maximum and starting health, as well as add other Components, like an aura of fire.

Templates will be declared in files, with each Template specifying its name, description, parent, and Components with overriden values.

## External File Formats
For now, the files will be formatted as YAML, as then I will not have to make a custom parser in this stage.
The files will be: Template files, Component declaration file, and Game Object files.
The Game Object files will probably be merged into multiple files, as one file per Game Object would result in a lot of small files.
These files will be based on the nodes of the world tree.

The Component declaration file will contain at least one Component declaration.
That declaration will state the component name, description, associated class, required Components, and fileds.
Each field will state its name, data type, description and optionally a default value.
The name of the file does not play a role, but the extension has to be ".components".
All found files with this extension will be read on startup.

```
# main.components
- name: health
  description: A simple health component
  class: net.pilif0.open_desert.components.HealthComponent
  required: none
  fields:
      - name: maxHealth
        type: int
        description: The maximum allowed health
        default: 1
      - name: health
        type: int
        description: The current health
        default: 1
- ...
```

The Template files will each have a single Template described in them.
The name of the template will be same as the file name and the file extension will be ".template".
The Template description will contain the name, description, parent, and list of Components with overriden values.

```
# chicken.template
name: chicken
description: A chicken
parent: animal
components:
    - health:
        maxHealth: 10
        health: 10
    - model:
        mesh: chicken.obj
        texture: chicken.png
	- ...
```

The Game Object files will each have at least one Game Object described in them.
The name of the file will not matter, but the extension has to be ".gameobjects".
Each Game Object stated in the file will have its handle, template, and list of Components with overriden default values (simillar to Templates but this time with relation to the Template defaults not the Component defaults).

```
# root.gameobjects
- handle: 1234567890
  template: chicken
  components:
      - position:
    	  position: (12, 3, 45)
      - health:
          health: 3
- ...
```

# Sources
[A Data-Driven Game Object System](http://gamedevs.org/uploads/data-driven-game-object-system.pdf) &mdash; Scott Bilas (GDC 2002)  
[Component - Decoupling Patterns](http://gameprogrammingpatterns.com/component.html) &mdash; Game Programming Patterns by Robert Nystrom  
[SpatialOS ECS implementation](https://docs.improbable.io/reference/11.1/index)
