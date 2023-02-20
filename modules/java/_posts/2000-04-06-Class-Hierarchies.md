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

### Inheritance vs. Aggregation

Within the Java class hierarchy, there are important relationships that describe the associations between different classes: Inheritance ("is-a") and Aggregation ("has-a"):

* __Inheritance ("Is-a"):__ relationship is also known as subclassing, and it means that one class is a specialized version of another class. For example, if we have a `Car` class and a `Jeep` class that inherits from the `Car` class, we can say that "Jeep is a Car." This relationship implies that the Jeep class has all the properties and methods of the Car class, as well as additional properties and methods specific to a jeep.

* __Aggregation ("Has-a"):__ relationship, on the other hand, describes a composition between classes, where one class contains an instance of another class as a member variable. For example, if we have a `Car` class and an `Engine` class, we can say that "Car has an Engine." This relationship implies that the `Car` class contains an instance of the `Engine` class as one of its member variables, and can use its methods to perform actions related to the engine.

Both Inheritance and Aggregation relationships are important in object-oriented programming and can be used to create flexible, modular, and reusable code. 

### Superclass and Subclass

When a new class is defined by adding onto an existing class, the new class is called the __subclass__ (derived class, or child class) and the existing class is called the __superclass__ (base class, or parent class). The subclass _inherits_ from the superclass (all methods and attributes) and the subclass _extends_ the superclass.

You can think of a superclass as a blueprint for a set of related classes. For example, you could have a superclass called `Vehicle` that defines common properties such as "NumberOfWheels", "Color," "Fuel," and "Speed," as well as methods such as "start()," "accelerate()," and "brake()." Then, you could create subclasses such as `PassengerVehicle,` `TransportationVehicle,` that inherit these properties and methods from the `Vehicle` superclass, but also have their own unique properties such as "loadCapacity," "NumberOfPassengers," or methods such as "loadPassenger()" and "loadContainer()."

Subclasses can override the methods of their superclass, meaning that they can provide a new implementation for a method defined in the superclass. For example, the `PassengerVehicle` subclass could override the "start()" method to implement a specific way of starting that is different from the generic "start" method defined in the `Vehicle` superclass.

Inheritance through superclasses is a powerful feature in Java that allows you to create a hierarchy of related classes and promote code reuse. By defining common properties and methods in a superclass, you can avoid duplicating code in your subclasses and make your code more modular and maintainable.

### extends
In Java, the `extends` keyword is used to create a subclass that inherits properties and methods from a superclass. The keyword is followed by the name of the superclass that the subclass is extending. The "extends" keyword is used in the class definition, like this:

```java
public class Subclass extends Superclass {
    // subclass members
}
```
In this example, "Subclass" is the name of the subclass, and "Superclass" is the name of the superclass that it is extending. It is important to note that a Java class can only extend *one* superclass at a time, but a superclass can have *multiple* subclasses. Also, the `extends` keyword is used for class inheritance, while the "implements" keyword is used for interface implementation.

## Example: clocks
Sometimes, we may find that two classes are very similar: `Clock` which tells us the time, and `AlarmClock` which tells us the time and has an alarm. They are so similar, in fact, `AlarmClock` is a subclass of `Clock`. This means `AlarmClock` _is-a_ `Clock` (Anything a clock can do, so can an alarm clock), but `Clock` _is-not-a_ `AlarmClock` (a clock may not has all the behaviors of an alarm clock).

### Clock
```java
public class Clock {
   protected String brandName;
   protected int currentHour, currentMinute, currentSecond;
   
   public Clock(String brandName) {
      this.brandName = brandName;
      update();
   }
}
```

### AlarmClock
```java
public class AlarmClock extends Clock {
   private int alarmHour, alarmMinute;

   public AlarmClock(String brandName, 
                 int alarmHour, int alarmMinute) {
      super(brandName);
      this.alarmHour = alarmHour;
      this.alarmMinute = alarmMinute;
   }
}
```
The full implementation of both `Clock` and `AlarmClock` classes can be found in [this repo on the coursepack](https://github.com/sde-coursepack/Clocks/tree/master).

## Code elements

### `public`, `protected`, `package-protected`, and `private`
There are four access modifiers that can be used to restrict access to classes, fields, and methods: `public`, `protected`, `package-protected` (also known as default), and `private`.

* `Public`: The "public" access modifier is the least restrictive and allows access to a class, field, or method from any other class, regardless of its package. This means that public members are visible and accessible to all classes in any package. Public members are typically used for methods or variables that need to be accessed by external classes.

* `Protected`: The "protected" access modifier allows access to a class, field, or method from the same class, any subclass, or any class in the same package. This means that protected members are visible and accessible to classes within the same package, as well as subclasses in any package. Protected members are typically used for methods or variables that need to be accessed by subclasses but not by external classes.

* `Package-Protected` (default): The "package-protected" access modifier (also known as default access) allows access to a class, field, or method from the same package. This means that package-protected members are visible and accessible to classes within the same package but not to classes in other packages. Package-protected members are typically used for methods or variables that need to be accessed by other classes in the same package.

* `Private`: The "private" access modifier is the most restrictive and allows access to a class, field, or method only from the same class. This means that private members are not visible or accessible to any other class, including subclasses in the same package. Private members are typically used for methods or variables that should not be accessed by any other class.

The choice of access modifier depends on the level of access control you want to enforce for a class, field, or method. Public members are accessible to all classes, protected members are accessible to subclasses and classes in the same package, package-protected members are accessible only within the same package, and private members are accessible only within the same class.

```java
package edu.virginia.cs.oo;

public class MyClass {
    public int publicVar = 1;
    protected int protectedVar = 2;
    int defaultVar = 3; // package-protected
    private int privateVar = 4;
    
    public void publicMethod() {
        System.out.println("This is a public method");
    }
    
    protected void protectedMethod() {
        System.out.println("This is a protected method");
    }
    
    void defaultMethod() { // package-protected
        System.out.println("This is a package-protected method");
    }
    
    private void privateMethod() {
        System.out.println("This is a private method");
    }
}
```
In this example, we have a class called "MyClass" with four different access modifiers for its fields and methods. To use these members from another class, we would need to create an instance of MyClass and call the appropriate methods or access the fields:
```java
package edu.virginia.cs.oo;

public class Main {
    public static void main(String[] args) {
        MyClass myObj = new MyClass();
        
        System.out.println(myObj.publicVar); // Outputs 1
        myObj.publicMethod(); // Outputs "This is a public method"
        
        // The following lines will not compile, as they attempt to access non-public members
        // System.out.println(myObj.protectedVar);
        // myObj.protectedMethod();
        // System.out.println(myObj.defaultVar);
        // myObj.defaultMethod();
        // System.out.println(myObj.privateVar);
        // myObj.privateMethod();
    }
}
```
In this example, we can access the "publicVar" field and "publicMethod" method of MyClass from the Main class, as they are marked as public. However, we cannot access the other members of MyClass, as they are marked as protected, package-protected, or private, and are not visible from the Main class.


### @Override

## `super` keyword

### `super()` in Constructors

### `super.` usage

