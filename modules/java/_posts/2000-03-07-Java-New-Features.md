---
Title: Testing With Exceptions
---

# Java New Features

## Lambda Bodies and Streams

This new feature will be covered in depth in the "Code Quality" module.

## var keyword

In Java 10, java introduced the `var` keyword that allows you to create local variables with an *implied* type.

```Java
import java.util.*;

public class VarExample {
    public List<Integer> listOfFirstNNumbers(int n) {
        var numberList = new ArrayList<Integer>();
        for (var i = 0; i < n; i++) {
            numberList.add(i);
        }
        return numberList;
    }
}
```

In the above, numberList will be an `ArrayList<Integer>` and i will be an `int`. Just like any other
Java variable, the type of the variable **still cannot be changed after initialization**.

Note that the data type must be derivable *at compile-time*. So something like `var x = null` is *not* allowed, because the data
type cannot be derived when the line is compiled. Additionally, you must immediately initialize the variable, so you cannot,
for example, say:

```java
    var x;
    x = 5;
```

The variable must be initialized at the time it is created.

Additionally, `var` **can only be used for local variables.** You cannot use `var` with class fields. Additionally,
you cannot use `var` as a return-type or parameter type for methods.

## Records

## Text Blocks

## Switch Expressions

## Sealed Classes