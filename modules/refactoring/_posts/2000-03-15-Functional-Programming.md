---
Title: Functional Programming
---

* TOC
{:toc}

# Functional Programming

Functional Programming is a programming paradigm (Object-oriented programming is another paradigm) where we treat functions like **first-class citizens**. What we mean by this is that functions can be treated as variables, passes as arguments, and even returned by functions. 

We can actually show this behavior in Python:

```python
def my_func(input):
    print("I am a function")
    print("My input is", input)
    
x = my_func
print(x)
print(x(10))
```

This prints:

```shell
<function my_func at 0x00000178068A2E18>
I am a function
My input is 10
```

That is, we assigned the variable `x` to the function `my_func`.

Unfortunately, Java *does not support treating functions* as first-class citizens. That is, we cannot assign functions to variables, pass them as arguments, etc.

## What Java has: SAMs

However, Java does support a workaround that lets us get closer to functional programming via **Functional Interfaces**. Functional interfaces are interfaces that can be used to define a **single abstract method** SAM. One example of a SAM interface you have seen is `Comparator`.

```java
public class StringIgnoreCaseComparator implements Comparator<String> {
    public int compare(String s1, String s2) {
        s1 = s1.toUpperCase();
        s2 = s2.toUpperCase();
        return s1.compare(s2);
    }
}
```

A `Comparator` is an interface that defines one function, `compare`, which takes in two instances of a particular class and returns a number indicating which item should go first if sorted. For example, if I call `compare(a, b)`:
- **Positive** - `a` comes after `b` when sorted
- **Negative** - `b` comes after `a` when sorted
- **Zero** - `a` and `b` are equal

Thus, can think of each class that implements `Comparator` as a separate *way to sort*. In this particular case, I am sorting Strings in a way that ignores case (since, by default, String sorting is case-sensitive, with all capital letters coming before all lowercase letters).

This class is, effectively, a wrapper for a single function. Because this is a class, we can make instances of this class and pass it to functions expecting a `Comparator`. For example:

```java
    List<String> words = getWordsAsListFromFile("myfile.txt");
    words.sort(new StringIgnoreCaseComparator());
```

The function `sort` is a method on `List` that takes in a `Comparator` and sorts the list using that `Comparator`'s `compare` function.

In this way, we can think of `new StringIgnoreCaseComparator()` as **effectively** a function, even if syntactically it is a class.

## Class Explosion

One downside of this approach, however, is the potential for an explosion in the number of classes. For example, take the following class:

```java
public class UVAStudent implements Comparable<UVAStudent> {
    private final int id;
    private String firstName, lastName;
    private final String computingID;
    private StudentType studentType;
    private double gpa;

    public UVAStudent(int id, String firstName, String lastName, String computingID, double gpa, StudentType studentType) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
        this.computingID = computingID;
        this.gpa = gpa;
        this.studentType = studentType;
    }
    //getters for all fields, setters for all non-final fields
    ...
}
```

This data structure class represents a student at UVA. Imagine we were making a GUI with a table of all students that lets us sort by any field above. Well, we have 6 fields, so this means at least 6 `Comparator` classes (12 if you also want reverse). All of these classes **only exist** for the sake of sorting a `List` of `UVAStudent` objects, yet will pollute our namespace. Now imagine our app has even more data classes, where each one also has Comparators for each field, and you can see how this blows ups quickly.

## Anonymous classes

One way around this is the anonymous class. That is, rather than write `StudentIdComparator` as a separate class in the package, we can define the class *in-line* in our code.

```java
public class UVAStudentManager {
    
    public void getStudentsSortedById(List<UVAStudent> studentList) {
        studentList.sort(new Comparator<UVAStudent>() {
            @Override
            public int compare(UVAStudent o1, UVAStudent o2) {
                return Integer.compare(o1.getId(), o2.getId());
            }
        });
    }
}
```

Here, wee have created an **anonymous class**. We define a class *in-line* inside of the sort Function with:

```java
    new Comparator<UVAStudent>() {
        @Override
        public int compare(UVAStudent o1, UVAStudent o2) {
            return Integer.compare(o1.getId(), o2.getId());
            }
    }
```

This lets us create an instance of a new class when we need it. The class doesn't have a name in the global namespace. This feature works well with *functional interfaces*, when we don't want a long-term class that has several instances, but instead are simply using the class as a wrapper for a function we want.

However, that syntax is still a bit long. Instead, wouldn't it be great if we could write something like this?

```java
    public void getStudentsSortedById(List<UVAStudent> studentList) {
        studentList.sort((s1, s2) -> Integer.compare(s1.getId(), s2.getId()));
    }
```

That is, we just define a **function** in-line: a function takes in two students, and compares them by their id. 

