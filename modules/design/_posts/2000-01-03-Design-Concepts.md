---
Title: Design Concepts
---

# Design Concepts

Underpinning our design decisions are some key concepts. These concepts apply to any system we are designing, whether we are in a structured programming language like C, an object-oriented programming language like Java, a front-end centric language like JavaScript, or in a functional programming language like LISP. While in this course we focus on using Java, keep in mind that these specific concepts extend into all programming languages and paradigms.

* TOC
{:toc}



**In this module**, we give a brief summary of each concept. We then dive deeper into each concept in the next 4 modules.

## Modularity

Idea of modularity is that we break up our big problem into little problems. This is because little problems are easier to solve, and each unit is potentially reusable. We call each individual **unit of code** a module. This is inherently defined somewhat vaguely, as a module can mean very different things in different languages. For example, in Java, we typically think of modules as **classes**, whereas in non-OO languages, such as C, we tend to think of modules as a function or group of tightly-related functions in a single file. Note that a module can, itself, contain sub-modules. In this way, we can decompose our system hierarchically, with more abstract higher-level modules which depend upon more concrete lower level modules.

For example, lets say you are writing one particularly complex function. To make the code easier to test/understand, you start using helper functions (small functions that handle one part of the function). Every time you write a helper function, you are breaking up on big module into smaller modules. These smaller modules are, ideally, easier to understand and solve one part of the bigger problem. This process is called **decomposition**, and it is the heart of design: "How do we decompose our software?"

In modularity, we are concerned with how we **decompose** (break up) our software system into smaller parts. We can do this *functionally* as well as with classes (and often use both). 

A key difficulty in modularity is deciding:

1) How do we break up our solution into modules?  
2) How do we recombine these modules and ensure they work correctly together?  

When seeking modularity, we generally want modules that are *highly cohesive* (each module is narrowly defined around one particular feature/need) and *loosely couple* (interactions between modules are as simple as possible).

## Functional Independence

As we decompose our system into modules, we want each module to be *functionally independent*. That is, each module can perform its purpose with minimal interactions with the rest of the system. When interactions are necessary, as they will certainly be, those interactions are defined in as simple a way as possible, with as little information necessary as possible to perform the transaction. The "gold standard" of functional independence is when two modules only interact through passing arguments and receiving return values via public functions.

We call the measure of how interdependent two modules are "coupling". The more tightly coupled two modules are, the more difficult they are two develop independently. That is, a change in one module could create difficult-to-foresee side effects in another. Instead, it is preferable to have **loose coupling**.

This is often why static global variables are something we want to avoid in programming, as when several functions use the same static global variables, it becomes harder and harder to predict all the possible situations are functions will use that global data.

Coupling can be harder to understand as a newer programmer, as coupling only becomes obviously a problem after we need to change or modify existing software. When designing our software, or writing new software, we often may introduce coupling (intentionally or otherwise) with really "feeling" the impact that coupling creates. However, later, when changes need to be introduced, we may find it becomes difficult to separate two modules, or replace a module, due to over coupling.

## Abstraction

Abstraction is related to information hiding. **Abstraction** is the process of hiding all unnecessary details and exposing only required details. Typically, this means that we hide **implementation** details behind a minimal **interface** (be aware that we are using "interface" in the generic sense, not the Java keyword `interface`). 

Ultimately, we use our modules to **encapsulate** some behavior or data. For example, when you are using a Java `ArrayList<String>`, there is an underlying `String[]` that *is* the array the name of the list describes. However, you never need to access to that underlying Array. Instead, you have access to a set of behaviors, like `add`, `get`, `remove`, etc. All of these features **use** the underlying array. However, *how* they are used isn't important to understanding how to use the ArrayList.

In the same vein as `ArrayList`, we want to design our code so that users only have to understand the **interface**, not the **implementation**. 

Ultimately, the goal of abstraction is to separate implementation and interface such that, so long as the interface is maintained, changes to the implementation will not force changes to other modules.

## Information Hiding

Information hiding is the mechanism by which we create a limited **interface** to achieve abstraction. That is, beyond just encouraging a client (that is, some other part of our program using our module) to use our modules via the **interface**, we want to intentionally force clients to *only* use the interface. This ensures that our module is only used correctly, as any incorrect usage is prevented syntactically.

This is why in Java, C++, any many other languages, we typically use `private` fields, even if we have `public` getters and setters. If we create a module, and expect client classes to directly modify underlying data, then the client needs to understand **more information** about our module to use it correctly. Instead, we want our module to be usable with as *low a barrier of understanding as possible.*

A note that Python actually doesn't support rigorous information hiding. While it is convention in Python to use an underscore to indicate a private variable (for example, _size), this is only a convention. That is, there is no syntactical backing preventing you from directly accessing and modifying private fields and methods of another class.

## Trade-offs

This module is design simply to introduce the over-arching principles and vocabulary you want to use when considering software. However, be aware that there are always trade-offs. We will explore these trade-offs in the following modules.
