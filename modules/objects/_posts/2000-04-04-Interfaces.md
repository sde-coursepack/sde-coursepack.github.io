---
Title: Interfaces
---

* TOC
{:toc}


# Java `interface`

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

Note that you may think of other ways to implement this, or implement current time, etc. The point is, so long as our implementation provides for the features the `interface` describes, then it satisfies the `interface`. If, however, any functions are missing, or have different signatures, then Java will note a syntax error.

The idea here is to use Java's **syntax** to enforce intended **design**.

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

You have likely used several interfaces already in previous study. For example:

### Comparator

### Comparable

### Iterable

### Cloneable