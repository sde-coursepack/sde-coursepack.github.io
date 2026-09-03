---
Title: Duplicate Code and Clean Testing
---


# Duplicate code

As we mentioned before, we want our code to be DRY (Don't Repeat Yourself) as opposed to WET (Write Everything Twice). This is especially true when writing tests!

* TOC
{:toc}



---

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
