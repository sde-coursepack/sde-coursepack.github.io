---
Title: Functional Programming
---

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

One way around this is the anonymous class. That is, rather than write `StudentIdCCompartor` as a separate class in the package, we can define the class *in-line* in our code.

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


