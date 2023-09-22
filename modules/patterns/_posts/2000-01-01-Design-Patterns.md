---
Title: Design Patterns
---

* TOC
{:toc}

## Acknowledgement

This page in its current form was authored by teaching assistant Jade Gregoire, adapted from my class notes and slides.

# What are design patterns? 
Design patterns are a toolkit of solutions to general problems that arise in software design. A non-coding example of a design pattern would be meme structures: various memes will utilize a format with text at the top or bottom of an image, because they know that format works. Design patterns work in a similar manner, where the tried and true solutions can be applied to improve software quality. However, every design pattern is not necessarily applicable to each issue. 

# List of Design Patterns
In this course, we will review three major groups of design patterns: 
* Creational
* Structural
* Behavioral 
Each pattern type addresses a certain kind of problem that arises in code.

## Creational Design Patterns
Creational patterns are any design patterns that either hide or limit constructor usage. Essentially, they provide object creation and instantiation mechanisms that increase flexibility and re-use of existing code. Examples of creational design patterns include:
* Singleton
* Factory
* Abstract Factory

## Structural Design Patterns
Structural design patterns, on the other hand, have to do with de-coupling an abstraction from its implementation. Patterns under this group type typically assemble objects and classes into larger structures by combining multiple classes or bringing together existing objects. Structural design patterns include: 
* Adaptor 
* Facade 
* Proxy
* Bridge
* Decorator

## Behavioral Design Patterns
Our last group of design patterns is the behavioral design pattern. These take care of communication between objects and the assignment of responsibilities between them, by manifesting flexible behavior. Behavioral design patterns include:
* Iterator
* Observer
* Command
* Strategy

# Things to keep in mind
> "When you have a hammer, everything looks like a nail." 
<br>- Abraham Maslow

Design patterns should never be forced, or over-used. While they are great tools for addressing issues in software quality, it is easy to misuse design patterns, especially if you use too many or attempt to use a pattern when there is a better solution available. 

Above all, be aware that **over-design** is a thing. Remember the [YAGNI principle](https://sde-coursepack.github.io/modules/design/Design-Principles/#yagni-principle)!