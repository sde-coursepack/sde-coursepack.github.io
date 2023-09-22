---
Title: Duplicate Code and Clean Testing
---


# Duplicate code

As we mentioned before, we want our code to be DRY (Don't Repeat Yourself) as opposed to WET (Write Everything Twice). This is especially true when writing tests!

* TOC
{:toc}



---

## Clean Testing

__Starting Code Example:__ [LibraryTest.java](https://github.com/sde-coursepack/Library/blob/checkOut/src/test/java/LibraryTest.java)

Let's look at some tests we wrote back in the Test Driven Development unit.

First, let's look at our `Library class`. I only include method signatures of relavant methods for the sake of space.

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

Before we go on, I want to briefly highlight the value of *spacing*. Notice how I spaced out my instance variables into distinct groups. The first three variables, `testLibrary`, `testBookCopies`, and `testPatronList` all are need to initialize our `Library` test object. On the other hand, `gardensOfTheMoon` is a book that, while we may add it to the Library, is not necessary for instantiating `testLibrary`, so I use visual space to communicate it is a separate entity. Similarly, the `testPatron` object requires a `List<Book>` for keeping track of its checked out books. So I group `testPatron` and `testCheckOutList`, since I need the check-out list for creating my Patron.

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

a) We only want our tests to fail on their **own** conditions - outside factors should be removed from affecting our tests
b) We do not know what order our tests will run-in

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

Wow! Already our test is much easier to read. Before we commit this change, we want to make sure our test still passes. We run it, so we commit.

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

...and __AFTER___:

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

### We can be even DRY-er

Before we move on, let's look at our tests side-by-side:

```java
    @Test
    public void addBooksNewBooksTest() {
        testLibrary.addBooks(gardensOfTheMoon, 2); // add copies of a new book

        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book not added to Map");
        assertEquals(2, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies added");
    }

    @Test
    public void addBooksExistingBooksTest() {
        testBookCopies.put(gardensOfTheMoon, 2); // add existing book

        testLibrary.addBooks(gardensOfTheMoon, 2); // add new copies of existing book

        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book no longer in Map");
        assertEquals(4, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies after add");
    }
```

For now, focus on the two `assert` statements on the last two lines. These assert statements verify:

* that the book we added is now (or still) a key in our `testLibrary`'s `bookCopies` HashMap
* that the Library has as many copies of the book as we would expect

That is, two lines of code implement the same **idea** in both test methods. This is WET code, so let's get out our "Refactor -> Extract Method" towels to dry it off!

### Extract duplicate asserts to method

We start by simply extracting the method from our first test. Highlight the two lines, right-click->Refactor->Extract Method, and name it `assertLibraryHasNCopiesOfBook()`:

```java
    @Test
    public void addBooksNewBooksTest() {
        testLibrary.addBooks(gardensOfTheMoon, 2); // add copies of a new book

        assertLibraryHasNCopiesOfBook();
    }

    private void assertLibraryHasNCopiesOfBook() {
        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book not added to Map");
        assertEquals(2, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies added");
    }
```
We want to call our function `assert_________` where we fill in the blank with something meaningful. This is because this private helper function is effectively wrapping up our existing assert statements in a function. Starting the function name with `assert` clearly communicates to the reader "Hey, these are our test assertions!".

Notice we do *not* include a @Test tag. That's because *this isn't a test method*. This is a method **used** by our tests (a helper method).

#### Slow down a second...

Right now, our function is hard-coded with only the values used in our first test.

Before we proceed, we now need to stop and think:
* Will I ever use this function again?
    * We know the answer is yes, because we intend to incorporate it into our second `addBooks` test
* If so, do I need to change the arguments?
    * Yes! We obviously don't want to hardcode the number 2 for every time this function is called.

