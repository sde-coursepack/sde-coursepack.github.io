---
Title: Java Object Class
---

* TOC
{:toc}

# The `Object` Class

All Java class inherit from a `superclass` called Object, either directly or indirectly. This means whenever you create class, somet methods are inherited.

## toString()

The `toString` method is used to define a String representation of a class. For instance, when you print an instance of a class, it prints whatever the `toString` method returns.

**Default behavior:** returns `"[Class Name]@[memory address]`

## equals(Object o)

The `equals` method is used to define how to objects are judged for equality. For example:

```java
   a.equals(b);
```

You can define how to compare `a` to `b`. Note that the method takes in any `Object`, not just whatever the instance of `a` is. As such, most methods will use `instanceOf` to do a type check.

**Default behavior:** returns true **if and only if** the calling instance and parameter instance are stored at the same memory address (that is, they are *literally* the same instance)n

## hashCode()

Hashcode is used to produce a "random-looking" integer to describe an object for the purposes of storing in hashed datastructures (such as hash-sets and hash-maps).

**Default behavior:** Returns the memory address of the object represented as an int.

### equals() and hashCode() go together

You should always ensure that if two instances are "equal", that they produce the same hashcodes, whatever equal means for that particular class. Note that it is possible for two instances that are not equal to each other to produce the same hashcode. For example, the `String`s "VII" and "Ugh" produce the same `hashCode` value. But if two instances are considered equal, they must produce the same hashCode value.

### No mutable fields in hashCode()

Additionally, if you have any mutable fields in a given object, the value of that field should never be used in a hashCode function. This is because if that value changes, the object will no longer produce the same hash code. This would cause significant problems if you use that object in a HashMap or HashSet, as an object already in the map/set could become "lost", since it's location is based on a hash value that has changed.

