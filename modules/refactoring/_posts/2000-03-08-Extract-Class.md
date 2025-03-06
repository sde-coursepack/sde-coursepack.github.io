---
Title: Refactoring - Extracting a Class
---

# Extract a Class

In this module, we will discuss the refactoring of extracting a class. We may extract a class for the following reasons:

* A single class contains unrelated information, so we separate into two or more classes so that each class is more **cohesive**.
* Two classes share enough in common that they should be combined into a single hierarchy, so we extract a parent-class to reduce code repetition
* A function is complicated and requires a large number of local variables, meaning it may really be a class in disguise.

* TOC
{:toc}



## Single Responsibility Principle

At core here is the **Single Responsibility Principle** (SRP). Coined by __Clean Code__ author Bob Martin, the SRP means that "a class should have only one reason to change." For example, let's imagine we were writing a program that apportioned representatives to Congress based upon state population. This program:

- Reads in a list of states and populations from a file
- Apportions representatives to those states based on some algorithm
- Prints the states in some order (say alphabetical)

You may think, "Okay this seems straightforward: I'll write three functions in main, one for each of these three parts. Then my main function will simply call those three functions."

This is a reasonable approach for ad hoc software. But consider this: what if you were actually writing this software for Congress to use every 10 years when they reapportion the states? Now, you have to consider a number of complications:

1. What if Congress wants to use different sources of data?
   * Maybe the first time they use a .csv file
   * Later on, they want to use Excel
   * Or, to streamline the process, they want you to get the data directly from the Department of Commerce (who runs the Census) via a web-API
2. What if the algorithm changes?
   * It's changed many times!
   * George Washington's very first veto was about Apportionment 
   * Thomas Jefferson proposed an algorithm used for much of the early history 
   * It was replaced by Hamilton's algorithm 
   * That was replaced by Jefferson's algorithm again 
   * That was replaced in 1929 by our current algorithm, Huntington-Hill
3. What if they want results in different ways?
   * Printed alphabetically? In descending order by representatives? 
   * Uploaded to a server or database? 
   * Generate a PDF printout and automatically email it to several people?

All of these are reasonable changes that could occur. And that's without even adding new features! (Hey! Could you also use this software to calculate the electoral college for Presidential elections?)

That's a *lot* of reasons that the software can change. As such, we want to break each of these "functions" up into classes that handle different **behavior**.

## Apportionment classes

When defining a class, we want to focus on two things first:
1. What is the purpose of the class? 
2. What is its **interface**?

It's important to think about the interface first, as we want to design the simplest interface we can that meets the needs of the **client**. A **client** is any other part of our software system that interacts with this interface (not to be confused with **customer**, the human users of our system). 

Note that when we say **interface**, we do not strictly mean a Java `interface`. In general, we use the term to mean "the `public` methods that we interact with when using the class."

For example in the class below:

```java
    public class StateReader {
        public StateReader(string filename) {
            ...
        }
        public List<State> getStates() {
            ...
        }
    }
```

The class `StateReader`'s interface is:
- Get a StateReader for a particular file by filename
- Get the list of states from that file

Note that we explicitly do not include `private` fields and methods in the interface. Private fields and methods relate to **implementation**, not **behavior**, and we want our interface to be independent of the implementation.

By thinking about the interface first, we can already envision how this class will be used by the client **before** we've even implemented the class:

```java
    StateReader stateReader = new StateReader("censusData.csv");
    List<State> stateList = stateReader.getStates();
```

This is not dissimilar to how we think of functions. However, the key distinction between the class StateReader and a function that performs the same behavior is **encapsulation**.

### Encapsulation

The entire goal of classes is to hide implementation details. If, for example, our file structure changes or we need to use a completely different file format, we necessarily have to change our **implementation**. However, our interface **does not need to change** so long as we are a) taking in data from some file and b) returning a List of states. 

This means that we have made it so changes to the implementation of this behavior only require changes to StateReader.java, **and** the only reason we would change StateReader.java is to change the implementation of reading states.

So by extracting one behavior to a separate class, we have made Main smaller and less complicated, as well as clearly communicated where in the system the "getting states from a data source" behavior is located.

## Simplifying data access

Let's imagine we also extract our "apportion representatives" behavior to a class:

```java
public class ApportionmentStrategy {
    public Map<State, Integer> getRepresentation (List<State> stateList, int totalRepresentatives)
}
```

This seems straight forward: "You give me a list of states (with name and population) and a target number of representatives, and I'll give you a mapping of each state to how many representatives they have."

But think about this: how many functions do we **need** for our `Map` here? Well, we would probably want:
* get the number of representatives for a given state
* add some number of representatives to a particular state
* give me all the states in the apportionment

Well, the `Map<State, Integer>` interface supports that to some extent:

* `get(State state)` will give us the number of representatives ... or null if our State has none
* `put(State state, int representatives)` will set the number of representatives for `state`, but it will overwrite the previous number instead of adding. If the state already has reps and we want to add more reps, we'd have to do something like:
`apportionmentMap.put(ohio, apportionmentMap,get(ohio) + representatives)`, and we would also need to use `containsKey(State state)` to check if we simply use `put` or we have to use `get-put`
* keySet() will give me a `Set` with the States in stateList, but I'd have to convert it to a list if I wanted to sort it...

