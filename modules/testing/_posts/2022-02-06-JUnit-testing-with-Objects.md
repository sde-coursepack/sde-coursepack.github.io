---
Title: JUnit with Objects
---


# Testing with Objects

So far, with testing, we have largely focused on simple functions
like `int max(int a, int b, int c)`. This is a good place to start
because these functions have very easy to define input and output.

* __Input__: the input parameters, like `(3, 2, 1)`
* __Output__: the return value of the function, like `3`

However, much of our code doesn't work like this. In fact, if
we are using object-oriented design we cannot just test
the input arguments and output values and have a sufficiently
complete picture of our code. In this unit, we will look at testing mutable objects, and how our approach changes.

---

* TOC
{:toc}



---


## Different from testing methods

Consider the class [`NumberChanges`](https://github.com/sde-coursepack/TestingIntro/blob/main/src/main/java/edu/virginia/cs/testingintro/NumberChanges.java).
The purpose of this class is to keep track of a number. However,
it also keeps track of how many times that number has been changed.

For example, in a given instance, `nc`, of `NumberChanges` if:

* `nc.number` = 7
* `nc.changes` = 4

Then that means the *current* value of `number` for the instance
`nc` is 7, and that number has been changed (`changes`) 4 times.

---


## Testing state changes

Considering this example, what if we wanted to change `nc`'s `number`
to 13? Well, we could call:

`nc.setNumber(13)`

This will result in the values of `nc` changing:

* `nc.number` = **13** (the number we changed to)
* `nc.changes` = **5** (we changed the number again!)

In this case, our **input** is the **state** of `nc` **before** we
call `nc.setNumber(13)`, and the **output** is the **state** of `nc`
**after** we call `nc.setNumber(13)`

---


## Writing our test

When testing with objects where we are checking for **state changes**,
we follow the following recipe:

1) Create the test object and configure it into the starting state  
2) Execute the operation to be tested  
3) Check the state of the object after the operation to ensure it changed as specified  

Using our above example, we can write the test:

```java
    @Test
    void testAlreadyChangedSetNumberChanged() {
        //Setup and configure test object
        NumberChanges nc = new NumberChanges(7, 4);
        
        //run operation to be tested
        nc.setNumber(13);
        
        //Check to ensure object behaved as specified
        assertEquals(13, nc.getNumber(), "Number did not change to 13 correctly");
        assertEquals(5, nc.getTimesChanged(), "Number of times change did not correctly increment");
    }
```

Be aware that I'm intentionally added more comments than I
normally would as this is meant to be an illustrative example.

But let's break down each step:

1) Create the test object and configure it into the starting state

```NumberChanges nc = new NumberChanges(7, 4);```

It is important when performing this step that you **should never
call the method you are testing!**. The reason is that if you call
the method you are testing multiple times, it's difficult to tell
which time the tested method caused a bug if the test fails. We will discuss
this more in "Only call the tested operation once" below.

2) Execute the operation to be tested

```java
    nc.setNumber(13);
```

Note that if this function returned something, we could test 
the output value with an assertStatement. For
example, imagine `setNumber` returned a boolean (`true` if the
number changed, `false` if it didn't.) Example:

```java
    assertTrue(nc.setNumber(13));
```

However, in this case, our method **is** void, so there's no
output to check.

3) Check the state of the object after the operation to ensure it changed as specified


```assertEquals(13, nc.getNumber(), "Number did not change to 13 correctly");```  
```assertEquals(5, nc.getTimesChanged(), "Number of times change did not correctly increment");```

Note that we need to check *both* conditions in this case! This is
because the method can change *both* numbers, so we want to ensure
that both are changing correctly! **It is not sufficient to only
test `nc.getNumber()` OR `nc.getChanges`.

---


## A second test case:

Of course, as we test this method, there is another case to
consider. What if we call `setNumber(7)`, where the number is
already `7`. In this case, the number doesn't actually change,
and so we would expect `timesChanged` to remain the same.

```java
    @Test
    void testAlreadyChangedSameNumber() {
        //Setup and configure test object
        NumberChanges nc = new NumberChanges(7, 4);

        //run operation to be tested
        nc.setNumber(7);

        //Check to ensure object behaved as specified
        assertEquals(7, nc.getNumber(), "Number changed when it should have stayed 7");
        assertEquals(4, nc.getTimesChanged(), "Number of times changed when it shouldn't have");
    }
```

