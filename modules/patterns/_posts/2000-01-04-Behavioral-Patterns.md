---
Title: Behavioral Design Patterns
---

* TOC
{:toc}

## Acknowledgement

This page in its current form was authored by teaching assistant Jade Gregoire, adapted from my class notes and slides.

# Summary
Behavioral design patterns take care of communication between objects and the assignment of responsibilities between them, by manifesting flexible behavior. Behavioral design patterns include:
* Iterator
* Observer
* Command
* Strategy

# Iterator
Iterators allow you to visit all elements of collections one at a time. You've most likely seen this before: [Java has a built-in iterator pattern](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html). But, why use an iterator instead of loops? The key advantage of iterators is functional independence and information hiding: someone does not have to know how a collection is structured, does not have to worry about how the iterator works, and they can trust that the iterator will allow them to visit every element. 

# Observer
When you have objects that need to notify a varying list of other objects that some event has occurred (such as a variable change, or a method being called), an observer may the solution. An observer calls for an observer interface and a concrete observer that implements it, while looking at another concrete class. 

An example of this would be a weather data application: a list of users (observers) may want notifications about weather updates in their area. The app should be able to add and remove observers (if the users want to unsubscribe), and whenever the weather forecast gets changed or updated, the users should be notified of that change (with a update() function that gets called on each observer). 

# Command
The command pattern encapsulates information needed to perform an action. A command interface has the execute() method—and whenever a cncrete implementation of command is sent, their specific execute method runs. 

# Strategy
Sometimes, we want to specify a particular part of an algorithm (i.e., a “strategy”) in a given context at runtime. The strategy design pattern uses a strategy interface and concrete strategy classes. The concrete strategy is passed as an instance to the method that implements the rest of the algorithm at runtime.
