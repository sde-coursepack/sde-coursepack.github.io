---
Title: Structural Patterns
--- 

## Acknowledgement

This page in its current form was authored by teaching assistant Jade Heilemann, adapted from my class notes and slides.

# Summary
Structural design patterns have to do with de-coupling an abstraction from its implementation. Patterns under this group type typically assemble objects and classes into larger structures by combining multiple classes or bringing together existing objects. Structural design patterns include:
* Adaptor
* Facade
* Proxy
* Bridge
* Decorator

# Adaptor 
Adapters adapt an existing class or object to a new interface without changing the underlying class. They are useful to update interfaces while minimizing side effects/propagation of changes. This allows objects with incompatible interfaces to collaborate, by converting the interface of one object to something another object can understand. 

# Facade
To solve issues of temporal coupling and complex dependencies, a facade is usually a great solution. A facade olves these issues by providing a simplified interface to a library, framework, or any other complex set of classes. However, developers should be careful to not turn a facade into a god object or class. 

# Proxy
When facing an issue where you have a massive object consuming resources, but the object is only needed from time to time, you should probably use a proxy design pattern. Proxy patterns create a proxy class with the same interface as an original service object. 

To implement a proxy: First, create a service interface that declares the interface of the desired service. The service and proxy class will both impement this interface. The service class provides some sort of useful business logic. The Proxy class has a reference field that points to a service object. After the proxy finishes its processing, it passes the request to the service object. 

# Bridge
Bridges are used to split a large class into two separate hierarchies, an abstraction and its implementation. This allows for the mixing and matching of certain object properties. Bridges follow the open/closed principle: you can introduce new abstractions and implementations independently from each other. It also follows the single responsibility principle: you can focus on high-level logic in the abstraction, and on platform details in the implementation. Bridges are also good for not exposing platform details.

The abstract factory design pattern can be used alongside the bridge design pattern. The abstract factory can encapsulate the relationship between the abstraction and implementation, and hide the complexity from the client code. It is also iportant to note that a biirdge is usually designed up-front, whereas an adapter is used with an existing app to make incompatible classes work together.

# Decorator
Decorators are amazing for attaching new behaviors to objects! This is done by placing objects inside special wrapper objects that contain any new behaviors (so you can manually wrap, or add-on each new behavior wanted). All wrappers and base objects must follow a common interface. 