The first thing to note is that we wrote this test **as a separate test**
from our first one. This is because we are testing two **different behaviors**.

* Our first test checks the behavior when the number changes as a result of `setNumber`
* Our second test checks the behavior when the number is not changed by `setNumber` 

We want to keep these tests separate, as it's possible that one test **fails**
while the second test **passes**. Depending on which test fails, the
kind of bug we would be looking for would change.

---

## Good testing rules

Just like any other skill, there are good practices and bad practices. The next section describes some common practices in writing unit tests.

### One operation per test

We can think of the function `setNumber` as an operation on an instance
of `NumberChanges`. Every time we call `setNumber`, it has the possibility
of changing the **state** of the instance it is called on.

As a rule, we only want to test **one operation at a time**. Why would
this be? Well, consider the following test with `MySortedList`.

```java
    @Test
    void addValueToMiddle() {
        //create starter List
        ArrayList<Integer> starterList = new ArrayList<>();
        starterList.add(1);
        starterList.add(3);
        starterList.add(5);

        MySortedList myList = new MySortedList(starterList);
        myList.add(4); //should be added between 3 and 5

        assertEquals(4, myList.size(), "Incorrect size of list");
        assertEquals(4, myList.get(2), "4 not added at the correct index");
        assertEquals(3, myList.get(1), "3 not immediately before 4");
        assertEquals(5, myList.get(3), "5 not immediately after 4");
    }
```

Here, the **one operation** test is `myList.add(4)`. We create
the object already in its starting state for this test by
passing in a starterList [1, 3, 5]. Then, we are simply adding 4.

Let's say instead we did the following:

```java

@Test
    void addLotsOfValues() {
        //create starter List
        ArrayList<Integer> starterList = new ArrayList<>();
        starterList.add(1);
        starterList.add(3);
        starterList.add(5);

        MySortedList myList = new MySortedList(starterList);
        myList.add(4); //should be added between 3 and 5
        myList.add(0); //should be added before 1
        myList.add(6); //should be added before 6
        myList.remove(3);
        myList.remove(1);
        

        assertEquals(4, myList.size(), "Incorrect size of list");
        assertEquals(4, myList.get(1), "4 not added at the correct index");
        assertEquals(0, myList.get(0), "0 not immediately before 4");
        assertEquals(5, myList.get(2), "5 not immediately after 4");
        assertEquals(6, myList.get(4), "6 not at the end of the list");
    }
}
```

Is this test better because it's testing more things? Here's another
question: is this test **Sound** (see below)? It's hard to tell because the test
is so complicated. This test is testing 3 different `add` operations
and 2 `remove` operations. It therefore is difficult to **understand**
this test, meaning if this test fails it won't be immediately clear why.

Just looking at the `remove` operations, what do they mean?
* Does `remove(3)` mean remove the value 3 from the list?
* Does `remove(3)` mean remove the **value at index 3** from the list?

This test does not make that clear!

Remember, we don't just want our tests to help us find bugs,
our test are also a crucial tool to help us communicate how
to use our code correctly! It is vital to be able
to read and understand tests.

---

### Test the interface, not the implementation

Always try to write your tests to be *interface* facing, not implementation facing. For example, when tested `MySortedList`, it is a good idea to design tests that are agnostic of the underlying ArrayList. Instead, focus on how the outputs of methods in the public interface change based on the test method call.

This idea of focusing on the interface over implementation, or abstraction, is a valuable design tool as well as a valuable testing tool. If implementation details change, such as changing the fields/structures to improve efficiency, but our tests only focus on the interface, we can often avoid needing to change our tests when the implementation changes.


---


### Ensure sound Tests

A **sound** test is one that correctly tests against the specification.
If a test is **unsound**, that means it is incorrect, and will
act as misinformation.

For example, in the above test, let's first consider:

`assertEquals(6, myList.get(4), "6 not at the end of the list");`

This is **unsound**. If you follow the tests, it's impossible for 
size to be equal to `4` AND the value `6` to be at index `4`. If
the class is correctly implemented, this test will throw an `IndexOutOfBoundsException`,
causing this test to **fail**. This type of failure is called a **false positive**.

