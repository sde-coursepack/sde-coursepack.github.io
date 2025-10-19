---
Title: Anti-Patterns
---

* TOC
{:toc}

# What are Anti-Patterns?
Anti patterns are design patterns that are bad—ones that negatively effect the internal and external qualities of software. Examples of anti-patterns include:

* Big Ball of Mud 
Big Ball of Mud describes a system that lacks any perceivable architectural structure and organization. Such a system might result from developers rushing to build a system or neglecting long-term maintainability concerns, adding modules and components haphazardly without imposing any structure or refactoring as the system grew in size. 

* The God Object
The God Object describes a situation in a system where one module (a class or object) interacts with all other modules. For example, one god object could be a class containing global variables that are used in other modules throughout the system. Effecitvely, all of the other modules depend upon the god object for their information and functionality. Furthermore, the god object is likely to become large in size and unorganized in its purpose, attempting to do "too much" as the system grows in size.


In each of these situations, the poor organization of the code results in increased software entropy-- as the system grows in size, it will become increasingly difficult to navigate, maintain, and change without causing an issue in the existing code. Furthermore, in the case of a bug, it can be very messy to track down its origin and fix it without causing even more issues. Since these consequences are all bad signs for software development costs and the reliability of the system, these anti-patterns should generally be avoided in development.

