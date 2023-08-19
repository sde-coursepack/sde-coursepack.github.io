---
Title: Interfaces
---

# Interfaces
{: .no_toc }

An interface is a description of behaviors and a way to access them. For example, the interface on a vending machine has a way to insert money, a way to select a product, and a way to receive the product and any change. That is, interfaces describe how a client (the person using the interface) interacts with a system that actually implements the behavior.

It's important to understand that just like user applications and physical tools have interfaces, our code also has interfaces. Every time you create a class with methods, you have created an interface (API - application programming interface) that "clients" (you or other programmers using that code) will interact with. In this module, we will explore Java `interface`s.

---

## Contents
{: .no_toc }

* TOC
{:toc}

---


## Java `interface`

The Java `interace` keyword is a way to describe an interface separate from the implementation of that interface.

For the sake of this textbook, we will often talk about an interface in two different ways.

* interface as in "application programming interface", or API. This is the publicly visible methods of a class that allow you to use the class
* `interface` as in `public interface ClassName` in Java, that is, the specific Java keyword `interface`

This article focuses on the latter. For clarity, whenever I am specifically talking about `interface`. For clarity, on this website, I only mean the Java syntactic definition of an `interface` when I used block letters (that is, "code-like" letters). 

The purpose of a Java `interface` is to define a class-like API (application programming interface) that is *separate* from the implementing class. The advantage of this will become clear as we talk about polymorphism, but focuses on allowing us to separate interface details from implementation details, in fact hiding them outright.

Below is an example of an `interface` for, say, a calendar of events.

```java
public interface EventCalendar {
    public Date getCurrentDay();
    public List<Event> getEventOnDay(Date day);
    public Event getNextEvent();
    public void addEvent(Event event);
}
```

Whereas an implementing class may look like:
```java
public class EventCalendarImpl implements EventCalendar {
    private List<Event> eventList;
    public EventCalendarImpl() {
        eventList = new ArrayList<>();
    }
    
    @Override
    public Date getCurrentDay() {
        LocalDateTime now = LocalDateTime.now();
        return new Date(now);
    }
    
    @Override
    public List<Event> getEventOnDay(Date day) {
        eventList.stream().filter(event -> event.getDay().equals(day)).toList();
    }
    ... //etc
    @Override
    public Event getNextEvent() {
        //implementation
    }
    
    @Override
    public void addEvent(Event event) {
        eventList.add(event);
    }
}
```

Note that you may think of other ways to implement this, or implement current time, etc. The point is, so long as our implementation provides for the features the `interface` describes, then it satisfies the `interface`. If, however, any functions are missing, or have different signatures, then our program will have a Java syntax error, and be unable to compile. This is actually a great thing! The idea here is to use Java's **syntax** to enforce intended **design**.

This means a junior programmer can't accidentally implement a class with an incorrect API, since the API is enforced by syntax!

## `interface` limitations

An `interface` is not like a class, and cannot have several features that a class has.

### No Constructors

We cannot directly instantiate an `interface`, since an interface just describes a set of features. As such, you cannot have any constructors.

### No fields

An `interface` cannot have any fields because it should not hold any data. Data relates to details about a particular implementation.

### No private or protected methods

Because an *interface* describes features, but not implementation details, it cannot have `private` methods. This is because all methods need to be implemented by a class that implements that interface.

### No implemented methods

Similarly, we cannot have any implementations of methods, since the entire goal of using `interface` is to separate the interface (general idea of an API) from the implementation (the class that actually implements the behavior).

### `default` functions

Be aware there are such a thing as default functions that you may occasionally see in some classes, but as a general rule you shouldn't use them unless you have a **very, very good reason**. If you are unsure what such a reason would be, you shouldn't use the default method (and likely may want to use an abstract class instead).

## Interface usage examples:

You have likely seen or used several interfaces already in previous study. Some interfaces to be familiar with:

### Comparator and Comparable

These interfaces are used in sorting. 

__Comparable__ - defines the "natural order" that an object is sorted by. For example, if we had a class Student that we wanted, by default, to sort by Student number in *ascending* order, we might have something like:

```java
public class Student implements Comparable<Student> {
    private int studentNumber;
    private String firstName, lastName;
    ...
    
    @Override
    public int compareTo(Student other) {
        return this.studentNumber - other.studentNumber;
    }
}
```

The Comparable interface works under the idea that the integer returned has three possibilities, negative, positive, or zero.

For instance, if we called `a.compareTo(b)`, then the result could be:

* **negative** - `a` would come *before* `b` when sorted
* **positive** - `a` would come *after* `b` when sorted
* **equal** - `a` and `b` are equal for the purposes of sorting, and can come in either order

