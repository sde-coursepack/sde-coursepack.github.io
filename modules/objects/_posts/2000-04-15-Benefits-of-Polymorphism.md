---
Title: Class Hierarchies
---

# Benefits of Polymorphism

There are a number of design benefits to Polymorphism.

## Separate Interface and Implementation

Polymorphism allows us to separate the **interface** from the **implementation**, grouping classes with the same **interface** together, and ensuring our code is flexible and reusable.

### Interface

Remember, the interface is the description of the **behaviors** of a class. This goes a bit deeper than the Java `interface` keyword. For clarity, whenever I use the `interface` keyword, I will use the "computer text" format. Whenever I mean the abstract idea of an **interface**, I will use bold font.

For example, the `Comparator<E>` `interface` is a Java `interface`. That `interface` has one method:

* `int compare(E first, E second)`

To use this `interface` when writing our own class, we use:

```public class MyStringComparator implements Comparator<String>```

and write our own implementation of the method `int compare(E first, E second)`

However, the **interface** (the concept, not the keyword) is a bit more detailed. We don't want just **any** method called `compare` that has a function that takes in two of some datatype and return an int to implement `Comparator`

We want a method `compare` that tells us *how two elements should be sorted*. That is, the **interface** is both the syntax (`interface` and `implements`) and the **intended abstract behavior**.

In this way, an **interface** can be either an `interface` or an `abstract class`. In general, we should use some *abstraction* for every major behavior in our software system. That is, we hide implementation details behind an easy-to-understand interface that clearly communicated *intended abstract behavior*

### Implementation

At some point, we have to actually address the details. Abstract descriptions of behavior are the best starting point, but eventually somebody is going to start writing code. And that code has to actually carry out producing the intended behavior.

The key, however, is that all implementation details should be **hidden** from the **interface** (whether it is an `interface`, `class`, or `abstract class`). That is, any information that isn't strictly included in the **interface** should only exist inside the implementation.

However, with a good use of polymorphism, we can hide implementation details from *most* of our software system, instead utilizing only abstract **interface**. We will show what that can look like in the following design example.

### Example: Apportionment

When designing our software, we want to use polymorphism to our advantage where it makes sense. To consider how to do that, let's look at an example:

You are tasked with writing a program that apportions representatives to states for the US Congress based upon each state's population. That is, larger states can more representatives than smaller states. Your program needs to print out how many representatives each state gets.


Specifically, you are told:
* You will be given a file, such as .csv file, containing the name of each State and its total population
* You will then use a particular algorithm, such as The Hamilton/Vinton method)
* You need to display the states in some order, such as alphabetical order

From here, I see three very clear **abstract** behaviors:

* `StateSupplier` - Get a list of State objects from some resource.
* `ApportionmentMethod` - Take in a List of State objects (with name and population) and a number of representatives to Apportion. Return an Apportionment (mapping of states to a number of representatives)
* `ApportionmentFormat` - Take in an Apportionment and return a String of the desired output format.

By focusing on the **abstractions** first, we can find it will give us flexibility.

### StateSupplier

Consider the following abstract class:

```java
public abstract class StateSupplier {
    public abstract List<State> getStates();
}
```

This class describes a behavior: get me states! Where? That doesn't matter to describing the abstract behavior. Where the data comes from is an **implementation** detail.

From there, we can **implement** the behavior with something like:

```java
public class CSVStateSupplier extends StateSupplier {
    private String filename;
    
    public CSVStateSupplier(String filename) { this.filename = filename; }
    
    @Override
    public List<State> getStates() {
        FileReader fileReader = new FileReader(filename);
        ...//implementation goes here
    }
}
```

In our client class, we would use the abstract interface, with the exception of the constructor.

```java
    public void executeApportionment() {
        StateSupplier stateSupplier = getStateSupplier();
        List<State> = stateSupplier.getStates();
        ...
    }

    public StateSupplier getStateSupplier() {
        return new CSVStateSupplier("censusData.csv");
    } 
```

Remember, whenever possible, we want to use the most abstract type we can. So everywhere we can, we use `StateSupplier` as the data type, and **not** `CSVStateSupplier`.

#### What if things change?

What if our data source changes? For example, what if the Department of Commerce, who runs the Census, instead gives us a web-service that returns the most recent census. Now we have the advantage where our system can work without us manually copying and pasting files!

Does this mean we have to change `StateSupplier` or `CSVReader`? No! We can simply add a new **concrete** implementation of `StateSupplier`

```java
public class DeptOfCommerceStateSupplier {
    private final String COMMERCE_CENSUS_SERVICE_URL = "..." // link to census web-service
    
    @Override 
    public List<State> getStates() {
        URL webserviceURL = new URL(COMMERCE_CENSUS_SERVICE_URL);
        ...//implementation
    }
}
```