Well it turns out we can. Because the code above, as written, works. It sorts student in ascending order by their ID.

## Lambda functions

A lambda function is an **unnamed function**. Rather than naming a function, we create a function **when it's needed** on the fly *at runtime*. 

The general structure of a lambda statement is:

```java
(parameters) -> {code block}
```

For example, we can take our earlier `StringIgnoreCaseComparator` and define the `compare` function as a lambda body:

```java
    Comparator<String> ignoreCase = (String s1, String s2) -> {
        s1 = s1.toUpperCase();
        s2 = s2.toUpperCase();
        return s1.compareTo(s2);
    };
```

From there, we can shorten this a bit. For example, we know that the types of `s1` and `s2` can **only** be `String` because we are making a `Comparator<String>`. And so, we don't need to tell Java the datatype: Java will figure it out:

```java
    Comparator<String> ignoreCase = (s1, s2) -> {
        s1 = s1.toUpperCase();
        s2 = s2.toUpperCase();
        return s1.compareTo(s2);
    };
```

If we wanted, we could shorten the code block to one-line.

```java
Comparator<String> ignoreCase = (s1, s2) -> {
    return s1.toUpperCase().compareTo(s2.toUpperCase());
};
```

From there, we can actually go one step farther. Another valid format is:

```java
    (parameters) -> expression
```

In this case, whatever `expression` evaluates to is returned, without needing to invoke the `return` keyword. For example:

```(int x, int y) -> x + y```

Is a lambda function for getting the sum of two numbers. In this case, whatever x and y add up to will be returned

Using that, going back to our `ignoreCase` Comparator, we can say:

```java
Comparator<String> ignoreCase = (s1, s2) -> 
        s1.toUpperCase().compareTo(s2.toUpperCase());
```

To test this out, we can make a List of strings and sort it...

```java
    List<String> words = new ArrayList<>(List.of("Apple", "Zebra", "banana", "Catfish", "aardvark"));
    words.sort((s1, s2) -> s1.toUpperCase().compareTo(s2.toUpperCase()));
    System.out.println(words);
```

...and it will print:

`[aardvark, Apple, banana, Catfish, Zebra]`

### Quick note:

If a lambda body has **exactly one argument**, you do not need parentheses around the first value (they are optional).

```(x) -> x * x```
```x -> x * x```

Both work, and both define a lambda for squaring an input number.

## We've seen this before

Think back to `assertThrows` in our testing unit:

```java
@Test
public void testWithdrawInsufficientFunds() {
    BankAccount account = new Account(500);
    
    assertThrows(InsufficientFundsException.class, ()-> 
        account.withdraw(700));
}
```

