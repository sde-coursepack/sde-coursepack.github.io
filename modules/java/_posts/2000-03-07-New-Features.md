---
Title: New Features
---


# Java's New Features
{: .no_toc }

In this module, I will show some new features in Java that have been added
with Java 8 or later. Please note that this is not an exhaustive list, as there
are tons of additions. [This Wikipedia page is a good starting point](https://en.wikipedia.org/wiki/Java_version_history)
if you want to see all additions. Here, I am highlighting a view that I personally
have found useful.

---

## Contents
{: .no_toc }

* TOC
{:toc}

---


## Lambda Bodies and Streams
__Introduced in Java 8__
This new feature will be covered **in depth** in the [Code Quality](https://sde-coursepack.github.io/modules/refactoring/Code-Quality/) module.

However, as a quick useful example, consider a sorting example we looked at in
[Prerequisite Knowledge](https://sde-coursepack.github.io/modules/java/Prerequisite-Knowledge/).

From the class [Enrollment.java](https://github.com/sde-coursepack/java-prerequisite/blob/main/src/main/java/example/Enrollment.java), line 23-25:

```java
    public void sortStudentsByName() {
        Collections.sort(students, new StudentNameComparator());
    }
```

And from the class [StudentNameComparator.java](https://github.com/sde-coursepack/java-prerequisite/blob/main/src/main/java/example/StudentNameComparator.java):

```java
    public int compare(Student s1, Student s2) {
        int lastNameComparison = s1.getLastName().compareTo(s2.getLastName());
        if (lastNameComparison != 0) {
            return lastNameComparison;
        } else { 
            return s1.getFirstName().compareTo(s2.getFirstName());
        }
    }
```

Written this way, every single way we could sort students would require
writing a separate Comparator class. While there's nothing *wrong* with this
from a design perspective, it can add a lot of boilerplate classes that all
basically do the same thing, but just sort on different fields.

Instead, we can *remove the need to write a separate class* by using a Lambda body. We can replace
Line 23-25 in [Enrollment.java](https://github.com/sde-coursepack/java-prerequisite/blob/main/src/main/java/example/Enrollment.java), line 23-25.
with:

```java
    public void sortStudentsByName() {
        students.sort((x, y) -> {
            if (x.getLastName().equals(y.getLastName())) {
                return x.getFirstName().compareTo(y.getFirstName());
            } else {
                return x.getLastName().compareTo(y.getLastName());
            }
        });
    }
```

And now, we no longer need the class StudentNameComparator.java at all!

While we won't dive deep into the mechanics of this yet, we are effectively defining
our own `compare(Student x, Student y)` function **in-place**. This means
we are taking all the complexity of sorting and encapsulating it
entirely within one function in the Enrollment class, rather than creating
a separate class which adds to the complexity of the whole code base.

---

## var keyword
__Introduced in Java 11__

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
you cannot use `var` as a return-type or parameter type for methods. Always remember that `var` does not change the nature of a Java variable. The variable still has a specific type that cannot change.

---

## Records
__Introduced in Java 16__

A record is a useful, shorter way of declaring a Java *Data Transfer Object*, or **DTO**.
A DTO is useful as a class for modeling data that can easily be passed between
different modules, tiers, and layers of a system. The idea is that these objects
should be used **only to store tightly coupled information**. The class shouldn't
*do* anything else. 

Think of DTOs as sort of like a paper form, where the form has
very specific fields to fill out. Once filled out ("instantiated" in Java
terms), the form can be copied, passed to other departments, etc. However, the
form only exists to show collected information. The form, by itself, can't do anything.
However, each department that receives the form can do something with it.

Here is an example of a Java Record below:

```java 
public record Employee(
        int id,
        String firstName,
        String lastName,
        String email
) {

}
```
This `Employee` record is a data type that has an id, first and last name, and an email.
You'll notice that the record has no class body. While you can add a body to the record, 
you often won't need to, because the record keyword automatically gives the class:

* getters  
* useful `toString()`
* useful `equals(Object o)` and `hashCode()`

You'll notice that I *didn't* list **setters**. That's because each field in the
`Employee` record are `private final` variables. That is, after an Employee record
is created, you cannot modify the fields. That is, each record is **immutable**[^1]. As
such, if you are modeling data that will need to change, you likely will still
want to use a Java class instead of a record.

However, this doesn't allow us to avoid writing a lot of boilerplate code. With just
the above `Employee` record, we can do the following:

### instantiate records
By default, we are given a Constructor that lets us specify the value
of each field. Remember, because these variables are *final*, we cannot
change their value after instantiation.

```java
    Employee steve = new Employee(15, "Steve", "Smith", "ssmith@company.com");
```
### get field values

We are given a *getter* for each field, which is just the field's name as a 
zero-argument method.

```java
    System.out.println(steve.id());
    System.out.println(steve.email());
```

### print
We are given a default `toString()` method that returns a string listing
the record's data type, and the value of each field. This code:

```java
    System.out.println(steve);
```

...prints out:

```
    Employee[id=15, firstName=Steve, lastName=Smith, email=ssmith@company.com]
```

Note that you can override the toString() method if you wish.

### check equality and store in hash tables

Records also give you a default `equals(Object o)` and `hashCode()` function.
The `equals` method returns **true** if-and-only-if *the values of every field
are equal*.

```java
    Employee alsoSteve = new Employee(15, "Steve", "Smith", "ssmith@company.com");

    // prints true
    System.out.println(steve.equals(alsoSteve));
```
The `hashCode()` function similarly uses all fields to form the `hashCode()`.

---

A downside of `record`s is that, as yet, they don't work with many ORMs like Hibernate. If you aren't familiar with ORMs, don't worry. It's something we will cover later in the class.

## Text Blocks
__Introduced in Java 15__

In Java, we generally assign long `String` values over multiple lines. Example:

```java
    String longText = "This is a really really really really really really " +
                "really really really really long String";
```

However, this approach can be annoying when trying to format data. This is because
we have to add our own `\n` and `\t` characters, as well as any other white space,
to get the output formatted how we want.

For example, say if we wanted to print a String that looked like:

```java
    1. This is an outline
        a. outlines have major parts with numbers
        b. and subparts with letters
    2. This is a major part
        a. This is a sub-part
            i. we can use RomanNumerals to go deeper
```

With Java as you have used it, that will probably look something like:

```java
String toPrint = "1. This is an outline\n" +
    "\ta. outlines have major parts with numbers\n" + 
    "\tb. and subparts with letters\n" +
    "2. This is a major part\n" +
    "\ta. This is a sub-part\n" +
    "\t\ti. we can use RomanNumerals to go deeper\n";
```

However, with Java Text Blocks, we can do this much more easily:

```
String toPrint = """
    1. This is an outline
        a. outlines have major parts with numbers
        b. and subparts with letters
    2. This is a major part
        a. This is a sub-part
            i. we can use RomanNumerals to go deeper""";
```

This is an example of using a text-block. Java will preserve the format of your
text, including new-lines, indents, spaces, etc. For you.

You can also use this with the *formatted* function to generate a String from
a textblock with variable values plugged in. Example:

```java
String studentInfoWithGPA = """
    Name: %s
    Computing ID: %s
    Student Number: %d
    GPA: %.2f
    """.formatted(name, compID, studentNumber, getGPA());
```

If you are unfamiliar with the formatted function, here is a [good tutorial on GeeksforGeeks.](https://www.geeksforgeeks.org/java-string-format-method-with-examples/).

---

## Switch Expressions
__Introduced in Java 14__

Switch statements, which are *not new*, let you do something like:

```java
    public int getDaysInMonth(int month) {
        switch (month) {
            case 1, 3, 5, 7, 8, 10, 12:
                return 31;
            case 4, 6, 9, 11:
                return 30;
            case 2:
                return 28;
            default:
                throw new IllegalArgumentException(
                        "Invalid month number: " + month);
        }
    }
```

Switch expressions let us shorten the above to:

```java
    public int getDaysInMonth(int month) {
        return switch (month) {
            case 1, 3, 5, 7, 8, 10, 12 -> 31;
            case 4, 6, 9, 11 -> 30;
            case 2 -> 28;
            default -> throw new IllegalArgumentException(
            "Invalid month number: " + month);
        };
    }
```

We can also use it in assignment statements:

```java
    public String getDayOfTheWeekName(int dayOfTheWeek) {
        int normalizedWeekday = dayOfTheWeek % 7;
        String nameOfDay = switch(normalizedWeekday) {
            case 0 -> "Sun";
            case 1 -> "Mon";
            case 2 -> "Tues";
            case 3 -> "Wednes";
            case 4 -> "Thurs";
            case 5 -> "Fri";
            case 6 -> "Satur";
            default -> "IGNORED";
        };
        return nameOfDay + "day";
    }
```

This syntax is very similar to Lambda bodies, which we will cover in the 
[Code Quality](https://sde-coursepack.github.io/modules/refactoring/Code-Quality/) module.

---

## Sealed Classes and Interfaces
__Introduced in Java 15__

We like to use Java `interface`s and `abstract class`es in Java for polymorphism.
However, because interfaces and abstract classes are generally `public`, it prevents
us from controlling which classes can implement an interface or extend an abstract class.

**Generally, this isn't a problem.** In fact, the very idea of polymorphism exists to
support the *Open-Closed Principle:* 

>"software entities (classes, modules, functions, etc.) should be *open for extension*
>but *closed for modification*." Bertrand Meyer, __Object-Oriented Software 
>Construction__ (1988).

However, if we want to prevent a class or interface from being extended, we can make it
a **sealed class**.

Example:

```java
public sealed interface TimeKeeper permits StopWatch, IntervalTimer {
    void startTimer();
    long getElapseTimeInMilliseconds();
    void stopTimer();
}

```

This means that *only* the classes `StopWatch` and `IntervalTimer` can implement
`TimeKeeper`. That is, those two classes are the only valid classes to use the
TimeKeeper interface.

In general, if you are unsure if you should make an interface/class sealed, **then you should not**.
However, this can be useful *if you intentionally want to restrict an interface or
class from being extended.*

---

Footnotes:

[^1]: - Note that you can have mutable fields in a record. For example, you can have
a record which contains a List or Map. This lets you still use things
like the ``add()`` functions on those fields. However, **this is bad design**. If you 
plan for a class to be mutable, you generally shouldn't use Java `record`s. 
Additionally, these field objects cannot be
re-instantiated after the Employee record is created. If your record has such fields,
you *must override the `equals` and `hashCode` functions* so that they do not
rely on any fields with a mutable datatype.