Now, we simply **replace the constructor call** in our client method:

```java
    public void executeApportionment() {
        StateSupplier stateSupplier = getStateSupplier();
        List<State> = stateSupplier.getStates();
        ...
    }

    private StateSupplier getStateSupplier() {
        return new DeptOfCommerceStateSupplier();
    }
```

That's it! One line of code change! And with no giant "If-statement of doom"! Everything else is just adding new code! This is very powerful, as it is much easier to add code than to change code.

And this was enabled by our setup with polymorphism!

### ApportionmentMethod

Now that we have the idea, let's apply it to the other **abstractions** we came up with:

```java
public abstract class ApportionmentMethod {
    public abstract Apportionment getApportionment(List<State> stateList, int representatives);
}
```

Then we can implement the Hamilton/Vinton method, which we'll shorten to `HamiltonApportionmentMethod`:

```java
public class HamiltonApportionmentMethod extends ApportionmentMethod {
    public Apportionment getApportionment(List<State> stateList, int representatives) {
        Apportionment apportionment = new Apportionment();
        int totalPopulation = getTotalPopulation(stateList);
        ...//implementation continues
    }
}
```

Now, going to our `executeApportionment` function

```java
    public void executeApportionment() {
        StateSupplier stateSupplier = getStateSupplier();
        List<State> = stateSupplier.getStates();
        ApportionmentMethod method = getApportionmentMethod();
        Apportionment apportionment = method.getApportionment();
    }

    private ApportionmentMethod getApportionmentMethod() {
        return new HamiltonApportionmentMethod();
    }
```

We want to change the method to Huntington-Hill (the current method used by the US Congress, which was adopted in 1929)?

```java
public class HuntingtonHillApportionmentMethod extends ApportionmentMethod {
    public Apportionment getApportionment(List<State> stateList, int representatives) {
        Apportionment apportionment = new Apportionment();
        allocateOneRepresentativeEach(apportionment, stateList);
        Map<State, Double> priorityMap = getPriorityMap(stateList);
        ...//implementation continues
    }
}
```

...and then simply change one line again:

```java
    public void executeApportionment() {
        StateSupplier stateSupplier = getStateSupplier();
        List<State> stateList = stateSupplier.getStates();
        ApportionmentMethod method = getApportionmentMethod();
        Apportionment apportionment = method.getApportionment(stateList, 435);
    }

    private ApportionmentMethod getApportionmentMethod() {
        return new HuntingtonHillApportionmentMethod();
    }
```

### ApportionmentFormat

You're starting to get it now, right? Let's define the abstract behavior:

```java
public abstract class ApportionmentFormat{
    public abstract String getString(Apportionment Apportionment);
}
```

...and then the implementation...

```java
public class AlphabeticalApportionmentFormat {
    public String getString(Apportionment Apportionment) {
        StringBuilder sb = new StringBuilder();
        for (State state : Apportionment.getStates())
        ...//implementation
            
    }
}
```

...and then our `execute method`

```java
    public void executeApportionment() {
        StateSupplier stateSupplier = getStateSupplier();
        List<State> stateList = stateSupplier.getStates();
        ApportionmentMethod method = getApportionmentMethod();
        Apportionment apportionment = method.getApportionment(stateList, 435);
        ApportionmentFormat format = getApportionmentFormat();
        System.out.println(format.getString(Apportionment));
    }

    private ApportionmentFormat getApportionmentFormat() {
        return new AlphabeticalApportionmentFormat();
    }
```



### What if we need to change at runtime?

What if we want to make our program respond dynamically? For example, depending on user-input at runtime, maybe we want to use *either* Hamilton or Huntington-Hill?

Well, now we simply update our `getApportionmentMethod()`

```java
    private ApportionmentMethod getApportionmentMethod() {
        if (isHamiltonApportionment()) {
            return new HamiltonApportionmentMethod();
        } else {
            return new HuntingtonHillApportionmentMethod();
        }
    }
```

Yes, this is one method returning two different data types. But because we are using the **abstract** data type `ApportionmentMethod`, this is completely fine!

## Reduced coupling

Notice a few things about our apportionment example?

* We can combine *any* StateSupplier with *any* ApportionmentMethod
* We can combine *any* combination of those two with *any* ApportionmentFormat
* We managed this flexibility without any subclass having to interact directly with *any other* subclass. The author of `HamiltonApportionmentMethod` doesn't need to know anything about `StateSupplier` or any of its subclasses, nor know anything about `ApportionmentFormat` or any of its subclasses.

The only *shared* classes are data structures: `State` (`String name`, `int population`) and `Apportionment` (effectively wrapper for a `Map<State,Integer>`). However, none of the subclasses have to worry about where there input is coming from, as that is handled by the interface.

## Dependency

Something interesting happens here because of our use of Polymorphism.