Obviously, we want to include the number of copies we are expecting (in this case, `2`) in our arguments, since in our next test this number of copies is different (`4`). Additionally, it seems reasonable that in the future, we may be testing with more books than just `gardensOfTheMoon` as we evolve our software (for instance, we may have other Malazan books like `deadhouseGates` and `memoriesOfIce`). Since this *could* vary, we should also include the `Book` we are checking in arguments.

So, we update our new method:

```java
    @Test
    public void addBooksNewBooksTest() {
        testLibrary.addBooks(gardensOfTheMoon, 2); // add copies of a new book

        assertLibraryHasNCopiesOfBook(gardensOfTheMoon,2);
    }

    private void assertLibraryHasNCopiesOfBook(Book book, int numberOfCopies) {
        assertTrue(testBookCopies.containsKey(book), "Test book not added to Map");
        assertEquals(numberOfCopies, testBookCopies.get(book), "Incorrect number of copies added");
    }
```

We run our test to ensure it still passes (since we've done a lot of code changes). It does, so we commit.

### Reuse our extracted function

Now we can apply that change to our second test.

Once again, __BEFORE__
```java
    @Test
    public void addBooksNewBooksTest() {
        testLibrary.addBooks(gardensOfTheMoon, 2); // add copies of a new book

        assertLibraryHasNCopiesOfBook(gardensOfTheMoon,2);
    }

    private void assertLibraryHasNCopiesOfBook(Book book, int numberOfCopies) {
        assertTrue(testBookCopies.containsKey(book), "Test book not added to Map");
        assertEquals(numberOfCopies, testBookCopies.get(book), "Incorrect number of copies added");
    }

    @Test
    public void addBooksExistingBooksTest() {
        testBookCopies.put(gardensOfTheMoon, 2); // add existing book

        testLibrary.addBooks(gardensOfTheMoon, 2); // add new copies of existing book

        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book no longer in Map");
        assertEquals(4, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies after add");
    }

```

... and __AFTER__

```java
    @Test
    public void addBooksNewBooksTest() {
        testLibrary.addBooks(gardensOfTheMoon, 2); // add copies of a new book

        assertLibraryHasNCopiesOfBook(gardensOfTheMoon,2);
    }


    @Test
    public void addBooksExistingBooksTest() {
        testBookCopies.put(gardensOfTheMoon, 2); // add existing book

        testLibrary.addBooks(gardensOfTheMoon, 2); // add new copies of existing book

        assertLibraryHasNCopiesOfBook(gardensOfTheMoon,4);
    }

    private void assertLibraryHasNCopiesOfBook(Book book, int numberOfCopies) {
        assertTrue(testBookCopies.containsKey(book), "Test book not added to Map");
        assertEquals(numberOfCopies, testBookCopies.get(book), "Incorrect number of copies added");
    }
```

You'll note I moved the helper function down below our second test. As a *general* (but not strict) rule, we design Java classes to be read top-to-bottom. So we want the high-level functions first, and the low-level functions that are called by the high-level functions second.

---

### Repeating for our other tests

As you'll recall, I also wrote a number of tests for the `Library` method `checkOut()`. Here is an example of one:

```java
    @Test
    public void checkOutEquivalenceTest() {
        Map<Book, Integer> testBookCopies = new HashMap<>();
        List<Patron> testPatronList = new ArrayList<>();
        Book gardensOfTheMoon = new Book(1,
                "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
                "Steven Erikson");
        List<Book> patronCheckedOut = new ArrayList<>();
        Patron testPatron = new Patron(12, "John", "Smith", patronCheckedOut);
        Library testLibrary = new Library(testBookCopies, testPatronList);
        testBookCopies.put(gardensOfTheMoon, 2);
        testPatronList.add(testPatron);

        testLibrary.checkOut(testPatron, gardensOfTheMoon);
        assertEquals(1, testBookCopies.get(gardensOfTheMoon),
                "Library has wrong number of copies of test book");
        assertTrue(patronCheckedOut.contains(gardensOfTheMoon),
                "Patron doesn't have test book in their checked out list");
        assertEquals(1, patronCheckedOut.size(), "Patron doesn't have right number of books checked out");
    }
```

Wow, this test is **huge**! It's even bigger than the last tests we cleaned up. However, we can apply the same process here. I'm not going to show every single step like before, but here is my "cleaned-up" version:


```java
    @Test
    public void checkOutEquivalenceTest() {
        givenBookCopyEntryToTestLibraryMap(gardensOfTheMoon, 2);
        givenPatronEnrolledInTestLibrary(testPatron);

        testLibrary.checkOut(testPatron, gardensOfTheMoon);

        assertTestLibraryHasNCopiesOfBook(gardensOfTheMoon, 1);
        assertCheckOutListEqualsListOf(testCheckOutList, gardensOfTheMoon);
    }

    private void givenBookCopyEntryInTestLibraryMap(Book book, int value) {
        testBookCopies.put(book, value);
    }

    private void givenPatronEnrolledInTestLibrary(Patron testPatron) {
        testPatronList.add(testPatron);
    }

    private void assertTestLibraryHasNCopiesOfBook(Book book, int numberOfCopies) {
        assertTrue(testBookCopies.containsKey(book), "Test book not added to Map: " + book);
        assertEquals(numberOfCopies, testBookCopies.get(book), "Incorrect number of copies of " + book + " present");
    }

    private void assertTestLibraryDoesNotCarryBook(Book book) {
        assertFalse(testBookCopies.containsKey(book), "Library should not be carrying " + book);
    }

    private void assertCheckOutListEqualsListOf(List<Book> testCheckOutList, Book... books) {
        List<Book> expected = convertArrayToArrayList(books);
        assertEquals(expected, testCheckOutList, "Check Out List is not as expected.");
    }

    private ArrayList<Book> convertArrayToArrayList(Book[] bookArray) {
        List<Book> bookList = Arrays.asList(bookArray);
        return new ArrayList<>(bookList);
    }
```

#### Given

In general, we describe tests as "Given [starting state], when [operation performed], then [assert statements]". I use this `given-when-then` nomenclature to name my functions. Just like I named my post-condition checking functions `assert______`, I like to name my *setup functions* `given______`, as it helps clarify the different roles they play. This is a style I have adopted, and not one that you should strictly expect to be followed everywhere. However, I find it helps me keep my tests clean and readable.

#### `Book... books` aka varargs

If you aren't familiar with the syntax of `assertCheckOutListEquals(List<Book> testCheckOutList, Book... books)`, this is an example of `varargs` (variable-length arguments). This allows me to call the function assertCheckOutList with multiple books if I want to test the `testCheckOutList` value having more than one book. For example:

`assertCheckOutListEquals(testCheckOutList, gardensOfTheMoon, deadhouseGates, memoriesOfIce)`

The key is that `varargs` then combines all 3 of these into an array of Book objects (`Book[]`), which I then convert to an `ArrayList<Book>` (`convertArrayToArrayList(Book[] bookArray)`). From there, I can just use `assertEquals` to determine if I my actual checkOutList matches my expected list of books from varargs. This is useful if we want to handle an indeterminate number of arguments dynamically.

---

## Can be we **too** DRY

Consider our two `addBooks` test cases.

```java
    @Test
    public void addBooksNewBooksTest() {
        testLibrary.addBooks(gardensOfTheMoon, 2); // add copies of a new book

        assertTestLibraryHasNCopiesOfBook(gardensOfTheMoon, 2);
    }

    @Test
    public void addBooksExistingBooksTest() {
        givenBookCopyEntryToTestLibraryMap(gardensOfTheMoon, 2);

        testLibrary.addBooks(gardensOfTheMoon, 2);

        assertTestLibraryHasNCopiesOfBook(gardensOfTheMoon, 4);
    }
```

Some of you may note that both tests contain the line:

`testLibrary.addBooks(gardensOfTheMoon, 2);`

Should I extract that as a function? Something like:

```java
    public void addBookCopiesToTestLibrary(Book book, int copies) {
        `testLibrary.addBooks(book, copies);
    }
