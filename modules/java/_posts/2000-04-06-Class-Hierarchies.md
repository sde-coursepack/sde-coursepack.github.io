<style>
img[src*='#center'] {
    display: block;
    margin: auto;
}
</style>

---
Title: Class Hierarchies
---

# Class Hierarchies

## Definitions
In Java, class hierarchy refers to the organization of classes. All classes in Java are organized into a hierarchy, with the root of the hierarchy being the `Object` class. This means that every class in Java ultimately inherits from the `Object` class, and can access its methods. Java's class hierarchy is based on the idea of inheritance, which allows classes to _inherit_ properties and behaviors from other classes. Specifically, a class that inherits from another class is called a `subclass` (or derived class), while the class it inherits from is called the `superclass` (or base class). 
  
![At the top of the class hierarchy is the `Object` class](https://docs.oracle.com/javase/tutorial/figures/java/classes-object.gif#center)

 <p align="center">At the top of the class hierarchy is the `Object` class (Source: Oracle's Java Documentation)</p>

The Java class hierarchy provides a way for developers to organize their code and create reusable, modular software components. It also allows for polymorphism, which means that objects of different classes can be used interchangeably, as long as they share a common superclass. This feature makes it possible to write __clean code__ which is more flexible, scalable, and easier to maintain.

### is-a vs. has-a

Within the Java class hierarchy, there are important relationships that describe the associations between different classes: "is-a" and "has-a":

* __"Is-a"__ relationship is also known as inheritance or subclassing, and it means that one class is a specialized version of another class. For example, if we have a `Car` class and a `Jeep` class that inherits from the `Car` class, we can say that "Jeep is a Car." This relationship implies that the Jeep class has all the properties and methods of the Car class, as well as additional properties and methods specific to a jeep.

* __"Has-a"__ relationship, on the other hand, describes a composition or aggregation between classes, where one class contains an instance of another class as a member variable. For example, if we have a `Car` class and an `Engine` class, we can say that "Car has an Engine." This relationship implies that the `Car` class contains an instance of the `Engine` class as one of its member variables, and can use its methods to perform actions related to the engine.

Both relationships are important in object-oriented programming and can be used to create flexible, modular, and reusable code. 

### Superclass and Subclass

### extends

## Example: clocks

### Clock

### AlarmClock

## Code elements

### `public`, `protected`, `package-protected`, and `private`

### @Override

## `super` keyword

### `super()` in Constructors

### `super.` usage