Faulty tests can result in two types of errors:

__False Positive__: A test fails, indicating a defect, but there is no defect with the code  
__False Negative__: A test passes, but it shouldn't, as there is an underlying defect that the test fails to find

It may seem odd that a "False Positive" is a **fail**, not a **pass**. However
remember our goal in testing: **We are trying to find bugs!** As such,
**a test failing is a positive result** because it indicates the
presence of a defect. A test passing is a positive result because it
indicates that we don't need to look deeper for defects there.

**Unsound test** should be avoided **at all costs**. Whenever a test
fails, the first thing you should check is always to make sure
that the **failing test is sound.** Do not start debugging until
you have verified that the test is sound!

---


### One assertion per test?

There is a school of thought that says you should only have
one assert statement per test. For example, I would rewrite:

```java
    @Test
    void testAlreadyChangedSameNumber() {
        //Setup and configure test object
        NumberChanges nc = new NumberChanges(7, 4);

        //run operation to be tested
        nc.setNumber(7);

        //Check to ensure object behaved as specified
        assertEquals(7, nc.getNumber(), "Number changed when it should have stayed 7");
        assertEquals(4, nc.getTimesChanged(), "Number of times changed when it shouldn't have");
    }
```

As two separate tests:

```java
    @Test
    void testSetNumberUnchanged_Number() {
        NumberChanges n = new NumberChanges(5);
        n.setNumber(5);
        assertEquals(5, n.getNumber());
    }

    @Test
    void testSetNumberUnchanged_TimesChanged() {
        NumberChanges n = new NumberChanges(5);
        n.setNumber(5);
        assertEquals(0, n.getTimesChanged());
    }
```

There is a good reason behind this suggestion: imagine if both assertions...
* `assertEquals(5, n.getNumber());`
* `assertEquals(0, n.getTimesChanged());`

...**Fail**. Well, in the first case with only one test, I would
only see the *AssertionError* for the first assert statement: `assertEquals(5, n.getNumber());`.
However, in the second case with two tests, I would see **both** AssertionErrors. 

**That said, I do not follow this practice, nor do I 
generally recommend it**. My reason is that if
a test fails, I'm going to be debugging regardless, as so I will, in
the debugger, see the state of both fields of the object anyways. I also
dislike the 2nd approach because it encourages *copying and pasting* code which
is *always* a bad idea.

My biggest concern, however, is **readability**. I want a test to
be quickly readable, and I want to test to be a simple statement of
*how the specification describes the operation.* The first test
clearly states "this function can change both of these fields, but
in this particular setup it doesn't". Because it's all in one
test, I think of it as **a single operation that affects both fields.**
When I split them into two tests, the two test independently are less
**understandable**, because each of them tells only half the story.

To be clear, I am not saying one assertion per test is a bad idea.
However, I personally practice "One Operation per test" and find it
sufficient for my needs and leads to more understandable tests.

---


### Only call the tested operation **once**

```java
    @Test
    void testSetNumberChangedSeveralTimes() {
        NumberChanges n = new NumberChanges(5);
        n.setNumber(7);
        n.setNumber(13);
        n.setNumber(7);
        n.setNumber(13);
        n.setNumber(7);
        n.setNumber(13);
        n.setNumber(7);
        assertEquals(7, n.getNumber());
        assertEquals(7, n.getTimesChanged());
    }
```

Let's say the above test **failed**. If it did fail, **which
call of setNumber cause the failure?** Was it one call in
particular, or was it several? It becomes very hard to know. As
such, it's better to manually set the state of the object in a
controlled way that doesn't rely on the very methods you are trying
to test.

---

### Don't try to test private methods

Often, we will write private "helper" methods in our code for analyzability reasons. This is often done through an `extract method` call. These methods are useful. However, you shouldn't try to test them directly. Remember, we want to test how an object *behaves*, not how it's *implemented*, and private methods are an implementation detail.

If you find that a private-method (or set of private methods) in a particular procedure is complicated enough that you need to test it, it might be worth considering if those methods should be extracted to a separate class which can be tested independently.

