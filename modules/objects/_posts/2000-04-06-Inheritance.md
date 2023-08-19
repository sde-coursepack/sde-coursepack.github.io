---
Title: Class Hierarchies
---

# Inheritance
{: .no_toc }

In this module, we will discuss inheritance and class hierarchies. Specifically, we will discuss parent and child classes, how they are used in Java, and the uses cases of inheritance.

---

## Contents
{: .no_toc }

* TOC
{:toc}

---


## Authorship note
This article was partially authored by [Prof. Rich Nguyen](https://www.cs.virginia.edu/~nn4pj/) using materials for class written by Prof. Will McBurney. Will McBurney edited and supplemented this content. Prof. Rich Nguyen co-taught the initial offering of this course in Fall 2022, and taught it solely in Spring 2023.


## Definitions
In Java, class hierarchy describe the tree-like structures formed by inheritance. All classes in Java are organized into a hierarchy, with the root of the hierarchy being the `Object` class. This means that every class in Java ultimately inherits from the `Object` class, and can access its methods. Java's class hierarchy is based on the idea of inheritance, which allows classes to _inherit_ properties and behaviors from parent classes. Specifically, a class that inherits from another class is called a `subclass` (or derived class), while the class it inherits from is called the `superclass` (or base class).

![At the top of the class hierarchy is the `Object` class](https://docs.oracle.com/javase/tutorial/figures/java/classes-object.gif#center)

At the top of the class hierarchy is the `Object` class (Source: Oracle's Java Documentation)

The Java class hierarchy provides a way for developers to organize their code and create reusable, modular software components. It also allows for polymorphism, which means that objects of different classes can be used interchangeably, as long as they share a common superclass. This feature makes it possible to write __clean code__ which is more flexible, scalable, and easier to maintain.

## Inheritance vs. Aggregation

Within the Java class hierarchy, there are important relationships that describe the associations between different classes: Inheritance ("is-a") and Aggregation ("has-a"):

* __Inheritance ("Is-a"):__ relationship is also known as subclassing, and it means that one class is a specialized version of another class. For example, if we have a `Car` class and a `SUV` class that inherits from the `Car` class, we can say that "SUV is a Car." This relationship implies that the SUV class has all the properties and methods of the Car class, as well as additional properties and methods specific to a jeep.

* __Aggregation ("Has-a"):__ relationship, on the other hand, describes a composition between classes, where one class contains an instance of another class as a member variable. For example, if we have a `Car` class and an `Engine` class, we can say that "Car has an Engine." This relationship implies that the `Car` class contains an instance of the `Engine` class as one of its member variables, and can use its methods to perform actions related to the engine.

Both Inheritance and Aggregation relationships are important in object-oriented programming and can be used to create flexible, modular, and reusable code.

## Superclass and Subclass

When a new class is defined by adding onto an existing class, the new class is called the __subclass__ (derived class, or child class) and the existing class is called the __superclass__ (base class, or parent class). The subclass _inherits_ from the superclass (all methods and attributes) and the subclass _extends_ the superclass.

You can think of a superclass as a blueprint for a set of related classes. For example, you could have a superclass called `Car` that defines common properties such as "NumberOfWheels", "Color," "Fuel," and "Speed," as well as methods such as "start()," "accelerate()," and "brake()." Then, you could create subclasses such as `SUV,` `Truck,` that inherit these properties and methods from the `Car` superclass, but also have their own unique properties such as "loadCapacity," "NumberOfPassengers," or methods such as "loadPassenger()" and "loadContainer()."

Subclasses can override the methods of their superclass, meaning that they can provide a new implementation for a method defined in the superclass. For example, the `SUV` subclass could override the "start()" method to implement a specific way of starting that is different from the generic "start" method defined in the `Car` superclass.

Inheritance through superclasses is a powerful feature in Java that allows you to create a hierarchy of related classes and promote code reuse. By defining common properties and methods in a superclass, you can avoid duplicating code in your subclasses and make your code more modular and maintainable.

### extends
In Java, the `extends` keyword is used to create a subclass that inherits properties and methods from a superclass. The keyword is followed by the name of the superclass that the subclass is extending. The "extends" keyword is used in the class definition, like this:

```java
public class SUV extends Car {
    // subclass members
}
```
In this example, `SUV` is the name of the subclass, and `Car` is the name of the superclass that it is extending. It is important to note that a Java class can only extend *one* superclass at a time, but a superclass can have *multiple* subclasses. Also, the `extends` keyword is used for class inheritance, while the "implements" keyword is used for interface implementation.


## Code elements

### `public`, `protected`, `package-protected`, and `private`
There are four access modifiers that can be used to restrict access to classes, fields, and methods: `public`, `protected`, `package-protected` (also known as default), and `private`. Consider this `Car` class:

```java
package edu.virginia.cs.oo;

public class Car {
    public String make;
    protected String model;
    String color; // package-protected
    private int year;

    public Car(String make, String model, String color, int year) {
        this.make = make;
        this.model = model;
        this.color = color;
        this.year = year;
    }

    public void start() {
        System.out.println("Starting the " + make + " " + model);
    }

    protected void drive() {
        System.out.println("Driving the " + color + " " + make + " " + model);
    }

    void stop() {
        System.out.println("Stopping the " + color + " " + make + " " + model);
    }

    private void maintenance() {
        System.out.println("Performing maintenance on the " + year + " " + make + " " + model);
    }
}
```
In this example, we have a `Car` class with four different access modifiers used for its fields and methods:

* The `public` access modifier is used for the make field and the start method. This means that they can be accessed from anywhere, even outside the `Car` class or the `edu.virginia.cs.oo` package.

* The `protected` access modifier is used for the model field and the drive method. This means that they can only be accessed within the `Car` class, the `edu.virginia.cs.oo` package,  or any subclasses that inherit from it. Subclasses can access protected members even if they're in a different package.

* The default access modifier (i.e. no access modifier specified) is used for the color field and the stop method. This means that they can only be accessed within the same package as the `Car` class, `edu.virginia.cs.oo`. This is also known as "package-protected" access. Child classes outside the `edu.virginia.cs.oo` package **cannot** access package-protected elements.

* The `private` access modifier is used for the year field and the maintenance method. This means that they can only be accessed within the Car class itself. They cannot be accessed from any subclasses, even if they inherit from the Car class or are in the same package.

Here's an example usage of the Car class to demonstrate these access modifiers:

```java
package edu.virginia.cs.notsamepackage;

public class Main {
    public static void main(String[] args) {
        Car myCar = new Car("Ford", "Mustang", "blue", 2022);
        System.out.println(myCar.make); // Output: "Ford"
        // System.out.println(myCar.model); // Compilation error, model is protected
        // System.out.println(myCar.color); // Compilation error, color is package-protected
        // System.out.println(myCar.year); // Compilation error, year is private
        myCar.start(); // Output: "Starting the Ford Mustang"
        myCar.drive(); // Compilation error, drive is protected
        myCar.stop(); // Output: "Stopping the blue Ford Mustang"
        // myCar.maintenance(); // Compilation error, maintenance is private
    }
}


```
In this example, we create a `Car` object with the make "Ford", model "Mustang", color "blue", and year 2022. We then try to access each of the fields and methods of the Car class from the main method using the myCar object.


## `super` keyword
We can also use the super keyword to call superclass methods from within a subclass. For example, if the Car class had a method called drive(), we could override this method in the SUV class and call the superclass implementation using the super keyword:

```java
package edu.virginia.cs.oo;

public class SUV extends Car {
    private int numSeats;
    
    public SUV(String make, String model, String color, int year, int numSeats) {
        super(make, model, color, year);
        this.numSeats = numSeats;
    }

    @Override
    public void drive() {
        super.drive();
        System.out.println("And carrying " + numSeats + " passengers");
    }


    public String getMake() {
        return super.make;
    }

    public String getModel() {
        return super.model;
    }
}
```

### `super()` in Constructors

In this example, the `SUV` class has a constructor that takes three parameters: the make, model, and numSeats of the SUV. We want to initialize the make and model fields of the `Car` superclass, so we use the `super` keyword to call the superclass constructor with those parameters. We then initialize the numSeats field with the numSeats parameter.

Using the `super` keyword to call the superclass constructor is **syntactically necessary** in this case because the `Car` class does not have a zero-argument constructor. In fact, you must call the `super` constructor *in the very first line of the child class's constructor! For example, the following is syntactically invalid, even though it has nothing to do with fields of the parent class:

```java
public class SUV extends Car {
    private int numSeats;

    public SUV(String make, String model, String color, int year, int numSeats) {
        System.out.println("Hello World!"); //syntax error - this will not compile because super must come first!
        super(make, model, color, year);
        this.numSeats = numSeats;
    }
    ...
}
```

By calling the superclass constructor with the appropriate parameters, we can initialize these fields in the superclass and make them available for use in the subclass. Note that if the parent class has a zero-argument constructor (such as the `Object` class that all classes extend), an explicit call to `super` is not required. However, if you wish to invoke a constructor that does have arguments, you must call `super` **on the very first line of the constructor**.

### `super.` usage

The `SUV` class overrides the `drive()` method of the `Car` class and adds a message about the number of passengers the SUV is carrying. We call the superclass implementation of `drive()` using the `super.` keyword to avoid duplicating the output about the make and model of the car.

### @Override
We can use the @Override annotation in Java to indicate that a method in a subclass is intended to override a method in the parent class. Note that we use the `@Override` annotation on the start method in the SUV class to indicate that we intend to override the same-named method in the parent class. We can quickly demo the idea here:

```java
package edu.virginia.cs.oo;

public class Main {
    public static void main(String[] args) {
        SUV mySUV = new SUV("Jeep", "Wrangler", "black", 2023, 5);
        mySUV.drive(); // Outputs "Driving the black Jeep Wrangler" and "And carrying 5 passengers"
    }
}
```
The full implementation of both `Car` and `SUV` classes can be found in [this repo on the coursepack](https://github.com/sde-coursepack/Car).

Note that the `@Override` annotation is never required. However, it is a good idea to include it. This is because if the interface you are implementing or class you are extending changes, `@Override` of any methods whose signatures have changed will throw a syntax error. This helps alert any developers to the change, and ensures their code can be updated accordingly.


## Another Example: Clocks
Sometimes, we may find that two classes are very similar: `Clock` which tells us the time, and `AlarmClock` which tells us the time and also has an alarm. They are so similar, in fact, that we can implement `AlarmClock` as a sub-class (or child class) of `Clock`. This means `AlarmClock` _is-a_ `Clock` (Anything a clock can do, so can an alarm clock), but `Clock` _is-not-a_ `AlarmClock` (a clock may not has all the behaviors of an alarm clock).

### Clock
```java
public class Clock {
    protected String brandName;
    protected int currentHour, currentMinute, currentSecond;

    public Clock(String brandName) {
        this.brandName = brandName;
        update();
    }

    public Clock(String brandName, int currentHour, int currentMinute, int currentSecond) {
        this.brandName = brandName;
        this.currentHour = currentHour;
        this.currentMinute = currentMinute;
        this.currentSecond = currentSecond;
    }

    public void update() {
        LocalDateTime dateTime = LocalDateTime.now();
        DateTimeFormatter f = DateTimeFormatter.ofPattern("HH:mm:ss");
        String s = dateTime.format(f);
        currentHour = Integer.parseInt(s.substring(0, 2));
        currentMinute = Integer.parseInt(s.substring(3, 5));
        currentSecond = Integer.parseInt(s.substring(6, 8));
    }

    public int getCurrentHour() {
        return currentHour;
    }

    public int getCurrentMinute() {
        return currentMinute;
    }

    public int getCurrentSecond() {
        return currentSecond;
    }

    public void setCurrentHour(int currentHour) {
        this.currentHour = currentHour;
    }

    public void setCurrentMinute(int currentMinute) {
        this.currentMinute = currentMinute;
    }

    public void setCurrentSecond(int currentSecond) {
        this.currentSecond = currentSecond;
    }

    public String toString() {
        return brandName + " brand clock - " + getTimeAsString();
    }

    protected String getTimeAsString() {
        return currentHour + ":" + currentMinute + ":" + currentSecond;
    }
}
```

Here, this class is *almost* a simple data structure, with fields, getters, and setters. However, the `update` method is particularly noteworthy. Specifically, the `update` method is used to set the state of our `Clock` instance to be equal to the current local time.

```java
    public void update() {
        LocalDateTime dateTime = LocalDateTime.now();
        DateTimeFormatter f = DateTimeFormatter.ofPattern("HH:mm:ss");
        String s = dateTime.format(f);
        currentHour = Integer.parseInt(s.substring(0, 2));
        currentMinute = Integer.parseInt(s.substring(3, 5));
        currentSecond = Integer.parseInt(s.substring(6, 8));
    }
```

I also want to note that the fields are all `protected` rather than `private`; this will be relevant in the next section.

### AlarmClock

Below is the code for the class `AlarmClock` which is a sub-class of `Clock`. Specifically, the `AlarmClock` adds a feature where an alarm will go off if `update` is called during the alarm's time.

```java
public class AlarmClock extends Clock {
    private int alarmHour, alarmMinute;

    public AlarmClock(String brandName,
                      int alarmHour, int alarmMinute) {
        super(brandName);
        this.alarmHour = alarmHour;
        this.alarmMinute = alarmMinute;
    }

    public AlarmClock(String brandName, int alarmHour, int alarmMinute,
                      int currentHour, int currentMinute, int currentSecond) {
        super(brandName, currentHour, currentMinute, currentSecond);
        this.alarmHour = alarmHour;
        this.alarmMinute = alarmMinute;
    }


    public int getAlarmHour() {
        return alarmHour;
    }

    public int getAlarmMinute() {
        return alarmMinute;
    }

    public void setAlarmHour(int alarmHour) {
        this.alarmHour = alarmHour;
        checkAlarm();
    }

    public void setAlarmMinute(int alarmMinute) {
        this.alarmMinute = alarmMinute;
        checkAlarm();
    }

    @Override
    public void update() {
        super.update();
        checkAlarm();
    }

    @Override
    public String toString() {
        String s = brandName + " brand alarm clock " + getTimeAsString();
        return s + "\n\tAlarm: " + hour + ":" + minute + (isPM ? "p.m." : "a.m.");
    }

    private void checkAlarm() {
        if (currentHour == alarmHour && currentMinute == alarmMinute) {
            System.out.println("BEEEP BEEP BEEEP BEEP BEEP");
        }
    }
}
```

You may notice a couple things here. First, where are the fields for `brandName`, `currentHour`, `currentMinute`, and `currentSecond`? After all, we reference two of these values in the function `checkAlarm`, right?

```java
    private void checkAlarm() {
        if (currentHour == alarmHour && currentMinute == alarmMinute) {
            System.out.println("BEEEP BEEP BEEEP BEEP BEEP");
        }
    }
```

Also, what's with the `super.update()` call in the AlarmClock `update` function?

```java
    @Override
    public void update() {
        super.update();
        checkAlarm();
    }
```

The rest of this module will focus on answering these questions.

### Method Overriding

Let's look at some code that uses `Clock` and `AlarmClock`:

```java
    Clock clock = new Clock("Casio");
    AlarmClock alarmClock = new AlarmClock("Amazon Basics", 13, 35);
    
    System.out.println("clock -> " + clock);
    System.out.println("alarmClock -> " + alarmClock);
```

The following will print:

```text
    clock -> Casio brand clock - 1:36:8 p.m.
    alarmClock -> Amazon Basics brand alarm clock 1:36:8 p.m.
	    Alarm: 1:36p.m.
```

Why the different printing? Because the two classes have different `toString()` methods. 

__Clock.toString__
```java
    public String toString() {
		return brandName + " brand clock - " + getTimeAsString();
	}
```

__AlarmClock.toString__
```java
    @Override
	public String toString() {
		String s = brandName + " brand alarm clock " + getTimeAsString(); //don't need super because no overriding method
		return s + "\n\tAlarm: " + hour + ":" + minute + (isPM ? "p.m." : "a.m.");
	}
```

This is because `AlarmClock` is *overriding* the parent class (`Clock`) `toString` method. Overriding in this context, means replacing. That is `AlarmClock` *has it's own* `toString` method, so it doesn't need or consider the parents.

Note that the reason `AlarmClock` overrides `Clock`'s `toString` method is **not** because of the `@Override` tag. The `@Override` tag is there for stability and design purposes, but has no impact on the actual running code. The reason for the override is because `AlarmClock`'s `toString` method has the same method signature; that is, a method with the name `toString()` that takes in no arguments.

I'm going to change one subtle detail of the code above. Specifically, I'm changing the compile-time type of the variable `alarmClock` to `Clock`. The constructor, however, is still `AlarmClock`. Will this change what prints?

```java
    Clock clock = new Clock("Casio");
    Clock alarmClock = new AlarmClock("Amazon Basics", 13, 35);
    
    System.out.println("clock -> " + clock);
    System.out.println("alarmClock -> " + alarmClock);
```

The answer is no! While, at compile time, the variable `alarmClock` is treated as a `Clock`, at runtime, it is still an `AlarmClock`, so when `toString` is called, Java still invokes the `AlarmClock` class's `toString` method. This is an example of using `polymorphism`.

### `super` in the Constructor

As mentioned before, when extending a parent class that does not have a zero-argument constructor (like `Clock`), every child must explicitly invoke the parent's compiler. This is done using `super` with the arguments you wish to pass to the parent constructor. This is similar to how we can use `this` to "pass argument" to a separate constructor, like we showed in the `PezDispenser` example in the "OO-Refresher" module.

For example, in the first constructor of `AlarmClock`

```java
    public AlarmClock(String brandName, int alarmHour, int alarmMinute) {
		super(brandName); 
		this.alarmHour = alarmHour;
		this.alarmMinute = alarmMinute;
	}
```

Here, we are invoking the constructor `Clock(String brandName)`, since that matches our argument of a single String. On the other hand, the other `AlarmClock` constructor invokes the other `Clock` constructor.

### `super` in other methods

Consider the `AlarmClock update()` method. The idea of this method is, in addition to updating the time, I want to check the alarm. This means I want to override the parent `update` method in `Clock`, but I still want to utilize its code for updating the current time fields. I just *also* want to check the alarm.

```java
    @Override
    public void update() {
        super.update();
        checkAlarm();
    }
```

In this context, `super.update()` means "call the parent class's update method". Technically, you could use this to call *any* parent method, but you only *need* to use `super` when you want to invoke a parent method you have overridden.


### Refactoring to remove `protected` fields

Now, you may think it's necessary to leave the fields as `protected` so that the child can access them. But, in this case, you'll notice that we have getters and setters already for these fields in the parent class, `Clock`. As such, we can actually make the fields private, and just have the child class use those getters and setters when needed.

For example, we can update `AlarmClock`'s `checkAlarm` method to be:

```java
    private void checkAlarm() {
		if (getHour() == alarmHour && getMinute()== alarmMinute) {
			System.out.println("BEEEP BEEP BEEEP BEEP BEEP");
		}
	}
```

That is, we use the getters `getHour()` and `getMinute()` rather than the fields `currentHour` and `currentMinute`.

The advantage of this approach is that I am not interacting with fields of the parent, but rather the parent's shared *behavior* of its interface. This is beneficial here in the same way it's beneficial to generally use private fields and public methods. However, there may be instances where you *do* what the child directly manipulating fields in the parent class because those interactions are complicated enough to warrant that access. In that case, `protected` is fine. But when feasible, fields should remain `private`, even to the class's children.