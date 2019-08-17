---
layout: post
title: Components
categories: [Open Desert]
tags: [entity system, components]
date: 2017-09-26 12:13
---
I am already using a simple entity system in the project.
Currently this is done by inheritance.
As I am working on adding animation to the current implementation of sprites, I am starting to see that I need more flexibility.
From what I've seen in other engines, this kind of flexibility could be provided by using a component system instead of inheritance.
As this would be a big and involved change, I want to first explore how I can best implement it.

# Requirements
First step is sorting out what I need, and want, the resulting solution to do.
The solution:
- Shall provide an easy way to reuse code.
- Shall give acces to the owner from inside of the component.
- Should have the correct type of the owner, not just the superclass `Entity`.
- Shall be easily attached/detached to/from entities.

# Options
The third requirement is optional, but would provide A LOT of simplification when using the components.
In the current system, there are two ways this could be done.
One is having an implementation of the superclass `Component` for each of the various entity types.
Second is using generics and passing the actual entity type that way.

The first solution can be discarded quickly, as that would make adding entity types require significantly more boilerplate code.
The implementations would be nearly identical, with just a few member types different.
The second solution is better, but would still add more code that usually isn't necessary, without increasing flexibility by much.

There is another option if you remove the assumption that we do not want to change the current entity system.
That option is making everything into a component and making the entities basicaly just containers for components.
From what I've seen in other engines, this is how most of them work.
And I can now see why that is.
This option provides a lot of flexibility, and also makes it easier to specify entities in external files.

# Decision
I have therefore decided to go for the most complicated but possibly the most reasonable option, redesigning the entity system.
This will require much more work now, but save me work later.

The specifics of the redesign will come in a different post when I finalise the plan.
