---
Title: Anti-Patterns
---

* TOC
{:toc}

# What are Anti-Patterns?

Anti-patterns are design patterns that are bad -- ones that negatively affect the internal and external qualities of software. Examples of anti-patterns include:

* **Big Ball of Mud:** Big Ball of Mud describes a system that lacks any perceivable architectural structure or organization. Such a system might result from developers rushing to build a system or neglecting long-term maintainability concerns, adding modules and components haphazardly without imposing any structure or refactoring as the system grows in size. The lack of organization in a Big Ball of Mud system causes issues for understandability -- it is difficult to determine how the system works and what each component is trying to do. The system's maintainability also suffers: given a new feature to implement, it is not immediately clear where and how to integrate the new code with the existing codebase, and in the case of a bug, it is difficult to track down where in the code the culprit of the incorrect behavior lies.

* **The God Object:** The God Object describes a situation in a system where one module (a class or object) interacts with all other modules. For example, one god object could be a class containing global variables that are used in other modules throughout the system. Effectively, all of the other modules depend upon the god object for their information and functionality. Furthermore, the god object is likely to become large in size and unorganized in its purpose, attempting to do "too much" as the system grows in size. Recall the Single Responsibility Principle (SRP): a class should only have one reason to change. With a god object that contains data or methods needed by various different modules, changes to any one of these other modules might require changes to the god object, violating the SRP. Generally, it is better to have smaller, cohesive modules, each with a single well-defined purpose, than to have a large, all-encompassing module (like a god object) that attempts to do many different, unrelated things.

In each of these situations, the poor organization of the code results in increased software entropy -- as the system grows in size, it will become increasingly difficult to navigate, maintain, and change without causing an issue in the existing code. Furthermore, in the case of a bug, it can be very messy to track down its origin and fix it without causing even more issues. Since these consequences are all bad signs for software development costs and the reliability of the system, these anti-patterns should generally be avoided in development.