If you look at the [official javadoc for the assertThrows method](https://junit.org/junit5/docs/5.0.3/api/org/junit/jupiter/api/Assertions.html#assertThrows-java.lang.Class-org.junit.jupiter.api.function.Executable-), you'll see the signature is:

```public static <T extends Throwable> T assertThrows(Class<T> expectedType, Executable executable)```

The JUnit 5 interface `Executable`, like `Comparator` is a *functional interface*. Like all other functional interfaces, it has one method:

`public void execute()`

This method takes in nothing, and returns nothing. So, when we write...

```()-> account.withdraw(700)```

...we are defining a function that takes in nothing, and simply calls `account.withdraw(700)`. And we are asserting that, while executing this function, an `InsufficientFundsException` will be thrown.

We could have written this out as:

```java
    Executable testExecutable = new Executable() {
        @Override
        public void execute() throws Throwable {
            testAccount.withdraw(600);
        }
    };
    assertThrows(InsufficientFundsException.class, testExecutable);
    assertEquals(500, testAccount.getBalance(),1e-4);
```

However, the above code is arguably harder to read than:

```java
    assertThrows(InsufficientFundsException.class, () -> testAccount.withdraw(600));
    assertEquals(500, testAccount.getBalance(),1e-4);
```

This is because there is more text that is **extraneous**, or unnecessary. For example `Executable`, `new Executable() `, `@Override`, etc. All of this introduces **accidental complexity** to our test that can be removed. Since both do the same thing, we use the second version.

In essence, we want to highlight the **function** and hide the **class**, since the class only exists to be a simple wrapper for the Function.

## Functional interfaces to know:

### Comparator<E>

__method__: `public int compare(E e1, E e2)`

__use__: used in sorting lists, specifically `Collections.sort(List<E> list, Comparator<E> comparator)` and `listInstance.sort(Comparator<E> comparator)`. Also used in defining `TreeSet`s.

__example__: `(a, b) -> a - b`

Assuming a and b are number types (`int`, `double`, etc.), would be used to sort number in ascending order.

### Executable (JUnit5)

__method__: `public void execute()`

__use__: used in `assertThrows`, `dynamicTest` and `assumingThat`.

__example__: `() -> student.enroll(cs3140)`

Defines an executable where a student enrolls in a given class. Specifically in JUnit, this is primarily used in `assertThrows` to check for an `Exception` being thrown.

Be aware that there is a separate Java class called `Executable` which **is-not** a functional interface. Java, however, does have an equivalent functional interface called `Runnable` which you used with Threads:

`Thread(Runnable target)`

As such, you can define a thread using lambdas:

`Thread newThread = new Thread(() -> myRunFunction())`

### Predicate<E>

__method__: `public boolean test(E e)`

__use__: used for checking if a value meets some condition. Specifically, it is used in the `filter` method for `Stream`s (covered in the next module).

__example__: `(student) -> student.getGPA > 3.5`

The above returns true for students with at least a 3.5 [would be eligible for the Dean's List at UVA](https://college.as.virginia.edu/awardsandhonors).

### Consumer<E>

__method__: `public void accept(E e)`

__use__: Take in some value and doing something with it. Used specifically in Java's `foreach` function on Collections.

__example__: `(value) -> System.out.println(value)`

This can allow us to replace:

```java
    for(Item item : itemList) {
        System.out.println(item);
    }
```

With:

```itemList.forEach(item -> System.out.println(item))```

### Supplier<E>

__method__: `public E get()`

__use__: get results from some source and do something with it.

__example__: `() -> (int) (Math.random() * 6) + 1`

Generate a random integer from 1-6 (rolling a six-sided die);

### Function<E, R>

__method__: `public R apply(T)`

__use__: pass some instance of T as an argument, return an instancec of R. Used in the `map` function in Java Streams.

__example__: `x -> x.toString().toUpperCase()`

A function that takes in some object `x` and returns its `String` representation as an uppercase `String`.

### ActionListener

__method__: `public void actionPerformed(ActionEvent e)`

__use__: Used in event-driven applications to respond to user interactions. We will see this in our GUI unit working with JavaFX.

__example__: `e -> handleButtonPress()`

Structurally, the same thing as `Consumer<ActionEvent>`, its more specialized for event-driven applications (typically GUIs).

## Method Captures

Taking a second, let's look back at our `Consumer` example:

```itemList.forEach(item -> System.out.println(item))```

In this case, you'll notice that the input to our `Consumer` is exactly the same as our input to `System.out.println`. That is, our consumer is simply passing our existing input to another function. Whenever we are using a lambda body to simply pass our input to another function that expects the same input, we can use a method capture.

```itemList.forEach(System.out::println)```

That is, we simply use `System.out.println()` **as the Consumer function**.

Another example of this can be seen with `Comparator`.

Say we had the following code for a given `List<UVAStudent>`:

```java
    studentList.sort((x, y) -> {
            if (x.getLastName().equals(y.getLastName())) {
                return x.getFirstName().compareTo(y.getFirstName());
            } else {
                return x.getLastName().compareTo(y.getLastName());
            }
        });
```

This lambda-body is too big. But this doesn't mean we can't use this function in a lambda body. First, we can definite a function in our `StudentManager` class:

```java
    public int compareLastNameThenFirstName(UVAStudent s1, UVAStudent s2) {
        if (s1.getLastName().equals(s2.getLastName())) {
            return s1.getFirstName().compareTo(s2.getFirstName());
        } else {
            return s1.getLastName().compareTo(s2.getLastName());
        }
    }
```

And now we can invoke that function in our lambda body:

```java
public class UVAStudentManager {
    public void sortStudentsLastNameThenFirstName(List<UVAStudent> studentList) {
        studentList.sort(this::compareLastNameThenFirstName);
    }

    public int compareLastNameThenFirstName(UVAStudent s1, UVAStudent s2) {
        if (s1.getLastName().equals(s2.getLastName())) {
            return s1.getFirstName().compareTo(s2.getFirstName());
        } else {
            return s1.getLastName().compareTo(s2.getLastName());
        }
    }
}
```

Note that we use `this` because we are calling that function on our instance. However,, because the function `compareLastNameThenFirstName` doesn't use any instance variables (it only uses the functions inputs), we can safely make the function `static`. Because the function is static, we can refer to it by its class name:

```java
public class UVAStudentManager {
    public void sortStudentsLastNameThenFirstName(List<UVAStudent> studentList) {
        studentList.sort(UVAStudentManager::compareLastNameThenFirstName);
    }

    public static int compareLastNameThenFirstName(UVAStudent s1, UVAStudent s2) {
        if (s1.getLastName().equals(s2.getLastName())) {
            return s1.getFirstName().compareTo(s2.getFirstName());
        } else {
            return s1.getLastName().compareTo(s2.getLastName());
        }
    }
}
```

We will look more at lambda bodies and method captures in the next unit.