It's worth noting that you can't call a private method directly in a test. This may make you think it's worth making a method `protected` for the sake of testing. But by introducing a `protected` method, you run the risk of complicating how your class is used by other classes in the same package.

In short, if you find that you need to test at a more precise level, that tells you that your class is likely doing too much or getting overly complicated, and decomposition might be a better choice.

---


## Setting up test objects

When we are setting up our test objects, you may notice we
can end up with redundant code. For example, consider
the following three tests for the `add` method in `MySortedListTest`:

```java
    @Test
    void addValueToMiddle() {
        //create starter List
        ArrayList<Integer> starterList = new ArrayList<>();
        starterList.add(1);
        starterList.add(3);
        starterList.add(5);

        MySortedList myList = new MySortedList(starterList);
        myList.add(4); //should be added between 3 and 5

        assertEquals(4, myList.size(), "Incorrect size of list");
        assertEquals(4, myList.get(2), "4 not added at the correct index");
        assertEquals(3, myList.get(1), "3 not immediately before 4");
        assertEquals(5, myList.get(3), "5 not immediately after 4");
    }

    @Test
    void addValueToBeginning() {
        //create starter List
        ArrayList<Integer> starterList = new ArrayList<>();
        starterList.add(1);
        starterList.add(3);
        starterList.add(5);

        MySortedList myList = new MySortedList(starterList);
        myList.add(0); //should add at the beginning of the list

        assertEquals(4, myList.size(), "Incorrect size of list");
        assertEquals(0, myList.get(0), "0 not added correctly at the beginning");
        assertEquals(1, myList.get(1), "1 not immediately after 0");
    }

    @Test
    void addValueToEnd() {
        //create starter List
        ArrayList<Integer> starterList = new ArrayList<>();
        starterList.add(1);
        starterList.add(3);
        starterList.add(5);

        MySortedList myList = new MySortedList(starterList);
        myList.add(6); //should be added at the end of the list

        assertEquals(4, myList.size(), "Incorrect size of list");
        assertEquals(6, myList.get(3), "6 not added correctly at the end");
        assertEquals(5, myList.get(2), "5 not immediately before 6");
    }
```

All three of these tests have the same setup steps:

```java
        //create starter List
        ArrayList<Integer> starterList = new ArrayList<>();
        starterList.add(1);
        starterList.add(3);
        starterList.add(5);

        MySortedList myList = new MySortedList(starterList);
```

Anytime you find yourself copying and pasting code like
this, you should stop! This is violating the **DRY Principle**:
*Don't Repeat Yourself.* Now, it makes sense that all three
of these tests use the same starting state. But because
they do, we can instead add creation of this object in one place.

First, since we're using this list in multiple tests, **let's make
it global**, that is, make it an instance variable of the test class!
I know this sounds bad, but stick with it for now:

```java
class MySortedListTest {

    MySortedList myAddTestList; //instance variable

    @Test
    void isEmptyTestInitiallyTrue() {
        MySortedList myList = new MySortedList();
        assertTrue(myList.isEmpty());
    }
    
    ...
}
```

Now, let's create a method that initializes this list to 
[1, 3, 5] like we do in each test:

```java
class MySortedListTest {

    MySortedList myAddTestList;
    
    public void setupMyAddTestList() {
        ArrayList<Integer> starterList = new ArrayList<>();
        starterList.add(1);
        starterList.add(3);
        starterList.add(5);

        myAddTestList = new MySortedList(starterList);
    }
```

We're almost there, but we have a problem! The tests `addValueToMiddle`,
`addValueToBeginning`, and `addValueToEnd` could run in **any order**. There
is no way to control the order they run in. Further, we want all
tests to be **independent.** That is, the results of one test
have no bearing on the results of another test. And so, we want
to re-run this method **before every test** to ensure that if any
test changes the state of `myAddTestList`, that the list is reset
before running the next test.

---


### @BeforeEach tag

```java
class MySortedListTest {

    MySortedList myAddTestList;
    
    @BeforeEach
    public void setupMyAddTestList() {
        ArrayList<Integer> starterList = new ArrayList<>();
        starterList.add(1);
        starterList.add(3);
        starterList.add(5);

        myAddTestList = new MySortedList(starterList);
    }
```