This interface is *close*, but not exactly what we want. Further, [there are tons of methods in `Map` that we don't need](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html):
* clear()
* compute(K key, BiFunction<? super V, ? extends V> remappingFunction)
* entrySet()
* equals(Object o)
* hashCode()
* merge(K key, V value, BiFunction<? super V, ? extends V> remappingFunction)
* remove(Object key)

...etc. We won't use any of these methods intentionally.

In short, the *Map<State, Integer>* isn't exactly what we want. What we want, is something like:

```java
public class Representation {
    public void addRepresentativesToState(State state, int representatives);
    public int getRepresentativesFromState(State state);
    public List<String> getStateList();
}
```

By *encapsulating* our `Map` inside of this class, we are able to give it precisely the interface we want.

### Implementing Apportionment
```
public class Representation {
   Map<State, Integer> apportionmentMap = new HashMap<>();
   
   public void addRepresentativesToState(State state, int newRepresentatives) {
       int currentRepresentatives = getRepresentativesForState(state);
       apportionmentMap.put(state, currentRepresentatives + newRepresentatives);
   }

   public int getRepresentativesForState(State state) {
       return apportionmentMap.getOrDefault(state, 0);
   }

   public List<State> getStateList() {
       return new ArrayList<>(apportionmentMap.keySet());
   }
    
}
```

If you think about it, when interacting with the previous `Map`, we would have had to write much of this code anyways. Now, we extract it in its own class. This supports the Single Responsibility Principle, because now the class only changes if we need to change how apportionment is implemented and described at the data level. If we want to use a TreeMap for ease of default sorting, we can make that change inside of `Representation`, and no other class is affected or needs to change.


## Extracting an interface


Imagine now that we have two different classes that implement the same **behavior** in different ways:

```java 
public class HamiltonMethod {
   public Representation getHamiltonApportionment(List<State> stateList, int representatives) {
      ...
   }
}
```

```java
public class HuntingtonHillMethod {
    public Representation getHuntingtonHillApportionment(List<State> states, int reps) {
       ...
    }
}
```

Now, the function names do not match, but both of these classes still exist to support the same **behavior**. As such, we want to *standardize* their interface. This is where we can extract an **interface** that describes the *behavior* with a *standardized interface*.

```java
public interface ApportionmentMethod {
    Representation getApportionment(List<State> stateList, int representatives);
}
```

Then we change our other two classes to adhere to this interface **syntactically**.

```java
public class HamiltonMethod implements ApportionmentMethod {
   public Representation getApportionment(List<State> stateList, int representatives) {
      ...
   }
}
```

```java
public class HuntingtonHillMethod implements ApportionmentMethod {
   public Representation getApportionment(List<State> stateList, int representatives) {
      ...
   }
}
```

In this case, we are changing each class' method name to adhere to the interface, but we do not need to change the implementation. This has the added benefit elsewhere in the program. For example, a *client* class can now simply use an instance of the abstract *ApportionmentMethod* instead of having to juggle two different classes and figure out which to use.

For instance, let's say that we decided by default to use Huntington-Hill, but the user of our software can change the algorithm with command-line arguments by adding the argument "---hamilton" to their program.

In that case, we can have a function:

```java
    public ApportionmentMethod getApportionmentStrategy(List<String> arguments) {
        if (arguments.contains("--hamilton")) {
            return new HamiltonMethod();
        } else {
            return new HuntingtonHillMethod();
        }
    }
```

This lets us leverage polymorphism to make our code simpler, as now **every single other part of our program** only ever interacts with the abstraction `ApportionmentMethod` and never has to worry about whether under the hood the implementation is Hamilton or Huntington-Hill. We only need to know "which class to instantiate" when the program starts, and we never have to retain that knowledge elsewhere.

### The risk of abstraction

We have introduced what is called an **abstraction**. Be aware that in doing so, we have *coupled* `HamiltonMethod` and `HuntingtonHillMethod` somewhat. Namely, we are assuming that both can adhere to the given interface. We're also implicitly assuming that for *every* method we plan to implement going forward, we plan to adhere to this interface.

A place where this *could* become a problem is what if we need to introduce a method that takes in a List of States, but dynamically calculates on its own the number of representatives needed (say something like the Cube Root of the population)? Well, now our abstraction could become a burden, as we would potentially have to make several changes. Be aware that every abstraction we create comes with some coupling. In general, if an abstraction becomes burdensome, it might be better to remove it entirely!

## Conclusion

Learning how and when to decompose a program into separate classes can be one of the first big hurdles in understanding and implementing object-oriented programming. In this article, I gave 3 examples where I would separate one class into two:

1. Encapsulate a feature or behavior's **implementation** behind an interface 
2. Simplifying a complex interface into a simpler interface that more precisely meets our needs 
3. Combining two similar behaviors via polymorphism by extracting an abstract parent class.

We will revisit these ideas, the idea of polymorphism, and the Single Responsibility Principle throughout our [Design](https://sde-coursepack.github.io/modules/design/Design/) module.