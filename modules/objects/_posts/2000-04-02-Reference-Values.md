---
Title: Values vs. References
---

* TOC
{:toc}

# Values vs. References

In Java, all primitive data-types are stored as **values**, while all non-primitive types are stored as **references**. What does this mean?

Consider the following class:

```java
public class NumberAndList {
    private int number;
    private List<String> words;
    
    public NumberAndList() {
        number = 0;
        words = new ArrayList<>();
    }
    
    public void incrementNumber() {
        number++;
    }
    
    public int getNumber() {
        return number;
    }
    
    public void addWord(String word) {
        words.add(word);
    }
    
    public ArrayList<String> getWords() {
        return words;
    }
}
```

And now let's make an instance of that object:

```java
  NumberAndList nal = new NumberAndList();
```

A rough illustration of the state of the variable nal is shown below.

![nal_initial.png](..%2Fimages%2Foo-refresher%2Fnal_initial.png)

Note that in this diagram, each rectangle references an *instance* of the class it names. So we have an *instance* of NumberAndList, not the *class* NumberAndList.

Specifically, notice that the `int number` stores the **value** zero, while the `List<String> words` stores a **reference**. That is, we create an instance of the class `List<String>`, and the **memory address** of where that instance is stored is a reference. In this way, our instance variable `nal` doesn't *contain* a List, but it is attached to a `List<String>` through this reference. Additionally, the `ArrayList<String>` instance doesn't *contain* an array of `String`s, but rather is attached to an array of `String`s through a reference. This memory reference is represented in hexadecimal. The `ArrayList<String>`, however, does contain an integer to represent the current size of the `ArrayList`.

A quick note that `ArrayList`s work by actually creating an underlying array. By default, Java initializes the underlying array to be size 10. However, that does not mean `ArrayList` max out at 10 elements, as the ArrayList class will expand the size of it's underlying array as needed automatically. I point this out to note that this illustration is only showing 4 spaces in the underlying array for the sake of space. Additionally, all memory references are made up for the sake of this demonstration, and are determined at runtime by the Java Virtual Machine that is running the code.

If we ran the following code:

```java
  NumberAndList nal = new NumberAndList();
  nal.incrementNumber();
```

The picture would change as follows:

![nal_step_1.png](..%2Fimages%2Foo-refresher%2Fnal_step_1.png)

Note that in this case, the **value** of the variable `number` in our `NumberAndList` instance changed from 0 to 1. Contrast this with what happens when we run:

```java
  NumberAndList nal = new NumberAndList();
  nal.incrementNumber();
  nal.addWord("Apple");
```

![nal_step_2.png](..%2Fimages%2Foo-refresher%2Fnal_step_2.png)

Here, we have created a new `String` instance, `Apple`, which for the sake of this image is stored at memory location `a32398a4`. That *reference* is stored in the Array of Strings, which is reference by the ArrayList<String>. Note that the actual contents of our `nal` instance **didn't change**. Rather, the actual change occurred several references deep.

In this way, the variable `words` is **not a value**! It is a *reference* to an ArrayList. This can have some interesting consequences. Consider the following code block:

```java
  NumberAndList nal = new NumberAndList();
  nal.incrementNumber();
  nal.addWord("Apple");
  int x = nal.getNumber();
  ArrayList<String> myList = nal.getWords();
```

Here, a subtle but important thing happens:

![nal_step_3.png](..%2Fimages%2Foo-refresher%2Fnal_step_3.png)

Here, the two variables we created (`x` and `myList`) copy the values of the variables `number` and `words` in `nal`, respectively. However, this results in two very different behaviors.

Notice that when we say `myList = nal.getWords();`, this means that we are setting the variable `myList` to have **the same reference* as `words` in `nal`. That is, `myList` is **not** a copy of the `words` List; **it is literally the same List! By contrast, `x` is not the value as `number` in `nal`, but rather a copy of that value.

## The dangers of mutability

Consider, building from our previous example, the following:

```java
  NumberAndList nal = new NumberAndList();
  nal.incrementNumber();
  nal.addWord("Apple");
  int x = nal.getNumber();
  ArrayList<String> myList = nal.getWords();
  x = 5;
```

You'll note that the value of `x` changes, as we would expect, to 5. However, the value of `number` in `nal` *does not change*. This is because while `x` and `number` were at one point equal to each other, they were still *separate variables*. This meant their values can change independent of one another.

But now, consider this:

```java
  NumberAndList nal = new NumberAndList();
  nal.incrementNumber();
  nal.addWord("Apple");
  int x = nal.getNumber();
  ArrayList<String> myList = nal.getWords();
  x = 5;
  myList.add("Box");
```

Consider the possibly surprising result below:

![nal_step_5.png](..%2Fimages%2Foo-refresher%2Fnal_step_5.png)

Notice that we didn't just add `"Box"` to `myList`. We added "Box" to *both* `myList` **and** `words` in `nal`. This is because, while both variables are separate variables, their value is a **reference** to the *same ArrayList*.

## Why this is really bad!

This is a clear violation of abstraction! This allows us to edit an underlying `ArrayList` directly, rather than working only with the methods provided by `NumberAndList`.

More seriously, it's not obvious that any operation we perform on the variable `myList`, which is not even in the `NumberAndList` class will create side effects on the instance variable `nal`. This could very quickly turn into a serious issue where bugs are unintentionally injected, just because this relationship is unclear.

This is why we must always take care when using mutable values, such as Collections.

## Avoiding this problem

To avoid this problem, we want to make sure we **never** return the same reference to the same `ArrayList` that `nal.myList` references. Instead, we return a *copy* of the `List`.

One way to do this would instead be:

```java
public class NumberAndList {
    private int number;
    private List<String> words;
    
    ...
    
    public ArrayList<String> getWords() {
        ArrayList<String> wordsCopy = new ArrayList<>();
        for (String word: words) {
            wordsCopy.add(word);
        }
        return wordsCopy;
    }
}
```

In this way, we copy the List into a separate List, with a different memory reference, and return that instead. We can actually shorten this, as all Java Collections have a constructor used for making copies of themselves:

```java
public class NumberAndList {
    private int number;
    private List<String> words;
    
    ...
    
    public ArrayList<String> getWords() {
        return new ArrayList<>(words);
    }
}
```

This is a shorthand way of doing the same thing.

## Immutability

Now at this point, you may ask if we also need to copy the underlying String themselves, but it turns out we don't. This is because Strings are **immutable**. Once an instance is created, it cannot be changed.

This may sound odd, since the following code works:

```java
  String s = "Hello";
  s = s + " World!";
  System.out.println(s);
```

Didn't we just change the value of `s`? Well, yes and no.

* __Yes__- when we print the `String s`, it will print `Hello, World!`, but...
* __No__ - remember, `s`, being an object variable, stores a **reference**, *not the contents of the `String`*. And, as a result, the "value" of *s* (that is, the reference to the `String` contents) *changes* between the first and second line.

In this case, the "older" `String`, `"Hello"` is stored separately from the newer String we created, `"Hello, World!"`. This means that the `s` is not referencing the same "older" String, which remains unchanged.

## Takeaway

Be cautious whenever your class returns a mutable type. In general, if a class is using a mutable variable (such as a List), you should never return that variable directly! Instead, return a copy of that variable's contents! This is much safer from a program stability and debugging standpoint, although it comes at the cost of memory performance. If performance is a serious limitation, such as in embedded systems, then you should always take **great care** with mutable variables.