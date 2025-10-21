---
Title: Java Streams
---

# Java Streams

Using what we have learned about **Functional Programming**, we are now ready to tackle **Streams**.

A **stream** is like an assembly line. We take in some collection of information on one side, pass it through a number of steps, and then at the end we have some useful information. In this module, we will look at why Streams are useful, how they can improve our code readability, and how to use them.


* TOC
{:toc}



## Simple queries

We often want to gather summary data of some input set. For example, if I have a class:

```java
public class State {
    private String name;
    private int population;
    
    //normal constructors, getters, setters
}
```

I might say something like, "Give me the total population of all states." To implement this, your first thought would likely be an accumulator pattern.

```java
    public int getTotalPopulation(List<State> stateList) {
        int sum = 0;
        for (State state : stateList) {
            sum += state.getPopulation();
        }
        return sum;
    }
```


This is pretty straightforward, easy to read, succinct code. 

### Complex queries

But now let's consider a more complicated query:

"Give me the smallest 5 states, in order of smallest to largest."

This one gets tricky. There are a lot of ways to implement it. My naive approach that required the least mental effort is:

```java
public List<State> getSmallestNStates(List<State> stateList, int numberOfStates) {
        List<State> safeCopy = new ArrayList<>(stateList);
        safeCopy.sort(new Comparator<State>() {
            @Override
            public int compare(State o1, State o2) {
                return Integer.compare(o1.getPopulation(), o2.getPopulation());
            }
        });
        List<State> smallestNStates = new ArrayList<>();
        for (int i = 0; i < numberOfStates; i++) {
            smallestNStates.add(safeCopy.get(i));
        }
        return smallestNStates;
    }
```

Reading all this code, I can't help but think "there's gotta be an easier way." (You may be thinking I can replace the Comparator with a lambda body, and you're right, but we'll get there).

### Why is this so ugly?

Okay, now let's take a class called `Representation` (from our [Extract Class](https://sde-coursepack.github.io/modules/refactoring/Extract-Class/) module).

```java
public class Representation {
    Map<State, Integer> representation = new HashMap<>();

    public void addRepresentativesToState(State state, int newRepresentatives) {
        int currentRepresentatives = getRepresentativesForState(state);
        representation.put(state, currentRepresentatives + newRepresentatives);
    }

    public int getRepresentativesForState(State state) {
        return representation.getOrDefault(state, 0);
    }

    public List<State> getStateList() {
        return new ArrayList<>(representation.keySet());
    }
}
```

...and let's ask, "Print the states in alphabetical order with their number of representatives." This *sounds* like it should be simple. And yet, our code may look like:

```java
    public void printRepresentationAlpabeticalOrder(Representation representation) {
        List<State> alphabeticalStates = representation.getStateList();
        alphabeticalStates.sort(new Comparator<State>() {
            @Override
            public int compare(State o1, State o2) {
                return o1.getName().compareTo(o2.getName());
            }
        });
        List<Integer> parallelPopulationMap = new ArrayList<>();
        for (State state : alphabeticalStates) {
            parallelPopulationMap.add(representation.getRepresentativesForState(state));
        }
        for(int i = 0; i < alphabeticalStates.size(); i++) {
            System.out.println(alphabeticalStates.get(i) + " - " + parallelPopulationMap.get(i));
        }
    }
```

*Oof*. This code has 3 different for loops, and relies on a parallel ArrayList. Now, if you're clever, you can combine the two later loops into one, but even still, this feels far more complicated than it needs to be.

---

## Streams

So now we turn to Streams. Streams were introduced in Java 8, and allow us to utilize **functional programming** to simplify queries and operations on collections of data. For example, let's go back to our total population example:

```java
    public int getTotalPopulation(List<State> stateList) {
        int sum = 0;
        for (State state : stateList) {
            sum += state.getPopulation();
        }
        return sum;
    }
```

Instead of this, we can do this with streams:

```java
    public int getTotalPopulation(List<State> stateList) {
        return stateList.stream()
            .mapToInt(state -> state.getPopulation)
            .sum();
    }
```

