---
Title: Duplicate Code and Clean Testing
---


# Duplicate code

As we mentioned before, we want our code to be DRY (Don't Repeat Yourself) as opposed to WET (Write Everything Twice). This is especially true when writing tests!

* TOC
{:toc}



---

## FizzBuzz

One of the most classic programming interview questions is FizzBuzz. Typically, it will be described as something like:

> Loop through every number from 1-100 and do the following:
>   * If the number is divisible by 3, print "Fizz"
>   * If the number is divisible by 5, print "Buzz"
>   * If the number is divisible by both, print "FizzBuzz"
>   * Otherwise, print the number as is

An implementation of this idea in Java might look like:

```java
public class FizzBuzz {
    static final int LOOP_START = 1;
    static final int LOOP_END = 100;

    public static void main(String[] args) {
        for (int i = LOOP_START; i <= LOOP_END; i++) {
            if (i % 3 == 0 && i % 5 == 0) {
                System.out.println("FizzBuzz");
            } else if (i % 3 == 0) {
                System.out.println("Fizz");
            } else if (i % 5 == 0) {
                System.out.println("Buzz");
            } else {
                System.out.println(i);
            }
        }
    }
}
```

Specifically, the order of the if statements (checking 3 and 5 first) is important for this code to work correctly. The "Fizz" and "Buzz" statements can be in either order, but "FizzBuzz" must come first.

Now, in general, the point of the FizzBuzz question isn't really figuring out the above, but rather how you respond to questions about your solution, and how your code changes overtime.

For example, let's say the interviewer asks:

> Now, let's say that for numbers divisible by 7, you should print Bang. 3 and 7 should be FizzBang. 5 and 7 should be BuzzBang. A number divisible by all of 3, 5, and 7 should be FizzBuzzBang. To test more thoroughly (since 3*5*7 = 105, which is greater than 100), loop to 1000 instead.

Well, you **could** try to approach this in a *brute force* sort of way. Let's first add *just* the 7 case, so now the loop is:

```java
for (int i = LOOP_START; i <= LOOP_END; i++) {
    if (i % 3 == 0 && i % 5 == 0) {
        System.out.println("FizzBuzz");
    } else if (i % 3 == 0) {
        System.out.println("Fizz");
    } else if (i % 5 == 0) {
        System.out.println("Buzz");
    } else if (i % 7 == 0) {
        System.out.println("Bang");
    } else {
        System.out.println(i);
    }
}
```

And then updated `LOOP_END` to be 1000. But...now we need to add 3 more if statements, 3 and 7, 5 and 7, and 3, 5, and 7. We **could** do this, but then...well... this is what we get:

```java
public class FizzBuzz {
    static final int LOOP_START = 1;
    static final int LOOP_END = 1000;

    public static void main(String[] args) {
        for (int i = LOOP_START; i <= LOOP_END; i++) {
            if (i % 3 == 0 && i % 5 == 0 && i % 7 == 0) {
                System.out.println("FizzBuzzBang");
            } else if (i % 3 == 0 && i % 5 == 0) {
                System.out.println("FizzBuzz");
            } else if (i % 3 == 0 && i % 7 == 0) {
                System.out.println("FizzBang");
            } else if (i % 5 == 0 && i % 7 == 0) {
                System.out.println("BuzzBang");
            } else if (i % 3 == 0) {
                System.out.println("Fizz");
            } else if (i % 5 == 0) {
                System.out.println("Buzz");
            } else if (i % 7 == 0) {
                System.out.println("Bang");
            } else {
                System.out.println(i);
            }
        }
    }
}
```

First, this definitely is going to lead to me copying-pasting the else-if statement and just changing individual values (in fact, to write the above code, I did *exactly* that!), and any time you start doing this, an alarm should go off in your brain for two reasons:

1. Copying and pasting code that you then edit slightly leads to a large number of silly bugs due to forgetting to change a value, or getting some other minor detail wrong, and that debugging typically takes longer than typing the code
2. You have a duplicate code block!

The second problem? Any interviewer will then ask:

> Okay, now if the number is divisible by 11, print "Boing". Similarly, combine Boing with the other numbers. For example, 77 would be "BangBoing". To test more thoroughly, we'll now loop from 1 to 10,000.

You **really** shouldn't use the same strategy as before. At this point, however, if you're clever, you might notice that we can actually do this with a loop. 

## Cleaning up Tests

__Starting Code Example:__ [LibraryTest.java](https://github.com/sde-coursepack/Library/blob/checkOut/src/test/java/LibraryTest.java)

Let's look at some tests we wrote back in the Test Driven Development unit.

First, let's look at our `Library class`. I only include method signatures of relevant methods for the sake of space.

```java

public class Library {
    public static final int MAX_BOOKS_PER_PATRON = 3;

    private Map<Book, Integer> bookCopies;
    private List<Patron> patrons;

    public Library(Map<Book, Integer> bookCopies, List<Patron> patrons) {
        this.bookCopies = bookCopies;
        this.patrons = patrons;
    }

    public int getNumCopies(Book b) { ... }

    public void addBooks(Book b, int copies) { ... }

    public void checkOut(Patron p, Book b) { ... }
```

```java
public class LibraryTest {
    @Test
    public void addBooksNewBooksTest() {
        Map<Book, Integer> testBookCopies = new HashMap<>();
        List<Patron> testPatronList = new ArrayList<>();
        Library testLibrary = new Library(testBookCopies, patronList);
        Book gardensOfTheMoon = new Book(1,
                "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
                "Steven Erikson");

        testLibrary.addBooks(gardensOfTheMoon, 2);

        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book not added to Map");
        assertEquals(2, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies after add");
    }

    @Test
    public void addBooksExistingBooksTest() {
        Map<Book, Integer> testBookCopies = new HashMap<>();
        List<Patron> testPatronList = new ArrayList<>();
        Book gardensOfTheMoon = new Book(1,
                "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
                "Steven Erikson");
        Library testLibrary = new Library(testBookCopies, patronList);
        testBookCopies.put(gardensOfTheMoon, 2);

        testLibrary.addBooks(gardensOfTheMoon, 2);
        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book no longer in Map");
        assertEquals(4, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies after add");
    }
}
```

Notice how hard it is to read these tests! This is bad, because we need to make sure that we can **understand** our tests such that if they fail we know what specific feature and conditions they were testing.

### Reducing repetitive test object setup

Notice how repetitive these tests look! In fact, the setup for the two tests is nearly identical:

* We create an input HashMap for modeling `bookCopies`
* We create an empty Patron list (since this test doesn't require patrons)
* We create an identical book, `gardensOfTheMoon`
* We create our test library using our `testBookCopies` and `testPatronList` collections
* We call the same test function: `testLibrary.addBooks(gardensOfTheMoon, 2);`
* We have *nearly* the same assert statements, with only the specific value different (2 in the first test, 4 in the second).

This isn't DRY at all!

Since in nearly every test (even for other functions), we can expect to use:
* A library
* A HashMap for `testBookCopies`
* An ArrayList for `testPatronList`
* Usually at least one book
* Usually at least one patron
* That patron has an existing list of checked out books

...we can define these test objects at the class level as instance variables!

```java
public class LibraryTest {
    private Library testLibrary;
    private Map<Book, Integer> testBookCopies;
    private List<Patron> testPatronList;

    private Book gardensOfTheMoon;

    private Patron testPatron;
    private List<Book> testCheckOutList;
    
    ...
```

Before we go on, I want to briefly highlight the value of *spacing*. Notice how I spaced out my instance variables into distinct groups. The first three variables, `testLibrary`, `testBookCopies`, and `testPatronList` all are need to initialize our `Library` test object. On the other hand, `gardensOfTheMoon` is a book that, while we may add it to the Library, is not necessary for instantiating `testLibrary`, so I use visual space to communicate it as a separate entity. Similarly, the `testPatron` object requires a `List<Book>` for keeping track of its checked out books. So, I group `testPatron` and `testCheckOutList`, since I need the check-out list for creating my Patron.

### @BeforeEach for test setup

And then, we can **re-initialize** each value at the start of every test using the JUnit tag @BeforeEach

```java
    @BeforeEach
    public void setupDefaultTestObjects() {
        testBookCopies = new HashMap<>();
        testPatronList = new ArrayList<>();
        testLibrary = new Library(testBookCopies, testPatronList);

        gardensOfTheMoon = new Book(1,"Gardens Of The Moon: Book 1 of Malazan Book of the Fallen", "Steven Erikson");

        testCheckOutList = new ArrayList<>();
        testPatron = new Patron(12, "John", "Smith", patronCheckedOut);
    }
```

The @BeforeEach tag tells JUnit to run this function **before each test**. We need to do this because we want to make sure every test run is independent; that is, no single test affects any other. This is important because:

1. We only want our tests to fail on their **own** conditions - outside factors should be removed from affecting our tests 
2. We do not know what order our tests will run-in

You might be worried that all we're doing is adding a blank list of patrons, a blank map of books-to-copies entries, etc. However, remember: **all of these types are mutable**. What this means is even **after** I initialize `testLibrary` on the third line of my setup, I can **at any time** add values to either `patronList` or `testBookCopies` directly using the `add` and `put` methods respectively, and it will affect the state of `testLibrary`. We will use this to our advantage to make our tests much shorter and cleaner.

### Rewriting a test

Now that we have these fields that are already initialized, let's start cleaning up our code. Starting with our first dirty test:

```java
@Test
    public void addBooksNewBooksTest() {
        Map<Book, Integer> testBookCopies = new HashMap<>();
        List<Patron> testPatronList = new ArrayList<>();
        Library testLibrary = new Library(testBookCopies, testPatronList);
        Book gardensOfTheMoon = new Book(1,
                "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
                "Steven Erikson");

        testLibrary.addBooks(gardensOfTheMoon, 2);

        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book not added to Map");
        assertEquals(2, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies added");
    }
```

Since we now have instance already initialized variables for `testLibrary`, `testBookCopies`, `testPatronList`, and `gardensOfTheMoon`, we can simply **remove** the first 4 lines:

```java
@Test
    public void addBooksNewBooksTest() {
        testLibrary.addBooks(gardensOfTheMoon, 2);

        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book not added to Map");
        assertEquals(2, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies added");
    }
```

Wow! Already our test is much easier to read. Before we commit this change, we want to make sure our test still passes. We run it, and it passes, so we commit.

Now we do the same with our second `addBooks()` test.

__BEFORE__:

```java
    @Test
    public void addBooksExistingBooksTest() {
        Map<Book, Integer> testBookCopies = new HashMap<>();
        List<Patron> testPatronList = new ArrayList<>();
        Book gardensOfTheMoon = new Book(1,
                "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
                "Steven Erikson");
        Library testLibrary = new Library(testBookCopies, testPatronList);
        testBookCopies.put(gardensOfTheMoon, 2);

        testLibrary.addBooks(gardensOfTheMoon, 2);
        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book no longer in Map");
        assertEquals(4, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies after add");
    }
```

...and __AFTER__:

```java
    @Test
    public void addBooksExistingBooksTest() {
        testBookCopies.put(gardensOfTheMoon, 2); // add existing book

        testLibrary.addBooks(gardensOfTheMoon, 2); // add new copies
        
        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book no longer in Map");
        assertEquals(4, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies after add");
    }
```

...and once against our test passes! So we commit!

## Conclusion

I showed multiple code refactors built around extracting duplicate code here. Most of the refactoring I did was extracting methods, extracting fields, and renaming to improve readability. Using a good IDE like IntelliJ, this process is actually very quick once you get practice with it.