Consider what would happen if we wrote this entire class in one big main-class. Instead of writing a `class` for each of our `subclasses`, we instead wrote a function.

When we call a function, there are two things that happen:

* **We pass the control to the function:** - that is, the calling function stops executing and waits until the called function stops executing before the calling function
* **We have to call a specific function:** - we have to explicitly state what function we are calling. This means our calling function is "tied-to" our called function.

This means changes to implementation details in low level functions can trickle upward all the way to our high-level functions. This means that the more our software grows, the harder it becomes to maintain, because high-level functions are dependent on every other part of the system.

For example, if `executeApportionment()` wants to call a `getStates()` function, it has to say **exactly which function it will call**. This means the `getStates` function can only have 1 implementation. Now, you could put it an if-statement that calls one implementation or another, but how is the if-statement getting its information about which subfunction to call? Higher level functions will have to **know** and **understand** the inner workings of that if-statement in order to setup the correct configuration. Now our code is a tightly coupled mess.

### Dependency Inversion

Okay, let's leave that nightmare and come back to the flexible world of polymorphism we build in our Apportionment example.

It's important to understand that `executeApportionment` never picks what method to call! In fact, there are ways where to can write the class that executeApportionment is in where it **never has to know our subclasses even exist!**. The consequences of how this affects software construction are significant.

First, let's take a look at my implementation?

### My implementation

Here is my implementation of this class when I programmed the Apportionment Program

```java
public class Apportioner {
    private final StateSupplier supplier;
    private final ApportionmentMethod method;
    private final int representatives;
    private final ApportionmentFormat format;

    public Apportioner(Configuration configuration) {
        supplier = configuration.getStateSupplier();
        method = configuration.getApportionmentMethod();
        representatives = configuration.getRepresentatives();
        format = configuration.getApportionmentFormat();
    }

    public void executeApportionment() {
        List<State> stateList = supplier.getStates();
        Apportionment apportionment = method.getApportionment(stateList, representatives);
        System.out.println(format.getString(Apportionment));
    }
}
```

You'll notice there's not a single place in this class where I have to make decisions about which subclass to use. That's because I make those decisions in a class called `Arguments` which handles command-line arguments from the user to setup the programs `Configuration`. `Configuration` is simply a class that stores what "options" that the `Arguments` selected.

```java
public class Configuration {
    private StateSupplier supplier;
    private ApportionmentMethod method;
    private int representatives;
    private ApportionmentFormat format;

    public ApportionmentMethod getApportionmentMethod() {
        return method;
    }

    public void setApportionmentMethod(ApportionmentMethod method) {
        this.method = method;
    }

    public StateSupplier getStateSupplier() {
        this.supplier;
    }

    public void setStateSupplier(StateSupplier supplier) {
        this.supplier = supplier;
    }

    public int getRepresentatives() {
        return representatives;
    }

    public void setRepresentatives(int representatives) {
        this.representatives = representatives;
    }

    public ApportionmentFormat getApportionmentFormat() {
        return format;
    }

    public void setApportionmentFormat(ApportionmentFormat format) {
        this.format = format;
    }
}

```

If you look, you'll notice "Hey, this is just a POJO" (Plain Old Java Object). This is effectively a data structure, fundamentally no more complicated than our State class. This is just some fields, getters, and setters!

#### Back to dependency inversion

Now, you'll notice that we don't have the same situation as we did in our main function. Instead:

* **We pass the control to the function:** - that is, the calling function stops executing and waits until the called function stops executing before the calling function. **This part is unchanged.**
* **We call a function indirectly** - **Here's the change!** We are no longer calling a specific function! The author of `Apportioner.executeApportionment` does not have to know what function to call in what class. Which function is called when we call `getStates()`? `DeptOfCommerceStateSupplier.getStates()`? `CSVStateSupplier.getStates()`? Some new `StateSupplier` I just wrote? The author of `executeApportionment` doesn't know, and they don't need to!

That's because `executeApportionment` *is not dependent* on `CSVStateSupplier.getStates()` or any other subclass `getState`.

It is *only* dependent on the **interface** `StateSupplier`, and it's **intended abstract behavior**. This means that the **implementation** of any subclass of `StateSupplier` can change, or new **implementations** introduced, and `Apportioner`'s author won't have to care!

Instead of high-level methods depending on low-level methods, we have high-level methods and low-level methods **independently** depending on an **interface**. As such, any implementation of `StateSupplier` can be implemented **independently** of `Apportioner` without necessarily needing to affect Apportioner, so long as the **interface** is maintained.

## Conclusion

I know these ideas are, if you'll forgive the pun, abstract, and they made be hard to tangle with. The key insight, however, is when you are thinking about your software, think about major actions not as *functions*, but instead as *abstract interfaces*. Start with the abstract description of the behavior you wish to implement *first*, and implement specifics *second*.