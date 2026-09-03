---
Title: Duplicate Code and Clean Testing
---


# Duplicate code

As we mentioned before, we want our code to be DRY (Don't Repeat Yourself) as opposed to WET (Write Everything Twice). This is especially true when writing tests!

* TOC
{:toc}



---

## PBB Matching problem

Let's say we need to write code that takes in a `String` via standard-input (`stdin`) and checks if the parentheses, braces, and brackets ("binding") are *matched*. This could be used, for instance, to validate any type of nested data structure is valid. Examples:

* `([]{})` - this **is** matched, because every left-binding is matched with a right-binding in the same scope
* `[cat: {'color':'orange'}]` is also matched, since we ignore any non-binding characters.
* `(){]` - this is **not** matched, because the left-curly brace `{` has no matching right curly-brace, and similarly the right bracket `]` has no matching left brace
* `([)]` - this is **not** matched. While every left-binding symbol has a matching right-binding symbol, the symbols are not in the same scope, and so this is invalid.

A solution could look like:

```java
import java.util.Scanner;
import java.util.Stack;

public class PBBMatching {
    static void main(String[] args) {
        Scanner s = new Scanner(System.in);
        String input = s.nextLine();
        s.close();
        var arr = input.toCharArray();
        var stack = new Stack<Character>();
        String result = "Yes";
        for (char c : arr) {
            if (c == '(' || c == '{' || c == '[') {
                stack.push(c);
            } else if (c == ')') {
                if (stack.isEmpty() || stack.pop() != '(') {
                    result = "No";
                    break;
                }
            } else if (c == ']') {
                if (stack.isEmpty() || stack.pop() != '[') {
                    result = "No";
                    break;
                }
            } else if (c == '}') {
                if (stack.isEmpty() || stack.pop() != '{') {
                    result = "No";
                    break;
                }
            }
        }
        if (!stack.isEmpty()) {
            result = "No";
        }
        System.out.println(result);
    }
}
```
Now, so far, we have a single giant main method. Let's clean that up a bit.

### Extract stdin reading

First, I'll extract getting the first line from standard input to a function:

```java
public static String getStdinLine() {
  Scanner s = new Scanner(System.in);
  String input = s.nextLine();
  s.close();
  return input;
}
```


### Extract matching loop
Similarly, I can note that the String `result` is functioning more like a `boolean` than a String, and that the primary point of my `for` loop over the characters in the String is just to determine that `boolean`. So let's extract the loop as a `boolean` function.

To do this, first, we'll remove the line:

```String result = "Yes";```

And change, inside our loop 2-4th if conditions, we'll remove:

```java
result = "No";
break;
```

... and replace it with ...

```return false;```

And then replace:

```java
if (!stack.isEmpty()) {
    result = "No";
}
```

...with the simpler...

```java
return stack.isEmpty();
```

So now our extracted function looks like:

```java
public static boolean isBindingMatched(String s) {
    var arr = s.toCharArray();
    var stack = new Stack<Character>();
    for (char c : arr) {
        if (c == '(' || c == '{' || c == '[') {
            stack.push(c);
        } else if (c == ')') {
            if (stack.isEmpty() || stack.pop() != '(') {
                return false;
            }
        } else if (c == ']') {
            if (stack.isEmpty() || stack.pop() != '[') {
                return false;
            }
        } else if (c == '}') {
            if (stack.isEmpty() || stack.pop() != '{') {
                return false;
            }
        }
    }
    return stack.isEmpty();
}
```

### `main` function

Now our 'main' function looks like:

```java
static void main(String[] args) {
    String input = getStdinLine();
    String result = isBindingMatched(input) ? "Yes" : "No";
    System.out.println(result);
}
```

Quick aside, `isBindingMatched(input) ? "Yes" : "No";` is a **ternary** expression. The structure is: 

`value =` *boolean expression* `?` *[value-if-true]* `:` *[value-if-false]*

Or, written out another way, this could be:

```java
String result = "";
if (isBindingMatched(input)) {
    result = "Yes";
} else {
    result = "No"
}
```

The ternary expression just gives us a more concise way to express this idea.

### Cleaning up `isBindingMatched`

When looking at the `isBindingMatched`, you probably noticed it was a fairly large function, but also the last several else-if conditions are nearly identical, with only the *literal* values changed to match particular bindings. This is a sign of duplicate code.

The problem with this is that this code can keep growing and growing. For instance, what if we want to ensure we also match `<` and `>` braces? Now, we would have to make two changes to this code:

First, updated the first if block in the loop:

`if (c == '(' || c == '{' || c == '[' || c == '<') {`

Hopefully you can see that if we need to support additional "left-symbols", this line can keep growing and get harder to read.

Second, we need to add another `else if` block:

```java
else if (c == '>') {
    if (stack.isEmpty() || stack.pop() != '<') {
        return false;
    }
}
```

...which again is nearly identical code. 

So first, let's recognize there are, broadly, one of three things that happen in our for loop:

1. If `ch` is a left-binding, we push it onto our `stack`
2. If `ch` is a right-binding, we check if this right-binding matches the left-binding on top of the `stack`, returning false immediately if the `stack` is empty or the top value doesn't match.
3. If `ch` is neither a left- nor right-binding, we do nothing, since it doesn't affect the matching status.

Using this, let's reset a bit. Let's introduce two class-level constants:

```java
public class PBBMatching {
    static final String LEFT_BINDINGS = "([{";
    static final String RIGHT_BINDINGS = ")]}";
    ...
}
```

I intentionally built these constants so that, for any index `i`, `LEFT_BINDINGS[i]` and `RIGHT_BINDINGS[i]` are matching bindings: `0` for parentheses, `1` for brackets, `2` for curly-braces.

Now, using this, we can check introduce a couple simple well-named functions:

```java
public static boolean isLeftBinding(char ch) {
    return LEFT_BINDINGS.indexOf(ch) >= 0;
}

public static boolean isRightBinding(char ch) {
    return RIGHT_BINDINGS.indexOf(ch) >= 0;
}
```

These two functions check if a given symbol is a left or right binding. From there, we need a function that, given a right-binding, returns the matching left-binding.


```java
public static char getLeftBinding(char rightBinding) {
    if (!isRightBinding(rightBinding)) {
        throw new IllegalArgumentException(
                "Error: " + rightBinding + " is not a valid right binding"
        );
    }
    int index = RIGHT_BINDINGS.indexOf(rightBinding);
    return LEFT_BINDINGS.charAt(index);
}
```

As we talked about with defensive programming, we check our guard conditions (that is, our input must actually be a right-binding for the function to make sense) up front. From there, we just get the matching left-binding for our right-binding.

Using this, we can rewrite our for-loop in a much simpler way. First, we can change this:

```java
if (c == '(' || c == '{' || c == '[') {
    stack.push(c);
}
```

Into this: 

```java
if (isLeftBinding(c)) {
    stack.push(c);
}
```

And then we can replace all of our else-if branches:

```java
else if (c == ')') {
    if (stack.isEmpty() || stack.pop() != '(') {
        return false;
    }
} else if (c == ']') {
    if (stack.isEmpty() || stack.pop() != '[') {
        return false;
    }
} else if (c == '}') {
    if (stack.isEmpty() || stack.pop() != '{') {
        return false;
    }
}
```

...with a single `else-if`...

```java
else if (isRightBinding(c)) {
    if (stack.isEmpty() || stack.pop() != getLeftBinding(c)) {
        return false;
    }
}
```

And so now our overall `isBindingMatched` function is:

```java
public static boolean isBindingMatched(String s) {
    var arr = s.toCharArray();
    var stack = new Stack<Character>();
    for (char c : arr) {
        if (isLeftBinding(c)) {
            stack.push(c);
        } else if (isRightRinding(c)) {
            if (stack.isEmpty() || stack.pop() != getLeftBinding(c)) {
                return false;
            }
        }
    }
    return stack.isEmpty();
}
```

#### Final code

```java
import java.util.Scanner;
import java.util.Stack;

public class PBBMatching {
    static final String LEFT_BINDINGS = "([{";
    static final String RIGHT_BINDINGS = ")]}";

    public static void main(String[] args) {
        String input = getStdinLine();
        String result = isBindingMatched(input) ? "Yes" : "No";
        System.out.println(result);
    }

    public static String getStdinLine() {
        Scanner scanner = new Scanner(System.in);
        String input = scanner.nextLine();
        scanner.close();
        return input;
    }

    public static boolean isLeftBinding(char ch) {
        return LEFT_BINDINGS.indexOf(ch) >= 0;
    }

    public static boolean isRightBinding(char ch) {
        return RIGHT_BINDINGS.indexOf(ch) >= 0;
    }

    public static char getLeftBinding(char rightBinding) {
        if (!isRightBinding(rightBinding)) {
            throw new IllegalArgumentException(
                    "Error: " + rightBinding + " is not a valid right binding"
            );
        }
        int index = RIGHT_BINDINGS.indexOf(rightBinding);
        return LEFT_BINDINGS.charAt(index);
    }

    public static boolean isBindingMatched(String s) {
        var arr = s.toCharArray();
        var stack = new Stack<Character>();
        for (char c : arr) {
            if (isLeftBinding(c)) {
                stack.push(c);
            } else if (isRightBinding(c)) {
                if (stack.isEmpty() || stack.pop() != getLeftBinding(c)) {
                    return false;
                }
            }
        }
        return stack.isEmpty();
    }
}
```

#### Benefit

First, one benefit is now that we have several small testable functions, so if we need to debug, that is much easier to do since we can call each function one at a time. Additionally, if we wanted to add support for `<` and `>`, we don't have to change any code in our functions, but rather the value of two constants:

```java
static final String LEFT_BINDINGS = "([{<";
static final String RIGHT_BINDINGS = ")]}>";
```

As an additional note, if I expected these constants to change frequently, or for different data formats I would want different values, I would like move these constants to some type of configuration file, such as a properties file. However, for now, I'm satisfied that this solution will be stable for the problem we're solving.


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

Already our test is much shorter, and arguably easier to read. Before we commit this change, we want to make sure our test still passes. We run it, and it passes, so we commit.

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

### Trade-off

Note this comes with a trade-off - where we initialize our objects is separate from our tests. In general, I find this trade-off worthwhile so long as I am simply building my objects and injecting test objects in the BeforeEach. Any actual *data* in the objects relevant to whether a specific test should be in that test, **never** in the `BeforeEach` method. For instance, as a general rule, I avoid every populating any collections (lists, maps, etc.) in the @BeforeEach, since the contents of those collections are likely to be relevant to specific tests.

Now, you might think "but `gardensOfTheMoon` is data", but we never modify any of the data in `gardensOfTheMoon` in our test. Rather, it is functioning more like a constant than a variable for testing purposes.

Additionally, in some cases, we may want to set up our objects differently in some tests than in others. In those tests, we can simply overwrite the values of `testBookCopies`, `testPatronList`, `testLibrary` within the test itself.

## Conclusion

I showed multiple code refactors built around extracting duplicate code here. Most of the refactoring I did was extracting methods, extracting fields, and renaming to improve readability. Using a good IDE like IntelliJ, this process is actually very quick once you get practice with it.
