---
Title: Polymorphism
---


# Polymorphism
{: .no_toc }

Polymorphism is a Greek word meaning "many-shaped". That definition lies at the core of how we use polymorphism in programming. Polymorphism is *the* key idea that makes object-oriented programming so powerful for designing software systems. In this module, we will explore the usage of Polymorphism. [In the next module](https://sde-coursepack.github.io/modules/objects/Benefits-of-Polymorphism/), we will discuss why polymorphism is such a powerful tool for building flexible code.


---

## Contents
{: .no_toc }

* TOC
{:toc}

---


## What does "Object-Oriented" give us, anways?

When people are first introduced to object-orientated programming, they probably think the key benefit is "classes and instances, methods and fields". Generally, what you are likely thinking of here is encapsulation. That is, we combine tightly coupled information into fields, and restrict its access through methods. However, that idea existing before object-oriented programming. 

For example, even though C is not object-oriented, we can use `structs` to handle encapsulation to some extent. For example, you can [find implementations of Linked-Lists in C](https://www.learn-c.org/en/Linked_lists) that look Object-Oriented. If you look at that link, you'll see a `struct` called Node. You'll see methods on a Linked List, like `push`, `print_list`, etc.

The only real difference is instead of saying something like:

`myLinkedList.push(5)`

You would say:

`push(myLinkedList, 5)`

This isn't a dramatic change. If the only thing Object-orientation gives us is the dot operator, I can't imagine it would be a big deal.

No, the key contribution of Object-orientation is **polymorphism**. The idea that a single class can have many shapes, and thus instances of that class can be used in different contexts!

## Liskov Substitution Principle

The Liskov Sustituion Principle is named for famous computer science pioneer and Turing Award winner [Barbara Liskov](https://en.wikipedia.org/wiki/Barbara_Liskov). The principle, which forms the basis of polymorphism in Object-oriented programming, is:

> Let Φ(x) be a property provable about objects x of type T. Then Φ(y) should be true for objects y of type S where S is a subtype of T.

Okay, maybe that isn't the most readable thing in the world. Let's put it this less formal way:

> "Any behavior described by a super-type must be implemented by the child-type."

Remember the relationship of subclasses and superclasses (aka child classes and parent classes) is an "is-a" relationship.

* `Square` is-a `Rectangle`
* `Rectangle` is-a `Parallelogram`
* `Parallelogram` is-a `Quadrilateral`

Let's consider the properties of a quadrilateral:
* It is a polygon
* It has 4 sides
* It has 4 angles

This is true of **every other** quadrilateral

Parallelograms:
* Have two pairs of parallel sides
* Opposite sides are equal to each other
* Opposite angles are equal to each other

This is true of all `Square` and `Rectangle` instances, but **not** `Quadrilateral`

Rectangles:
* All angles are 90-degrees

True of all `Squares`, but not try of all `Parallelogram`s and `Quadrilateral`s

As you can see, this relationship is one-directional:

Everything that is true of a parent-class is true of a child-class
However, not everything that is true of a child-class is true of a parent-class.

## Comparator

Consider the Java `interface` `Comparator<E>`. Instances of `Comparator<E>` are used in shorting methods, like the `List<E>` method `void sort(Comparator<E> comparator)`

A `Comparator` is an `interface` with one method: `int compare(E first, E second)`. It describes a means of sorting two elements where:
* a **negative** return value means `first` should be sorted **before** `second`.
* a **positive** return value means `first` should be sorted **after** `second`.
* a **zero** return value means they are (at least as far as sorting is concerned) equal.

However, the `Comparator` interface is **abstract**. That is, there is no class that can be instantiated called `Comparator`. Rather, `Comparator` only exists as a *description of a behavior*

### Abstract vs. Concrete

Understand the idea of **abstract** constructs can be difficult for many. Often, I will hear people invoke terms like "Platonic forms" which, while accurate, only serves to make the product seem more complicated than it is.

Instead, I ask you to think of a sandwich. Which of the following pictures are sandwiches and which aren't?

![img.png](../images/polymorphism/sandwich1.png)

![img.png](../images/polymorphism/sandwich2.png)

![img.png](../images/polymorphism/pizza.png)

You can probably easily tell that the first two pictures are sandwiches, and the last picture is *not* a sandwich.

That's because in your head, you have an **idea** of what makes-up a sandwich. And that **idea** is flexible. For example, that idea can accommodate:
* A grilled cheese sandwich
* A peanut-butter jelly sandwich
* A turkey sandwich
* A cucumber sandwich
etc.

All of these sandwiches are **concrete**. When we say concrete we mean something that is more tangible: you can make, hold, and eat a turkey sandwich. I can make a grilled-cheese sandwich. But I can't make the **idea** of a sandwich. A sandwich is simply an idea that describes the construction, components, and behaviors (such as eating by holding the two slices of bread) of several food items.

So we say that sandwich is **abstract**, since it's a description of a set of food. Whereas if you hold a **specific kind** of sandwich in your hands, that sandwich must necessarily be a **concrete** subtype, like a grilled-cheese or ham and swiss.

### Back to Comparator

Imagine we had a list of Students:

```public
public class Student {
    private int id;
    private String firstname, lastname;
    private Date birthDate
    private int year;
    
    // insert constructors, getters, and setters
}
```

From our student list, we could come up with several **concrete** ways to compare students for sorting (that is, several **implementations** of the **abstract** behavior of Comparator<Student>)

* By ID
* By Birthdate
* By Last Name, then First Name
* By year, then by Last Name, then First Name
* By year, then by id
...etc.

However, we don't need to write functions that actually execute the sorting. We can simply write a `Comparator` for each sorting approach:
* `public class StudentIDComparator implement Comparator<Student>`
* `public class StudentNameComparator implement Comparator<Student>`
etc.

In this case, we would say that `Comparator<Student>` is a **super-type**. `StudentIDComparator` and `StudentNameComparator` are **sub-types** of `Comparator`.

Each of these are **concrete** implementations of the **abstract** `Comparator<Student>` interface. Specifically, they **implement** the behavior: "compare two students and tell me which should come first". So, I can use them to sort:

* `studentList.sort(new StudentIDComparator())`
* `studentList.sort(new StudentNameComparator())`
etc.

However, I **cannot** say `studentList.sort(new Comparator<Student>())` because `Comparator<Student>()` is **abstract**. I have to use a particular **concrete** subtype of `Comparator<Student>`


## Example: Java Collections Framework

One visible example of this is the **Java Collections Framework**. The Java Collections Framework defines a series of common collections in Java, such as `List`, `Set` and `Map`. `Map` is a little different, since we're keeping track of effectively two separate collections (the keys and the sets), so lets focus on List and Set. List and Set describe interfaces. However, these interfaces are very similar. So similar, in fact, that they both implement the `Collection` interface.

### Collection<E>

The `Collection<E>` interface defines basic operations for a `Collection` containing elements of type E. `Collection` is an interface, meaning it is abstract. However, `Collection<E>` is implemented by all `List` and `Set` implementations. However, `Collection` can also describe abstract data types like `Stack`, `Queue`, and `Deque`. As such, each of those classes also implement the `Collection` interface.

`Collection<E>` methods include:

* `boolean add(E e)` - attempt to add an element to the correct - return `true` if successfully added, falsed otherwise.
* `int size()`- returns the size of the collection (number of elements in it)
* `boolean isEmpty()` - returns `true` if the collection is empty, `false` otherwise
* `boolean contains(Object o)` - returns `true` if `o` is in the collection
* `boolean remove(Object o)` - attempts to remove the first instance of `o` from the Collection. If it removes an element, return `true`, otherwise returns `false` otherwise.
* `Iterator<E> getIterator()` - gets an iterator (used in for-each loops)
* `Stream<E> stream()` - returns a `Stream` for the `Collection`


There is no meaingful direct implementation of a `Collection`. A Collection is an abstract idea of "a group of stuff". However, it describes the basic set of behaviors that are **shared** by `Set`, `List`, `Stack`, `Queue`, and `Deque`. In this case, we are describing the abstract behavior (interface), not the concrete implementation (working code).

### Set

`Set` implements all of the methods of `Collection`. However, a set adds a restriction: no duplicate members can be added to the set. This means that the `add` function returns false when someone attempts to add a duplicate item to the Set.

`Set` has two main implementations (there are more, but we use these two the most):
* `HashSet` - set which is organized by a hash table
* `TreeSet` - set which is organized by a balanced binary search tree

Because of **polymorphism**, we can write:

* `Set<String> words = new HashSet<>()`
* `Set<Integer> numbers = new TreeSet<>()`

Additionally, we can also write:

* `Collection<String> words = new HashSet<>()`
* `Collection<Integer> numbers = new TreeSet<>()`

This is because both `HashSet` and `TreeSet` implement `Set`, and `Set` in turn implements `Collection`.

So, we can say:
* Every `HashSet` is a `Set`
* Every `Set` is a `Collection`
* Therefore, every `HashSet` is a `Collection`

The same of course is true with `TreeSet`.

**But why is this useful?**

Let's imagine, for a second that we were implementing a list of valid words for Scrabble using a `Set`

```java
public class ScrabbleWordSet {
    private TreeSet<E> legalWords;
    public ScrabbleWordSet(TreeSet<E> legalWords) { this.legalWords = legalWords; }
    
    public ScrabbleWordSet() { this(new TreeSet<>()); }
    
    public boolean addWord(String s) { return legalWords.add(word); }
    
    public boolean isLegalWord(String s) { return legalWords.contains(word); }
    
    public TreeSet<E> getLegalWords() { new TreeSet<>(legalWords); }
}
```

You used a `TreeSet` because you wanted getLegalWords() to return the set of words alphabetically. However, you are now finding that was a mistake. Your class is used far, far, more frequently for `isLegalWord` than it is for `getLegalWords`. In fact, the different is multiple orders of magnitude. As such, you speed up your program, you want to switch to the default `Set` type of legal words to `HashSet`.

However, you cannot just change this class! Other classes in your program are calling the Constructor with a `TreeSet` (both in your production code and in your test code). As such, changing the interface of the constructor to take in a HashSet<> instead can't be done!

**This is a because of a violation of abstraction**

What you did wrong was you told the client classes what kind of `Set` you were using -- `TreeSet`. Because you gave that information, and required a `TreeSet`, other classes used a `TreeSet`. Now, changing that data type is going to be hard, because **every single place** in your entire project (tests, production, etc.) that used the constructor

`ScrabbleWordSet(TreeSet<E> legalWords)`

... now must be changed

It would have been better in the first place if you simply **hid the specific set being used!**.

For instance:

```java
public class ScrabbleWordSet {
    private Set<E> legalWords;
    public ScrabbleWordSet(Set<E> legalWords) { this.legalWords = new TreeSet<>(legalWords); }

    public ScrabbleWordSet() { this(new TreeSet<>()); }
    
    public boolean addWord(String s) { return legalWords.add(word); }
    
    public boolean isLegalWord(String s) { return legalWords.contains(word); }
    
    public TreeSet<E> getLegalWords() { new TreeSet<>(legalWords); }
}
```

Now, you'll notice that, as far as the client is concerned, your class can take in any `Set`. I still have getLegalWords returning a `TreeSet` because I want the client to be aware that the set is **sorted** (`HashSet` cannot be sorted). This class is much more modifiable. For example, let's say I change the internals to a `HashSet`, but because I want getLegalWords to be sorted, I still have it return a `TreeSet` for the sorted quality.

```java
public class ScrabbleWordSet {
private Set<E> legalWords;
    public ScrabbleWordSet(Set<E> legalWords) { this.legalWords = legalWords; }

    public ScrabbleWordSet() { this(new HashSet<>()); }
    
    public boolean addWord(String s) { return legalWords.add(word); }
    
    public boolean isLegalWord(String s) { return legalWords.contains(word); }
    
    public Set<E> getLegalWords() { new TreeSet<>(legalWords); }
}
```

You'll notice that I made the change **without changing the interface**! This is the advantage of encapsulation: separate how code is used (interface) from how it workds (implementation). But this also shows the benefit of polymorphism: this class is now as **general** as possible:

* The field `legalWords` can be any kind of `Set` without changing any of the methods.
* The one-argument constructor can take in any set type, meaning if I change from a default of `TreeSet` to `HashSet`, how clients (outside classes) **use** this constructor is completely unaffected!
* Because I still want `getLegalWords` to return a sorted set, I still need to return `TreeSet` because it is sorted. However, I can simply build the `TreeSet` from the contents of `legalWords` when the method is called. Yes, this is slower than before, but since this method is called rarely, the gains from `contains` and `add` are worth the cost.

**As a general rule when designing classes, your interface should always accept the broadest type definition that meets your needed behavior.**

### List

`List<E>` also implements `Collection<E>`, but it adds in the concept of **order**. That is, every list as an order set of elements. Each element is stored at a particular index. As such, this gives us a few new functions that aren't present in `Collection<E>` that use indices:

* `E get(int index)`
* `E remove(int index)`
* `void sort(Comparator<E>)`

There are two main implementations of `List`:
* `ArrayList` - implemented using an array
* `LinkedList` - implemented using a connected set of Nodes

In **almost** every case, `ArrayList` is faster than `LinkedList`. The main reason is that the `get` function on an `ArrayList` is constant time, whereas on a `LinkedList`, it's linear-time, relative to the size of the index. Additionally, from a computer architecture standpoint, looping through sequential memory (like an array) is significant faster than looping through non-sequential memory (linked-list).

However, there are two cases where a `LinkedList` is preferable, and that is in a `Queue` and a `Deque` (double-ended queue). This is because in a `Deque`, we need to be able to add and remove to both the **front** and **back** of the list.

A LinkedList can add to and remove from the front and back in **constant-time**. An ArrayList, however, requires **linear-time** to add to or remove from the **front** of the list (it can add and remove from the back in constant-time). Not that there is a way to implement a constant-time `Deque` using arrays, but `ArrayList` doesn't support that, as it requires significant specialization.

As such, in any given situation where you use a `List`, you might eventually want to change the `List` from one implementation to another. As such, whenever possible, your public interface should use only a `List`, and not an `ArrayList` or `LinkedList`, for the same reason as set.

Again: **As a general rule when designing classes, your interface should always accept the broadest type definition that meets your needed behavior.**

## Example: `poi.ss.usermodel.*`

Now let's look at `poi.ss.usermodel*`. Poi describes an abstract interface of `Workbook`s, `Sheet`s, `Row`s, and `Cell`s. These classes describe the behaviors we need to read in Excel files:

* `Workbook` has methods for getting a `Sheet` by name or index
* `Sheet` has a method for getting a `Row` by index, or getting a `Row<Iterator>`
* `Row` has methods for getting a `Cell` by index, or getting a `Cell<Iterator>`
* `Cell` has methods for extracting the contents of a Cell

However, **none of these methods are implemented.** This is an abstract interface: the *idea* of how these classes should behave for any generic spreadsheet file.

This is just like `Collection`, `List`, and `Set`. All of these are abstract interfaces. However, you cannot instantiate any of these types. Instead, you have to instantiate things like `HashSet` and `ArrayList`.

The same is true with using `poi`

### XSSFWorkbook

This is a **concrete** (implemented) class for reading and writing to Excel files from Microsoft Office 2007 and later (".xlsx").

### HSSFWorkbook

This is a **concrete** (implemented) class for reading and writing to Excel files from Microsoft Office version **before** Office 2007 (".xls").

The reason for the difference is that the file format of these files in different. Office 2007 and later use a file format based around XML, and use the extension `.xlsx`. Earlier versions used an in-house format that doesn't cleanly match any standards, and used the extension `.xls`. Because the formats of the files are different, reading or writing to the two files, **the implementation must necessarily be different**. 

For example, below are two methods for opening an Excel file and reading the string contents of cell A1. The first is for an ".xlsx" file, and the second is for an ".xls" file:

```java
    public String openXLSXWorkbookAndGetA1(String filename) throws IOException {
        FileInputStream stream = new FileInputStream(new File(filename));
        XSSFWorkbook workbook = new XSSFWorkbook(stream);
        XSSFSheet sheet = workbook.getSheetAt(0);
        XSSFRow row = sheet.getRow(0);
        XSSFCell cell = row.getCell(0);
        return cell.getStringCellValue();
    }

    public String openXLSWorkbookAndGetA1(String filename) throws IOException {
        FileInputStream stream = new FileInputStream(new File(filename));
        HSSFWorkbook workbook = new HSSFWorkbook(stream);
        HSSFSheet sheet = workbook.getSheetAt(0);
        HSSFRow row = sheet.getRow(0);
        HSSFCell cell = row.getCell(0);
        return cell.getStringCellValue();
    }
```

If this code looks the same, **it's because it is!** The only difference is the type on the left hand side and the constructor! That means, for example, we could rewrite the first version (for ".xlsx" as):

```java
    public String openXLSXWorkbookAndGetA1(String filename) throws IOException {
        FileInputStream stream = new FileInputStream(new File(filename));
        Workbook workbook = new HSSFWorkbook(stream);
        Sheet sheet = workbook.getSheetAt(0);
        Row row = sheet.getRow(0);
        Cell cell = row.getCell(0);
        return cell.getStringCellValue();
    }
```

We can similarly rewrite the second version, and then by extracting the method from the code that takes a workbook and gets the value of A1:

```java
    public String openXLSXWorkbookAndGetA1(String filename) throws IOException {
        InputStream stream = getInputStreamFromFilename(filename);
        Workbook workbook = new HSSFWorkbook(stream);
        return getStringFromCellA1(workbook);
    }

    public String openXLSWorkbookAndGetA1(String filename) throws IOException {
        InputStream stream = getInputStreamFromFilename(filename);
        Workbook workbook = new HSSFWorkbook(stream);
        return getStringFromCellA1(workbook);
    }

    private InputStream getInputStreamFromFilename(String filename) throws FileNotFoundException {
        stream = new FileInputStream(new File(filename));
    }

    private String getStringFromCellA1(Workbook workbook) {
        Sheet sheet = workbook.getSheetAt(0);
        Row row = sheet.getRow(0);
        Cell cell = row.getCell(0);
        return cell.getStringCellValue();
    }
```

Now, our code is more DRY ("don't repeat yourself"), and we have removed the "clone" where we were doing the same functionality twice, but in different methods. Additionally, our method `getStringFromCellA1(Workbook workbook)` works with **any** `Workbook` that implements that abstract `Workbook` interface. 

For a hypothetical, imagine in the future Excel eventually launches a new version of Excel that uses [YAML](https://yaml.org/) as a storage format, and saves files as ".xlsy". From there, the developers of poi may make a new `YSSFWorkbook` class that implements the `Workbook` interface. At that point, the **only** code I would need to add this class is:

```java
    public String openXLSYWorkbookAndGetA1(String filename) throws IOException {
        InputStream stream = getInputStreamFromFilename(filename);
        Workbook workbook = new YSSFWorkbook(stream);
        return getStringFromCellA1(workbook);
    }
```

I would not need to make **any** changes to **any other method**, include `getStringFromCellA1(Workbook workbook)`. By simply adding the one method to instantiate my hypothetical `YSSFWorkbook` class, I am *done*. 

**This makes my code extremely reusable**, and this is only possible with polymorphism!

We will look at more benefits with Polymorphism in our next unit!