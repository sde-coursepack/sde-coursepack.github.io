---
Title: Prerequisite knowledge
---

# Prequisite Knowledge

Before continuing further, I want to take time and clarify what we
expect you to come into the course ready to do. While most people
in this course will be coming from CS 2100, some people may be coming
from other programming backgrounds.

This list is meant to briefly summarize what you are expected to know
and be comfortable with in the Java programming language. This means
both being able to read code and write code that uses the following
tools and techniques.

## Control Flow

* You should be *very comfortable* with ```if```, ```else```, and ```else if```
  * You should also be aware of ```switch``` statements. Example:
```java
public String getDayName(int day){
    switch(day) {
        case 0:
            return "Sunday";
        case 1:
            return "Monday";
        ...
        case 6:
            return "Saturday";
        default:
            throw new IllegalDayException();
    }
}
```
* You should be very comfortable with ```for``` and ```while``` loops
  * ```for(int i = 0; i < max_number; i++)``` 
  * ```for(Student student : listOfStudents)```
  * ```while(x > 0)```
  * How to detect and prevent infinite loops.

## Functions

You should be *very comfortable* writing functions in Java, and understand:
* Return types, including void
* Input parameters
* Local and global scope
* Return value
* Side Effects
* How to test functions via "main-method" testing

## Classes

You should be *very comfortable* with Java Classes
* Using pre-defined Java classes, like String and ArrayList
* Writing your own Java classes
* The difference between a class and an instance
* Methods and fields
  * getters and setters
* Constructors
  * Using constructors with the ```new``` keyword
  * Writing Constructors
* The ```static``` keyword and what it means
* Class field and method visibility with ```public``` and ```private``` keywords.

## Object Orientation

You should be familiar with:
* The Java ```interface``` and ```implements``` keywords
  * You should be aware of the interfaces
    * Java Collections interfaces, like ```List```, ```Set``` and ```Map```
    * ```Comparable<E>```
    * ```Comparator<E>```
    * ```Runnable```
* The Java ```extends``` and ```abstract``` keywords
* The Java ```Object``` class
  * ```Object``` class methods like ```equals(Object o)```, ```toString()```, and ```hashCode()```
* Polymorphism, such as:
```java
List<String> words = new ArrayList<>();
```

## Exception Handling

You should be familiar with:
* Common Java Exceptions and what they mean, like:
  * ```IndexOutOfBoundsException```
  * ```NullPointerException```
  * ```IllegalArgumentException```
  * ```ArithmeticException```
  * ```IllegalStateException```
  * ```ClassCastException```
* Java checked exceptions
  * ```FileNotFoundException```
  * ```IOException```
  * ```Exception```
* How to create your own Exceptions
* ```try```-```catch```-```finally``` blocks

## Example Code

Look at this [example Java code on Github](https://github.com/sde-coursepack/java-prerequisite/tree/main/src/main/java/example).
This code has 4 classes, which I encourage you to look at in this order:

1. Student.java
2. Enrollment.java
3. Main.java
4. StudentNameComparator.java

You should be able to understand all of this code *before* we dive
into our course material. This includes understanding what each
class does and how each class interacts with each other.

You should be able to download these
Java files, put them into a project in your IDE of choice, and run
the Main.java program, and you should be able to understand why
the output displays.

Note that, in your IDE, you'll have to add the code to a package called
"example". This can be done by right-clicking on your source code
folder (typically "src") and going to New -> Package. Then, simply
make a package called "example", and drop the downloaded files in there.
We will talk about packages, and why we have them, soon.

As we go further in the class, you'll learn how to "clone" the Github
Repository this code is in, as well as "build" it in order to run
the code as is.