This is where the `@BeforeEach` tag comes into play. As the name
suggests, this tag tells JUnit to run the function `setupMyAddTestList`
**before each test**. This ensures that the state of the instance variable
`myAddTestList` is always the same at the start of every test, no matter
what tests we've run, or how many of them we have run. Before every
test, we ensure `myAddTestList` is a new `MySortedList` with values
[1, 3, 5] already added, and no other values added.

Now we simply need to update our tests that use this object: `addValueToMiddle`,
`addValueToBeginning`, and `addValueToEnd`. For the sake of space,
I'll only show the first one, but you can use that to see how
to change the other two:

```java
    @Test
    public void addValueToMiddle() {
        myAddTestList.add(4); //should be added between 3 and 5
        assertEquals(4, myAddTestList.size(), "Incorrect size of list");
        assertEquals(4, myAddTestList.get(2), "4 not added at the correct index");
        assertEquals(3, myAddTestList.get(1), "3 not immediately before 4");
        assertEquals(5, myAddTestList.get(3), "5 not immediately after 4");
    }
```

This allows us to create one or more test objects all in one place,
which reduces the repetition in our tests (and thus to desire
to copy and paste code).

---


## What methods to test?

In general, we want to test all `public` and `protected` methods in a given class
to ensure correct behavior. These methods describe the classes
**interface**. We want to ensure the class **interface** behaves
correctly.

In general, we do not test `private` methods. While it is
possible to call these methods using Java Reflections (more on this
below), we primarily want our tests to test **how the object behaves**,
not *how the object is implemented.*

The reason for this is that implementation details of the object
may change. For example, right now we are using an `ArrayList<Integer>`
to represent `MySortedList`. We also use **lazy-evaluation**, where
we only sort the list when we *have to*. This means if we add
10 values, and only then use the `get` operation, we only sort 1 time
rather than 10 times. This sorting is handled by the private
method `sortList()`, but from an external perspective, we don't even
need to know that this method exists. Instead, we can test the
behavior around `add()` using `get()`, and in doing so know that
the list is sorted when we are calling `get()`. Exactly how the
list *becomes* sorted isn't relevant to the client using the `MySortedList`
class.

Ultimately, we test to the **interface**, not the implementation, because
of what simple question: **What if the implementation changes?** What
if we find a better underlying data structure to use, or we optimize
the `ArrayList` interactions later on so they are thread-safe? Or
what if we change our sorting algorithm? Or we get rid of the
lazy evaluation and instead sort on each add?

If we are testing on `private` fields and methods, we will have
tests that prevent the code from changing. Instead, we want
our **implementation** to be as flexible as possible, and test
only the **interface** of our classes.

---


## How many tests do we need to write?

This is a tough question, and there is never a single correct
answer even in a well-defined function. For example, consider
our function max(int a, int b):

```java
    public static long max(long a, long b) {
        return (a + b + Math.abs(a - b)) / 2;
    }
```

How many tests do we need to write? Well, my first instinct would be:

* Let's write one test where `a` is bigger than `b`
* Let's write another test where `b` is bigger than `a`
* Let's write another test where `a` and `b` are equal

But is that sufficient? What is `a` and/or `b` is negative? Are you
**absolutely certain** that would work if the first 3 tests work?
I know I wasn't when I first came up with this method. So now I want
to test:

* Where `a` and `b` are both negative
* Where `a` is negative and `b` is positive
* Where `a` is positive and `b` is negative

Of course, as we mentioned before, this function won't work
with really big numbers. So I'll want to change this function
to alert people when their numbers are too big. So now I need
to consider combinations of 'a' and 'b' are both very far from
zero, either positive or negative, and check for an Exception...

You can see how our testing expectations can explode. And this is the point,
there is no perfect answer to the questions "How many tests should
I write". However, there are strategies we can use to determine
how many tests we should write depending on our level of uncertainty,
and that will be the focus of the next module: Test Plans

---


## Accessing private fields

During testing, there may be times we want to directly access
private fields. For example, in `MySortedListTest`, in order to
set up our test state with `MySortedList`, we are effectively 
"setting" the value of the private field `mySortedList` with
the constructor. However, what is this constructor were not there?
That is, what if we wanted to **remove this constructor**?

