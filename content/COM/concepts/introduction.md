---
publish: true
title: Introduction
created: 2026-08-31T04:42:47.240Z
---

# Preliminaries

## Encapsulation, Inheritance and Polymorphism.

```cpp
class enemyNPC
{
	public:
		virtual void attackPlayer(void);
};  //  Encapsulation

class someEnemyFaction : enemyNPC  //  Inheritance
{
	public:
		void attackPlayer(void);
};

someEnemyFaction *factioned_enemy = new someEnemyFaction;
enemyNPC *enemy = factioned_enemy; //  Polymorphism
enemy->attackPlayer();
```

- `enemyNPC` is a depiction of an interface that would be inherited by a COM object.
- `someEnemyFaction` is a depiction of a COM object that would typically inherit various individual interfaces. It is the COM object's responsibility to support the individual interfaces that it supports.

Think of the interface as a "template" or a "frame" into which you fit in a COM object that you define along the lines of this "socket" (thus the virtual keyword for implementations within interfaces in C++). It should "fit". When you inherit an interface, it is expected from you to define the object that would work with the interface. The interface, in turn, would use _your_ specifically defined COM object as the basis of its further operations. You can define this COM object in any language as long as the interface understands it.

When roles are different but the understanding needs to be mutual.
_My understanding of things should bode well with your understanding of things. A barber needs to understand your expectation of a haircut, but he can't give you a haircut unless you tell him which "style" or how much "density" to maintain. Once you tell him that, he is the body of action who will execute the hair cut as per your criteria. How he holds the blade, comb and scissors and judges how deep to cut is abstract/unknown to you._

In our example, "enemy" has taken _your_ version of what you think is an enemy (i.e. someEnemyFaction) and taken it in stride because it understands it, because someEnemyFaction was based on its own (enemyNPC's) definition.

enemyNPC is your interface. someEnemyFaction is your COM object based on the interface.

But But But just remember one thing...

> A COM object never exposes itself or any of its data to the outside world; there is no concept of a pointer to a COM object. Instead, COM objects provide pointers to interfaces that they expose.

==USER CONTROLLED OBJECT CREATION==

| Technology     | Description                                                                                                                                                                                                                                                     |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| OLE Automation | The ability of an application to control another application's objects programmatically. An integral part of OLE Automation is the ability of an object to describe its capabilities through type descriptions. This feature is absolutely key to OLE controls. |
| OLE Documents  | All the document-centric features of OLE, such as linking and embedding, drag-and-drop, and visual editing.                                                                                                                                                     |
| OLE controls   |                                                                                                                                                                                                                                                                 |

## Binary re-use

Your windows dialog boxes are an example of binary re-use

## OLE

"Glue between objects". A specification for interfaces between objects. Rather than using "inheritance", which demands that interfaces between elements in each level of a hierarchy remain the same, OLE uses "containment", which is a technique allowing one object to embody another and to expose any number of the contained object's interfaces as its own.

# Significance?

- **RELIABLE software RE-USE**: Re-inventing the library wheel for each software was a huge waste of time, so certain standard functionality in various software could be **interfaced**, meaning it will work consistently with any other object that uses it.

## "I still don't understand what it is"

\~~You know how there are several libraries in Linux in a certain directory consisting of things like shared objects and libraries which are used by most programs? COM provides more or less the same thing, except it manages IPC, language neutrality and runtime object lifecycle for the developer. Developers could also add their own components that work across applications. In short, you can skip writing "glue code" and rely on a system that does it for you at an operating-system-wide level.~~.
Think of it as a female socket (the interface) to which a male socket (the COM object) attaches to. The COM object knows the inner shape and working of the interface, but adds some of its own implementations as well.

# Concept

## Programming Model

- Client-server, object-based programming model designed to promote software interoperability.
- Goal is to provide a means for client objects to make use of server objects, despite the fact that the two may have been developed by different entities, using different programming languages, at different times.
- To achieve this, COM defines a binary standard, which specifies how an object is laid out in memory at run time.
- By defining how an object is laid out in memory, COM allows any language that is capable of reproducing the required memory layout to create a COM _object_.

![[COM/svg/COM.svg]]

https://learn.microsoft.com/en-us/windows/win32/com/com-technical-overview

Think of the working of COM as "astral projection", where a sleeping body with its attributes and functions (dll) is walking in the form of pure energy (an instance). It is most alive in pure energy form and can interact with the world physically. It can also multiply. And to control it, you control the instance(s), not the binary format.