```

The question I would ask is this: does this make our tests **more understandable**? In this case, I would argue no. Remember, our tests generally have 3 steps:

1) Set up the test state for the test Object(s) (`given`)
2) Execute the Method Under Test (in this case, `testLibrary.addBooks`)
3) Check the post-conditions to ensure correct behavior (`assert`)

You'll notice all of my tests have extra line breaks between each step to improve readability.

If I "hide" `addBooks` in a private helper method, it because immediately less obvious what the Method Under Test is for our two tests. Additionally, because these *are* tests, I fully expect that if the interface or behavior of `addBooks` changes, I will necessarily have to update all of my tests for `addBooks` anyways, so I'm not saving future effort.

In short, I don't see value in extracting out that repetitive line of code, since it serves a vital purpose in the **understandability** of the test.

## DRY Testing Conclusion

Consider the before and after of our `TestLibrary` class:

[BEFORE](https://github.com/sde-coursepack/Library/blob/checkOut/src/test/java/LibraryTest.java)

[AFTER](https://github.com/sde-coursepack/Library/blob/testCleanUp/src/test/java/LibraryTest.java)

If any of you told me the Tests in the **Before** case are more readable, I would be a mix of very surprised and extremely suspicious that you are lying to me. In fact, I'm so confident that my tests are now "well written prose" that I can feel confident showing you this test I just wrote:

```java
    @Test
    public void testCheckOutWhenPatronAlreadyHasACopyTest() {
        givenBookCopyEntryInTestLibraryMap(gardensOfTheMoon, 2);
        givenPatronEnrolledInTestLibrary(testPatron);
        givenCheckOutListContainsBooks(testCheckOutList, gardensOfTheMoon);

        assertThrows(RuntimeException.class, () ->
            testLibrary.checkOut(testPatron, gardensOfTheMoon));

        assertTestLibraryHasNCopiesOfBook(gardensOfTheMoon, 2);
        assertCheckOutListEqualsListOf(testCheckOutList, gardensOfTheMoon);
    }