How could we configure the state of `MySortedList` without direct
access to the underlying `List` field or violating the rule "Only 
call the tested operation **once**" (meaning we can't call
`add` multiple times)? There are a few solutions:

### Add a getter and setter for the private field

If you want to modify and/or observe a private field, just
implement a getter and a setter. The only reason you might
want to avoid this is if you do not wish to add the getter
and setter methods to the public interface of the class. In fact,
very often we may explicitly not want to implement a public setter
method for a private field whose state is intended to be encapsulated
(that is, hidden) by our object. 

For example, `MySortedList`
is implemented in a way that it can be used by some client else without
having to know any details of how the class is **implemented**. The
client doesn't need to understand the lazy evaluation, for example.
However, having direct access to the underlying ArrayList would
mean the client would need to understand, for example, why the
underlying ArrayList isn't always sorted. Now interacting with
the class is more complicated, and that means correctly
using the class is harder.

### Make fields protected

The first would be to simply make mySortedList `protected` instead
of `private`. 

Making a variable or method `protected` means that any classes 
**in the same package** can treat the variable/method as though
it were *public*. Additionally, any class that *extends* a parent
class can access that parent's `protected` methods and fields as
though they were public.

However, to any class outside of the package that does not extend
the class, `protected` fields are treated as private.

This is the simplest solution, though there are drawbacks:
* By making something that was `private` instead `protected`, we
are changing the interface of the affected class. We run the risk
of developers mistakenly assuming they can access and use `protected`
field and method we **do not** want them to use. In short, this
violates the principle of **encapsulation**.
* By directly accessing fields and methods that were intended to
be `private`, our tests are more tightly coupled with **implementation**
details of the tested class. This means if the implementation changes, our
existing tests will likely break, and will have to be re-written, even
if the `public` **interface** does not change.

### Make an injectable Constructor

We can make a constructor that takes in all "state" fields of an object that we use for testing. 
This way, we still have this constructor
available for testing, which allows us to directly
configure the test object's starting state.

```java
public class MySortedList {
    private ArrayList<Integer> mySortedList;
    private boolean isSorted;
    
    protected MySortedList(ArrayList<Integer> mySortedList, boolean isSorted) {
        this.mySortedList = mySortedList;
        this.isSorted = isSorted;
    }
}
```

Now, we can simply directly instantiate objects into the state we wish to test them. Additionally, we can re-write our other constructors to avoid duplicate code:

```java
public class MySortedList {
    private ArrayList<Integer> mySortedList;
    private boolean isSorted;
    
    protected MySortedList(ArrayList<Integer> mySortedList, boolean isSorted) {
        this.mySortedList = mySortedList;
        this.isSorted = isSorted;
    }
    
    public MySortedList(ArrayList<Integer> mySortedList) {
        this(mySortedList, false);
    }
    
    public MySortedList() {
        this(new ArrayList<>(), true);
    }
}
```

This is a style I've adopted, and the advantage is that it lets me create tests directly via the constructor in one line of code:

```java
    var myTestSortedList = new MySortedList(new ArrayList<List.of(4, 5, 6), true);
```

However, this has many of the same pros and cons of making
a private field protected. It violates encapsulation, and creates a problem
of having a "special constructor" in the implementation that's
only used in the testing. In an ideal setting, the implementation
classes should not need to be aware of the classes that test them.
This means that the tests are tied not just to the interface, but the implementation, so if the implementation changes, the tests must necessarily change. Just like in other cases, the most reusable code will only use a public interface that hides implementation details. So you do face a trade-off with this approach.

### Testing with Reflections

One tool we can use instead in Java is Reflections. This approach can be complicated, and is *tightly coupled* to the implementation, and adds a significant testing overhead. I do not tend to use it, but it is worth being aware of.

### Mocking

Mocking is also a way to test a class operation while removing
any external dependencies. For example, in our tests, rather
than interacting with the underlying ArrayLists, we write our tests
so that we can "mock", or imitate, the behavior of the underlying
ArrayList. This technique is useful for doing unit testing on a class
that heavily relies on other classes, without having to
worry about doing full-on integration testing.

Like with Reflections, this topic is complicated enough to have its
own unit.