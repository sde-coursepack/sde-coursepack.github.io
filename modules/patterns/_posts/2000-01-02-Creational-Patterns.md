---
Title: Creational Patterns
---

* TOC
{:toc}

## Acknowledgement

This page in its current form was authored by teaching assistant Jade Gregoire, adapted from my class notes and slides.

# Summary
Creational patterns are any design patterns that either hide or limit constructor usage. Essentially, they provide object creation and instantiation mechanisms that increase flexibility and re-use of existing code. Examples of creational design patterns include:
* Singleton
* Factory
* Abstract Factory

# Singleton
Singletons are used when you want only one instance of an object to exist at any given time. This instance can be shared amongst different modules without the modules beng aware of each other. Singletons are commonly used for: 
* Logging
* Client side UI settings
* Service locators

# Factory 
Factories are useful for combining existing pieces of code into one class. They provide an interface for creating objects in a superclass, allowing subclasses to alter the type of object that will be created.

As an analogy, think of a car: each car will have shared elements that we do not want to build from scratch for each car type when we know they will be the same. Instead, we can think of a car as interchangeable pieces and a system (the factory) being able to put these different pieces together depending on the car type demanded. 

# Abstract Factory 
Similarly, abstract factories let you produce families of related object without specifying concrete classes. This pattern is used when you have multiple factories that produce the same abstract goods but have their own concretions of the product. At runtime, the concrete factory is picked based on what you want to use.
