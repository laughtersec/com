---
publish: true
title: COM Object Re-use
created: 2026-08-31T04:42:47.224Z
modified: 2026-08-31T13:09:37.999Z
tags:
  - theory
---

Often the COM objects at your disposal may provide only a limited subset of the functionality that you desire; when that happens, you must develop new components that fully implement the desired level of functionality. However, COM provides two techniques for reusing and extending the functionality of existing COM objects.

# Containment

Containment is the simplest form of COM object reuse. In containment, the new COM object, known as ==the outer object, simply acts as a client of the existing COM object==, which is known as the _inner_ object. Like all COM objects, the outer object exposes its own set of interfaces. However, instead of being solely responsible for providing the implementation for each of its interface functions, the outer object relies on the functionality supplied by the inner object for assistance.

![[COM/concepts/svg/containment.svg]]

Here is an example:

"Insert example here"

# Aggregation

> Aggregate
> "A material or structure formed from a mass of fragments or particles loosely compacted together"

The **material** here is the COM aggregate, and the **mass of fragments** here are the outer and inner COM objects.

The inner object's (the COM object's) interfaces are exposed directly as if they were implemented on the outer object itself.
Unlike containment, in which the inner object has no idea that it is being used as part of another object, in aggregation, the inner object is not only aware that it is being used as part of an aggregate, it must be developed specifically to support aggregation. Typically you can only reach the interface of a COM object via a pointer to it, and not the interface of another COM object. Querying interfaces that belong to a COM object to which we don't have a pointer to is not conventionally possible. So how do we "aggregate" two COM objects?

![[COM/concepts/svg/aggregation.svg]]

Thankfully, the COM library provides `CoCreateInstance`, which enables the much needed "cooperation" between the inner and outer objects (namely the pointer to the `IUnknown` interfaces of both objects). When the outer object creates the inner object using `CoCreateInstance`, the two objects exchange `IUnknown` interfaces. The outer object supplies a pointer to its `IUnknown` interface to the inner object as the second parameter to `CoCreateInstance`, while the inner object returns a pointer to its `IUnknown` interface to the outer object as the final parameter to `CoCreateInstance`.

If a client has a reference to an interface of the outer object and performs a `QueryInterface` call for an interface supported by the inner object, the outer object simply delegates the `QueryInterface` call to the inner object using its reference to the inner object's `IUnknown` interface. Likewise, if a client has a reference to an interface of the inner object, and performs a `QueryInterface` call for an interface supported by the outer object, the inner object simply delegates the `QueryInterface` call to the outer object's `IUnknown` interface.

# Which should I use?