```

And I'm confident you can explain:
* What the test is testing
* How the setup works (including the new function `givenCheckOutListIsListOf` with isn't shown)
* What it means if the test fails (that is, what feature isn't working correctly)

This is what we mean by "code that reads like well-written prose", and it's just important in tests as it is in our code.

### Never go to far

Ultimately, we need are tests to be readable, understandable, and most of all simple. We want to avoid complicated logic in our test code at all costs after all, if we have complicated logic, then we have to tests the tests, and at that point we're on a track towards unnecessary testing.

Never make the goal "the tests should be short". The goal should always be "the tests should be easy to understand at a glance." Length plays a role there, but the deeper you have to dive into a test to understand it, the worse it becomes. **Tests should be as declarative as possible!**

### That test cleanup was a lot of work

Yes it was! And it took a lot of time, too! But the reason it did was because **I didn't do it right the first time**. This is what we mean by technical debt. If I had started, when writing my first test, creating fields that are initialized in @BeforeEach, extracting methods as I went, **I would have saved time in the long run**. I can tell you that writing the dirty tests we started with took me a long time, and I made several mistakes that resulted in un-sound tests along the way. I wasted a lot of time writing and trying to fix dirty code!

However, in order for me to continue being able to efficiently practice TDD in my `Library` class, I needed to stop and clean-up my `LibraryTest` class. If I didn't let it get so dirty in the first place, I would have saved time! It's easier to keep a clean class clean than it is to clean up a dirty class. Remember:

>**"The only way to go fast is to go well"**

## Conclusion

I showed multiple code refactors built around extract methods here. While I focused on refactoring out duplication in a test class here, I would use this same process in any non-test class. Again, most of the refactoring I did was extracting methods, extracting fields, and renaming to improve readability. Using a good IDE like IntelliJ, this process is actually very quick once you get practice with it.