Instead of ...

```java
    public List<State> getSmallestNStates(List<State> stateList, int numberOfStates) {
        List<State> safeCopy = new ArrayList<>(stateList);
        safeCopy.sort(new Comparator<State>() {
            @Override
            public int compare(State o1, State o2) {
                return Integer.compare(o1.getPopulation(), o2.getPopulation());
            }
        });
        List<State> smallestNStates = new ArrayList<>();
        for (int i = 0; i < numberOfStates; i++) {
            smallestNStates.add(safeCopy.get(i));
        }
        return smallestNStates;
    }
```

...we can do this with streams:

```java
    public List<State> getSmallestNStates(List<State> stateList, int numberOfStates) {
        return stateList.stream()
            .sorted(Comparator.comparing(State::getPopulation()))
            .limit(numberOfStates)
            .collect(Collectors.toList());
    }
```

And instead of:

```java
    public void printRepresentationAlpabeticalOrder(Representation representation) {
        List<State> alphabeticalStates = representation.getStateList();
        alphabeticalStates.sort(new Comparator<State>() {
            @Override
            public int compare(State o1, State o2) {
                return o1.getName().compareTo(o2.getName());
            }
        });
        List<Integer> parallelPopulationMap = new ArrayList<>();
        for (State state : alphabeticalStates) {
            parallelPopulationMap.add(representation.getRepresentativesForState(state));
        }
        for(int i = 0; i < alphabeticalStates.size(); i++) {
            System.out.println(alphabeticalStates.get(i) + " - " + parallelPopulationMap.get(i));
        }
    }
```

...we can use streams:

```java
    public void printRepresentationAlpabeticalOrder(Representation representation) {
        representation.getStateList().stream()
            .sorted(Comparator.comparing(State::getName))
            .map(state -> state.getName() + " - " + representation.getRepresentativesForState(state))
            .forEach(System.out::println);
    }
```

## Wait...what is going on?

Don't worry, we will break this code down. But as you can see, understanding lambda bodies is going to be very valuable here. `Stream`s go together with functional programming like peanut butter and chocolate.


## Beginning - `.stream()`

A stream starts with a Collection, like a `List` or a `Set` (not a `Map` directly, however, though we can convert `keySet()`, `values`, and `entrySet()` of `Map`s to a `Stream`). From there, we make a stream using:

`myCollectionVariable.stream()`

This gives us a Stream that we can now perform zero or more **intermediate** operations on (which will be the bulk of our logic) and one *terminal* option on, which converts the stream back into something useful like a `List`, or gives us summary information (such as an integer sum).

---
---

## Intermediate operations

Think of intermediate operations as an assembly line. Each step on the assembly line, another step is implemented. Data from the streams comes in, a modification of that data goes out. `Stream`s are generically typed. For each of the methods below, we will assume the input is a `Stream<E>`, or Stream of type E.

For example, when I say the `sorted` method is `sorted(Comparator<E>)`, 'E' is the datatype stored in that `Stream` at that time. The time of the Stream can change over intermediate operations (specifically `map`), so in that context `E` refers to the data type of the `Stream` at the start of the `sorted` step.

---

### `sorted`

__method__: `sorted(Comparator<E> comparator)`

__example__: `.sorted((a, b) -> a.size() - b.size())`

__output__: `Stream<E>` sorted by the `comparator`

__explanation__: We may want to sort during our intermediate steps (such as when we get the five smallest states). This method lets us define a sorting methodology functionally. 

__related functions__: 

`Comparator<E> Comparator.comparing(Function<E,R> keyExtractor)`

Sort by a particular key. For example:

Assuming input is `Stream<State>.sorted(Comparator.comparing(x -> x.getName())`

This will sort states by their `name` field. Because `name` is a String, it uses the `String` instance method `compareTo`. When used with numbers, this defaults to sorting in ascending order.

Additionally, we can capture the getStateName() method to make this even simpler to read:

`.sorted(Comparator.comparing(State::getName())`