__Comparator__ - serves the same overall purpose of Comparable. However, Comparator lets you define *additional* orders, beyond just the natural order, to sort by. For example, if we wanted to sort students by `lastName` and then `firstName` in alphabetical order, we could create the following `Comparator`:

```java
public class StudentNameComparator implements Comparator<Student> {
    public int compare(Student a, Student b) {
        aLastName = a.getLastName().toUpperCase();
        bLastName = b.getLastName().toUpperCase();
        if (aLastName.equals(bLastName)) {
            aFirstName = a.getFirstName().toUpperCase();
            bFirstName = b.getFirstName().toUpperCase();
            return aFirstName.compareTo(bFirstName);
        } else {
            return aLastName.compareTo(bLastName);
        }
    }
}
```

Note that in the above code I'm using `toUpperCase` to normalize all letters in the names, as by default the `String` compareTo method *is* case-sensitive, and all capital letters come before all uppercase letters.

To sort a list using the natural order, simply knowing that the class of objects we are sorting `implements Comparable` is sufficient.

```java
    List<Student> studentList = ... // create an populate student list
    Collections.sort(studentList);
```

If, however, we want to sort using a `Comparator` defined method, we can use:

```java
    List<Student> studentList = ... // create an populate student list
    Collections.sort(studentList, new StudentNameComparator());
```

The advantage of this approach is that we can utilize the fact that `Collections.sort` is already an implementation of sorting  (and specifically, of a Merge Sort, which is a very efficient sorting algorithm), and simply by adhering to an interface that `Collections.sort` accepts, we can define how two elements are compared based on our needs.

### Iterable

Iterators are a design pattern that describes iterating through a data structure in such a way that you visit each element exactly once. We can use iterators to, say, print every element of a List:

```java
    List<Student> studentList = ... // create an populate student list
    Iterator<Student> iterator = studentList.iterator();
    while(iterator.hasNext()) {
        Student nextStudent = iterator.next();
        System.out.println(nextStudent);
    }
```

The advantage of this approach, however, is that this *same* code would work if, for example, we used a TreeSet:

```java
    TreeSet<Student> studentTree = ... // create an populate student list
    Iterator<Student> iterator = studentTree.iterator();
    while(iterator.hasNext()) {
        Student nextStudent = iterator.next();
        System.out.println(nextStudent);
    }
```

That is, the second example would still visit every element exactly once, even though the structure of a TreeSet is different from that of a List. A HashSet would also be "iterable" in this way.

Now, if you wanted to print every element of a TreeSet, you could also use an enhanced for loop:

```java
    TreeSet<Student> studentTree = ... // create an populate student list
    for(Student student : studentTree) {
        System.out.println(student)
    }
```

So we don't need iterators, right? Wrong! Because the above code **implicitly** uses an iterator. **Implicitly**, in this context, means "without actually typing the code". The enhanced for-loop shown above uses iterators. This means that *any* data structure you create that you want to loop through and visit each element, you can simply have that class implement `Iterable` (which in turn involves creating an `Iterator` class), and then you can use any tools that work with iterators!

The `Iterable` interface has only one function that we care about for now (there is another `default` function called `forEach` that we will discuss with Java Streams later on):

* `Iterator<E> iterator()` - returns an Iterator that has not yet visited any elements of the data structure, but is capable of visiting all of them.

`Iterator<E>` itself is an interface that has two methods we care about:

* `boolean hasNext()` - returns `false` if the iterator has reached the end of the data structure, otherwise returns `true` if there are more elements to visit
* `E next()` - returns the next element in the collection. Be aware that each element can only be visited once this way. Additionally, `next()` throws a `NoSuchElementException` if `next()` is called when the iterator has already reached the end of the data structure.

### Runnable and Callable

`Runnable` and `Callable` are interfaces used for describing procedures, and are often used in `Threads` to allow for multiprocessing. Specifically, each interface has one methods:

`Runnable`
* `void run()` - Executes a procedure, returning nothing

`Callable<E>`
* `E call()` - Execute a procedure that returns an instance of class E. For instance, `Callable<Integer>` would return an Integer.

Here, this allows us to create classes that exist to perform some procedure, without having to actually implement multithreading ourselves. We can simply use the existing Java implementation of `Thread`s. This ensures that our multithreading is easy and consistent to implement, while still allowing us to do whatever we need to inside of the `Runnable` or `Callable` class.

## Implementing multiple interfaces

Be aware that a class may implement multiple interfaces. For example, a class may implement `Comparable` and `Serializable` (an interface that describes how we turn an object instance into a byte string, and then turn the byte string back into the object instance) if we want to both sort with it and use networking tools that rely on the Serializable interface. You would simply have:

```java
public class Student implements Comparable<Student>, Serializable {
    ...
}
```