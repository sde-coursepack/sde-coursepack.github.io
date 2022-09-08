---
Title: Refactoring - Extracting a Class
---

# Extract a Class

In this module, we will discuss the refactoring of extracting a class. We may extract a class for the following reasons:

* A single class contains unrelated information, so we separate into two or more classes so that each class is more **cohesive**.
* Two classes share enough in common that they should be combined into a single hierarchy, so we extract a parent-class to reduce code repitition
* A function is complicated and requires a large number of local variables, meaning it may really be a class in disguise.