This line should be read as "Sort states by their name by their natural order" (where natural order is the result of `.compareTo()`). Be aware that you can only use this for data types that have implemented `Comparable`. `String`, `Integer`, `Double`, and other wrapper classes for primitive datatype have implemented `Comparable`, and are safe to use here.

Additionally, another useful function is `reversed()` which can be called on a Comparator to reverse it. For example, by default, `Comparator.comparing(State::getPopulation())` sorts in ascending order by population. But `Comparator.comparing(State::getPopulation()).reversed()` sorts in descending order.

---

### `filter`

__method__: `filter(Predicate p)`

__example__: `.filter(x -> x.getPopulation > 1000000)`

__output__: `Stream<E>` with only elements that return `true` for our `Predicate` function. Elements that return `false` are removed from the Stream.

__explanation__: We may want to "filter out" certain values from our stream, or only keep certain other values. For example, above, we are saying "only keep items whose population is over 1 million." Thus, any state with a smaller population will be removed by this filter.

---

### `limit`

__method__: `limit(long n)`

__example__: `.limit(10)`

__output__: `Stream<E>`, but only the first `n` elements.

__explanation__: In general, if we want to use `limit`, we only use it after `sorted`. For example:

```java
    stateList.stream()
        .sorted((s1, s2) -> s1.getPopulation() - s2.getPopulation())
        .limit(numberOfStates)
        .forEach(System.out::println)
```
...gives us a `Stream<State>` containing `numberOfState` objects. Because we sorted by `population` in ascending order immediately before calling `limit`, this means we have the `numberOfState` smallest states by population.

---

### `map`

__method__: `map(Function<E, R> f)`

__example__: `.map(x -> x.size())`

__output__: `Stream<R>`. In the example above, this would be something like Stream<Integer>, most likely.

__explanation__: Used to convert our data from one thing to another. For example, say we had a `Stream` of `String` that contain integers:

`["45", "13", "27"]`

We could do something like:

`.map(Integer::parseInt)`

This would result in a `Stream<Integer>`

`[45, 13, 27]`

In our apportionment example, we converted `State` objects into `String` objects with:

`.map(state -> state.getName() + " - " representation.getRepresentativesForState(state))`

---

### `distinct()`

__method__: `distinct()`

__example__: `.distinct()`

__output__: `Stream<E>` with all duplicate elements removed.

__explanation__: Uses `E`'s .equals() method to remove any duplicate elements from our Stream. For example:

`["John", "Mark", "John", "Kelly", "Kelly", "Kelly"]`

would become:

`["John", "Mark", "Kelly"]`

---

### `peek`

__method__: `peek(Consumer<E> e)`

__example__: `.peek(System.out::println)`

__output__: `Stream<E>` with no elements removed or altered (unless altered as a side effect of the `Consumer` function in `peek`)

For example:

`.peek(state -> setName(getName.toUpperCase())`

**will** change the name value of all states in the Stream simply because `setName` is a destructive function (one that changes the state of its instance).

__explanation__: Similar to `forEach`, this method performs some function on every element in the Stream. However, `forEach` is a **terminal** operation, while peek is an **intermediate** operation.

### `flatmap`

__method__: `flatmap(Function<E, R> f)` where Function must return a `Stream` of some time

__example__: `.flatmap(x -> x.stream())`

__output__: `Stream<R>` where all sub-collections in the previous stream have been flattened into a single stream.

__explanation__: useful when we have a Stream of Collections, or some type that is composed of another type, and we want to decompose these aggregations into their component parts.

For example, if our input stream were a Stream<List<Integer>> structured like:

```java
        List<Integer> a = List.of(12, 13, 14);
        List<Integer> b = List.of(26, 2, 19);
        List<Integer> c = List.of(3, 6, 9);
        List<List<Integer>> combined = List.of(a, b, c);
        System.out.println(combined);

        List<Integer> flat = combined.stream()
            .flatMap(x -> x.stream())
            .toList();

        System.out.println(flat);
```

This will print:

```shell
[[12, 13, 14], [26, 2, 19], [3, 6, 9]]
[12, 13, 14, 26, 2, 19, 3, 6, 9]
```

You can see that the collections have been "flattened" into a single Stream.

---
---

## Terminal Operations

All streams end with a single terminal operation.

### `foreach`

__method__: `forEach(Consumer<E> e)`

__example__: `.peek(item -> summaryList.add(item))`

__output__: `void` - forEach cannot be used to return anything directly

__explanation__: used for performing some action with items left in the `Stream` at the end of the intermediate operations.

### `count`

__method__: `count()`

__example__: `.count()`

__output__: `long` - the number of items left in the `Stream`

### collect()

__method__: `collect(Collector<E, A, R> c)`

__example__: `.collect(Collectors.toList())`

__output__: Some `R` value, which is an accumulation of the elements left in the `Stream`

__explanation__: Perform some aggregation on the left over items.

__related functions__

* `Collectors.toList()` - returns a `List<E>` which matches the state of the `Stream` at the end of the `Stream` chain of operations. Note that this List<E> is **unmodifiable**. However, you can do something like:

```java
List<E> outputList = originalList.stream()
        .[intermediate operations]
        .collect(Collectors.toList());
List<E> modifiable = new ArrayList<>(outputList);
```

A quick note that in Java 16, the `Stream` method `toList()` was added, so you can do:

```java
List<E> outputList = originalList.stream()
        .[intermediate operations]
        .toList();
```

Just be aware that this is only in Java 16 and later. That list returned is also **immutable**.

* `Collectors.toSet()` - returns a `Set<E>` which matches the state of the `Stream` at the end of the `Stream` chain of operations. Because this results in a Set, duplication elements are removed. Like `toList()`, the output set is **unmodifiable**.

* `Collectors.toCollection(ArrayList::new)` - allows you to immediately generate a *modifiable* collection if you wish.

* `Collectors.toMap(Function<E,R> keyFunction, <E, T> Function valueFunction)` - create a *modifiable* Map where, for each value, `keyFunction` is used to determine the 'key', and 'value' is used to determine the map.

```java
    Map<String, Integer> namePopulationMap = stateList.stream()
        .collect(Collectors.toMap(s -> s.getName(), s -> s.getPopulation()));
```

Note that unlike `toSet`, `toMap` will throw an `IllegalStateException` if you try to add duplicate keys.


* `Collectors.joining()` - can be used for joining elements of a `Stream<String>` together. Joining can take in an argument that is used as a delimiter between the Strings.

```java
List<String> myList = List.of("Aaron", "Betty", "Carol", "David");
String outputString = myList.stream.joining("\n");
System.out.println(outputString);
```

...prints:

```shell
Aaron
Betty
Carol
David
```

* `Collectors.counting()` - does the same thing as `count()`

---

### average, sum

Note that for all of these, the function specifies the numeric datatype returned. For example:

`Collectors.averagingDouble(Function<E, Double>)` will also have functions:
* `Collectors.averagingInt(Function<E, Integer>)`
* `Collectors.averagingLong(Function<E, Long>)`

For the sake of limiting repetition, I'm only going to show the `Double` version, but just remember you can replace `Double` with `Int` and `Long`, which makes the Function return types `Int` and `Long`. Be aware that `averagingInt` and `averagingLong` return `double`s, since the average of integer numbers may be a decimal number.

`double averagePopulation = stateList.stream().collect(Collectors.averagingInt(State::getPopulation)))`

If you already have a `Stream<Double>`, then just map the value to itself. For example: `(x -> x)`

```java
    double averagePopulation = stateList.stream()
        .map(State::getPopulation)
        .collect(Collectors.averagingInt(p -> p));
```

* `Collectors.summingDouble(Function<E, Double> function)` - same thing as `averaging` but returns the sum.

---

### max, min

* `max(Comparator<E> comparator)` - finds the maximum remaining value using `comparator`. In order to get the max, the `Comparator` passed in should be in "ascending" order.

* `min(Comparator<E> comparator)` - finds the minimum remaining value using `comparator`. In order to get the max, the `Comparator` passed in should be in "ascending" order.

