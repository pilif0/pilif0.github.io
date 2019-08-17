---
layout: post
title: Entity Component System
categories: [Open Sea]
tags: [C++, data-oriented, entity component system]
date: 2018-04-20 13:03
---
To allow easier interoperation of different parts of the engine on game objects, I have decided to add an entity component system.
I took inspiration from the ECS I designed and implemented for Open Desert and also the articles on the [bitsquid blog](http://bitsquid.blogspot.co.uk/2014/08/building-data-oriented-entity-system.html).
Given that this time I am working with C++, not Java, I decided to take as data-oriented approach as I could and use the language to its fullest.
All code related to the ECS is contained in the `open_sea::ecs` namespace.
The relevant pull request is available [here](https://github.com/pilif0/open-sea/pull/13).

# Entities
The entities only act as a reference holding different components together.
Therefore I chose to only represent them by numbers.
The two parts of an entity handle are index and generation.
Index corresponds to the index into the entity manager's array of living entities.
The generation corresponds to the value in that array at the index.
An entity is considered alive if and only if the generation at its index matches its generation.
This means that checking whether an entity is alive only takes array access and integer comparison, and therefore takes constant time.

In my implementation I chose to have entity handles be 32-bit integers and split them into 22-bit index and 10-bit generation.
This means that there can be up to 2^22 entities alive at the same time (~4 million) and each index can be reused 2^10 times before the handles start appearing again.
To spread out the usage of indices, there need to be at least 1024 freed indices available for one to be reused.
Otherwise a new one is taken starting with generation 0.
This is of course ignored when there are no more available indices.

In the end, it should take so long for an entity handle to reappear, that all references to its previous incarnation will have been removed.
Without the bound to spread out the index usage, you could see an entity handle reappear within 1024 entities.
For example repeatedly creating and killing entities at least 1024 times would result in repeated reuse of the first index.
Then the first entity would have the same handle as the 1024th.

# Components
In this system, components don't exist as singular objects for each entity.
Instead they are represented as elements in arrays in the component manager object.
The specific layout is left to the manager, as it knows best how its data looks and behaves.
Each manager takes care of allocation of space for its data and should be optimised to operate on many entities at once.

## Model Component
First implemented component is the model component.
This stores the connection between an entity and a model used to render it.
This component stores its data in a struct of arrays, each record containing the entity handle and model index.
The model index refers to an internal storage of the component, that stores a pointer to every unique model used by the entities.
Think of the data arrays as columns in a table and records as the rows.
Each record is at one index in both arrays, and that index is how you interact with it.
To look up indices of records for a set of entities, use the method `lookup(Entity*, int*, unsigned)` and provide it a pointer to the entities you want to look up, the destination where you want the indices written to, and the number of entities.
This shows the orientation towards operating on arbitrary large numbers of entities.

Note that the data arrays are kept tightly packed.
Therefore the indices looked up are only valid for a short time.
Any call to `destroy()` or `gc()` can invaldiate them.
The arrays are reallocated when there is not enough space for new records, so pointers into them are also not stable.

## Tranformation Component
A more complex component is used to store the transformation of the entity.
That is its position, orientation and scale in the world.
It also has an inbuilt hierarchy, so the transformation might not be relative to the identity transformation, but to some other entity's transformation.
The data is stored in a struct of arrays and handled largely as in the model component.
The family relationships are stored as linked lists.
That is, each parent knows the index of its first child and each child knows the index of its parent.
Each child also knows the indices of its previous and next siblings.
If any of these is invalid (for example parent when the entity is a root) then it is equal to -1.

Transformations of entities should be done through the methods provided &mdash; that is `translate(...)`, `rotate(...)`, `scale(...)`, `set***(...)`.
The matrix updates are done immediately when the transformation happens, and are also immediately relayed to all the entity's descendants.
Each record stores its local transformation components and then its world transformation matrix.

## Garbage Collection
Each manager is also responsible for its own garbage collection through its method `gc()`.
That mostly means removing any records belonging to dead entities.
In both currently implemented components, it is done by randomly accessing records and checking whether the entity it belongs to is alive.
If it is not, the record is destroyed.
This is done until a set number of live entities are encountered in a row.
This means when there is nothing to destroy, the method finishes in a small number of iterations.
If there are many records to destroy, it will destroy them quicker than when there are few.
The downside is that with this method records that should be destroyed might linger for a couple runs of `gc()` until they are destroyed.
The threshold of living encounters in a row can be adjusted to minimize this, or if it is critical that none of the records belong to dead entities a different method can be used.

# Systems
Sometimes an action requires data from multiple components.
This action should not be done by a single component manager, as those should be only responsible for their own data.
For this use there are systems.
System is a blanket term for any part of code (usually object) that operates on data from multiple components.
One example of this is the renderer system.

## Renderer
To make use of the model and transformation components for rendering, their data needs to be put together.
This is done by renderers, in this case objets of the `UntexturedRenderer` class.
When the renderer is asked to draw entities by calling `render(Camera, Entity*, unsigned)`, it sets its rendering state (sets a shader program and projection-view matrix) and then it gathers data from the model and transformation components.
It puts this data together into an array of structs that it then uses to set the world matrix, bind the VAO and draw the vertices.

Once I add support for other types of rendering (for example coloured models and textures), I will also add components to bind the new data with entities and renderers to use it.

# Conclusion
This version of the ECS is very different from the one I used in Open Desert.
Most noticably, it is turned by 90 degrees and instead of thinking about each entity and its components in isolation, it thinks about the components as a whole.
This means there is less abstraction (for example no `Component` base class), but this approach should work much better with the large numbers of entities in games.
It is also more suited to C++ with its high degree of memory control.
I will improve the system as I add more components and systems, and see what patterns emerge in the code.
Among the components and systems planned are for example first person controls and camera that follows an entity.