Note that this method returns `Optional<E>` and not `E`. The reason for this is that if the `Stream` is empty when `max` is invoked, there is no value present to be a maximum.

Below shows an example of dealing with this inconvenience

```java
    Optional<State> biggestState = stateList.stream()
        .max(Comparator.comparing(State::getPopulation));
    if (biggestState.isPresent()) {
        State state = biggestState.get();
        System.out.println(state.getName());
    } else {
        throw new RuntimeException();
    }
```

---

### reduce

`reduce` allows you to define a way to `reduce` a stream of multiple objects to one value.

For example, we can use `reduce` to get a sum:



```java
    int totalPopulation = stateList.stream()
        .map(State::getPopulation)
        .reduce(0, (subtotal, statePopulation) -> subtotal + statePopulation);
```

In the above arguments for `reduce`, we are saying:
* our initial value for `subtotal` is zero
* for each value in our stream, set `subtotal` to be equal to `subtotal + statePopulation`
  * The first argument in our function is our "summary" value
  * The second argument is each item in our list

---
---

## `parallelStream`

We can replace `stream()` with `parallelStream()` to take advantage of automated multi-threading. However, be aware that `parallelStream()` may have operations in an unpredictable order. For example:

```java
    public void printRepresentationAlpabeticalOrder(Representation representation) {
        representation.getStateList().stream()
        .sorted((s1, s2) -> s1.getName().compareTo(s2.getName()))
        .map(state -> state.getName() + representation.getRepresentativesForState(state))
        .forEach(string -> System.out.println(string));
    }
```

...always prints in the order resulting from `sorted`. However, replacing `stream()` with `parallelStream()`

```java
    public void printRepresentationAlpabeticalOrder(Representation representation) {
        representation.getStateList().parallelStream()
        .sorted((s1, s2) -> s1.getName().compareTo(s2.getName()))
        .map(state -> state.getName() + representation.getRepresentativesForState(state))
        .forEach(string -> System.out.println(string));
    }
```

...will result in values being printed out of order, as the thread execution timings are not predictable. For example, for the above, I got this as my last 10 lines:

```shell
  Vermont - 1
  California - 52
  Georgia - 14
  Louisiana - 6
  Wyoming - 1
  Wisconsin - 8
  Florida - 28
  Maryland - 8
  Connecticut - 5
  Hawaii - 2
```

...which clearly is not alphabetical. By contrast, when I simply use `stream()` I get the results in alphabetical order every time.

### Workaround

One workaround for this problem is to use `forEachOrdered()` instead of `forEach()` with multi-threading:

```java
    public void printRepresentationAlpabeticalOrder(Representation representation) {
        Representation.getStateList().parallelStream()
        .sorted((s1, s2) -> s1.getName().compareTo(s2.getName()))
        .map(state -> state.getName() + representation.getRepresentativesForState(state))
        .forEachOrdered(string -> System.out.println(string));
    }
```

### ParallelPerformance

Be aware that while `parallelStream()` will automatically use multi-threading, that doesn't necessarily mean your code will run faster. Managing threads creates a lot of overhead complexity. As such, `parallelStream` will typically only benefit you with *very* large data sources.


## Files.line

We can use `Files.lines()` and `BufferedReader.lines()` as a `Stream<String>` for File reading:

```java
List<State> stateList = br.lines()
        .map(line -> line.split(","))
        .map(line -> new State(line[0], Integer.parseInt(line[1])))
        .collect(Collectors.toList());
```

Of course, this approach isn't great for handling Exceptions that may emerge. For that, you'll need to look into the `CheckedFunction` and `Either<E>`, but I will leave that to you if you want to dive-deep on `Stream`s. Be aware that reading a file is necessarily a sequential-stream, and cannot be run in parallel.

## Conclusion

There is a lot of potential added complexity when working with `Stream`s at first. However, once you are more comfortable with functional programming (including lambda bodies), they can dramatically simplify the writing and reading of operations you previously did with `for`-loops on collections.