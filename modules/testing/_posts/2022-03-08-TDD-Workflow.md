---
Title: TDD Example
---

## [Source Code Starting Point](https://github.com/sde-coursepack/Library/tree/9d07a9430a738f6ccbd655a357986d236b68e41d/src/main/java)

In this unit, I will be showing the progress of a TDD workflow starting from this point. Necessarily, I have made additional commits to this GitHub project as I progressed, but this commit is the starting point for the function we will write.

# TDD Workflow

Let's consider writing a Library checkout system. We have three classes to consider. In the space below, I show the fields for each class. All of the fields have getters and setters unless otherwise noted, but for space they aren't included. 

`Patron` - a library patron, with a list of books checked out. Two patrons are equal if they have the same `id`.

```java
public class Patron {
    private final int id;
    private String firstName, lastName;
    private List<Book> booksCheckedOut;

    public Patron(int id, String firstName, String lastName, List<Book> booksCheckedOut) {
        this.id = id;
        this.firstName = firstName;
        this.lastName = lastName;
        this.booksCheckedOut = booksCheckedOut;
    }

    public void addBookToCheckedOut(Book b) {
        booksCheckedOut.add(b);
    }

    public void removeCheckedOutBook(Book b) {
        if (!booksCheckedOut.contains(b)) {
            throw new IllegalArgumentException(
                    generateRemoveCheckedOutBookError(b));
        }
        booksCheckedOut.remove(b);
    }

    public String generateRemoveCheckedOutBookError(Book b) {
        return "Patron cannot return book they have not checked out!\n" +
                "\t" + this +
                "\tCheck out list: " + this.booksCheckedOut +
                "\t" + b;
    }

    public int getNumberOfBooksCheckedOut() {
        return booksCheckedOut.size();
    }
}
```

`Book` - represents the information about a book. Two books are equal if they have the same `id`.

```java
public class Book {
    private final int id;
    private String title;
    private String author;

    public Book(int id, String title, String author) {
        this.id = id;
        this.title = title;
        this.author = author;
    }
}
```

`Library` - models a library who has a set of books and a list of patrons:

```java
public class Library {
    public static final int MAX_BOOKS_PER_PATRON = 3;

    private Map<Book, Integer> bookCopies;
    private List<Patron> patrons;

    public Library(Map<Book, Integer> bookCopies, List<Patron> patrons) {
        this.bookCopies = bookCopies;
        this.patrons = patrons;
    }

    public Library() {
        this(new HashMap<>(), new ArrayList<>());
    }

    public boolean isPatron(Patron patron) {
        return patrons.contains(patron);
    }
    
    public void addPatron(Patron patron) {
        if (!isPatron(patron))
        patrons.add(patron);
    }
    
    public int getNumCopies(Book b) {
        return bookCopies.get(b);
    }
    
}
```

## `addBooks(Book b, int copies)`

We need to implement a feature:

"The Library must be able to add one or more copies of a Book to it's Set of books".

Right now, our Library doesn't have a feature. So we decide to implement that feature. We start by creating a new branch named `addBooks` off of our 'main' branch.

```shell
PS C:\Users\pm8fc\my_projects_folder\Library> git branch
* addBooks
  main
```

And we add our function **stub** (that is, signature with no meaningful implementation) to our `Library` class.

```java
public void addBooks(Book b, int copies) { }
```

And we commit with the message "Added stub method addBooks to Library.java". You can see our [commit here.](https://github.com/sde-coursepack/Library/commit/a6b885dd99505b96afed888ec8dfebcae3555ea7)

### Writing our the first test

Now that we have our stub, we can write our tests. Remember, we write a test that initially *fails*, and then we write enough code so the test passes.

#### Create setup

For our first test, I'm going to assume we are adding 1 copy of a new book to the Library. To make testing easier, I'm going to directly inject my own `HashMap` into the `Library` so I can monitor the HashMap's state without relying on any other methods in `Library`, the Class Under Test (CUT).

```java
    @Test
    public void addBooksNewBooksTest() {
        Map<Book, Integer> testBookCopies = new HashMap<>();
        List<Patron> patronList = new ArrayList<>();
        Library testLibrary = new Library(testBookCopies, patronList);
        
        Book gardensOfTheMoon = new Book(1, 
                "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
                "Steven Erikson");
        
    }
```

The first 3 lines of the test create my test `Library` object. In this test, I don't care about `patronList` really, but the Library constructor needs one, so I just make an empty one. I do, however, care about the `bookCopies` map, and I will be using this later in the test.

Do I need to use long titles like this? No. However, it's my chance to mention that Malazan Book of the Fallen is the best epic fantasy series I've ever read. If you like Game of Thrones, but wish it were even bigger and even more complicated, check it out.


#### Call the test function and assertions

Now, I invoke the method we are testing, `addBooks()` and add my assertions to check if the function behaved correctly:

```java
    @Test
    public void addBooksNewBooksTest() {
        Map<Book, Integer> testBookCopies = new HashMap<>();
        List<Patron> patronList = new ArrayList<>();
        Library testLibrary = new Library(testBookCopies, patronList);
        Book gardensOfTheMoon = new Book(1, 
                "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
                "Steven Erikson");

        testLibrary.addBooks(gardensOfTheMoon, 2);
        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book not added to Map");
        assertEquals(2, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies added");

    }
```

The asserts mean if we add two copies of a book to a new Map, then:
* that Map should have that book as a key
* the value of that key should be 2, since that means we have two copies

#### This test fails!

Remember, our method is still a stub. It's important our test fails! This means we have code to write.

```java
Test book not added to Map ==> expected: <true> but was: <false>
Expected :true
Actual   :false
```

Now, we commit!

#### Commit - 1st test written and failing

I made a commit with the message: "Added test for addBooks() when book is not already in bookRecords." You can see [the commit on GitHub here.](https://github.com/sde-coursepack/Library/commit/5c8e26b73e3872b33839ced644acd674b7749d0b)

#### Made 1st test pass

Now I can start writing code! This actually seems simple to write:

```java
    public void addBooks(Book b, int copies) {
        bookCopies.put(b, copies);
    }
```

I just simply add the book as a key to the Map `bookCopies` with the specified number of books as the value.

#### Rerun test

I re-run my test, and it passes now! [So I commit.](https://github.com/sde-coursepack/Library/commit/04daf2215e2f151402d18292e1f5016fe1e73c06)

### Writing our second test

One thing we also need to consider is: what if we are adding copies of a book our Library already has? Let's write a test to account for this.

#### Setup

Our inputs are similar to before. However, this time, we are going to pre-populate our `Library` with two copies of Gardens of the Moon.

```java
@Test
    public void addBooksExistingBooksTest() {
        Map<Book, Integer> testBookCopies=new HashMap<>();
        List<Patron> patronList=new ArrayList<>();
        Book gardensOfTheMoon=new Book(1,
        "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
        "Steven Erikson");
        Library testLibrary=new Library(testBookCopies,patronList);
        testBookCopies.put(gardensOfTheMoon,2);
    }
```

The last line is what adds the books to the Library's `bookCopies` Map. Notice that I do this by directly accessing the `Map` I created: `testBookCopies`. I **do not** call `testLibrary.addBooks`. That's because **that is the method I'm testing**, and I only want to call the Method Under Test (MUT) **once** during the test. If I call it more than once, how will I know which call caused the failure?

#### Call the test function and assertions

Now we add our assertions. Since we are assuming we already have 2 copies of Gardens of the Moon, if we add two more, we should expect 4 copies:

```java
    @Test
    public void addBooksExistingBooksTest() {
        Map<Book, Integer> testBookCopies = new HashMap<>();
        List<Patron> patronList = new ArrayList<>();
        Book gardensOfTheMoon = new Book(1,
                "Gardens Of The Moon: Book 1 of Malazan Book of the Fallen",
                "Steven Erikson");
        Library testLibrary = new Library(testBookCopies, patronList);
        testBookCopies.put(gardensOfTheMoon, 2);

        testLibrary.addBooks(gardensOfTheMoon, 2);
        assertTrue(testBookCopies.containsKey(gardensOfTheMoon), "Test book no longer in Map");
        assertEquals(4, testBookCopies.get(gardensOfTheMoon), "Incorrect number of copies after add");
    }
```

we run our test and it fails: 

```shell
Incorrect number of copies after add ==> expected: <4> but was: <2>
Expected :4
Actual   :2
```

#### Commit

We [commit here](https://github.com/sde-coursepack/Library/commit/b3e773d1e4160d6953fe70d230d2f2388b54620b) to mark that we have written a new test that is failing.

#### Implement to pass test

Now, we go back to our `addBooks` function and consider how we need to change it. Considering it appears we now have two different cases to consider, I wrote:

```java
    public void addBooks(Book b, int copies) {
        if (bookCopies.containsKey(b)) {
            int currentCopies = bookCopies.get(b);
            bookCopies.put(b, currentCopies + copies);
        } else {
            bookCopies.put(b, copies);
        }
    }
```

I now re-run my test and it passes! Before I commit however, I re-run **all** of my existing tests in Library to make sure I didn't break anything. In this case, I didn't; my first test still passes as well. [So I commit](https://github.com/sde-coursepack/Library/commit/e71d4cd40c1504372b16112b54c934d540d36677).

#### Merging changes

Now, I want to merge my changes. So I do the following:

1) Pull the branch I'm on so I know I have the most up to date code (in case anyone else has committed and pushed to the branch)
2) I checkout the `main` branch and pull to see if anything has been committed there
3) I switch back to `addBooks` branch
4) I merge the `main` branch into the `addBooks` branch and handle any conflicts, commit and push.
5) I checkout `main` again
6) I then merge `addBooks` into `main` and push.
7) Commit the merge and push.

The commits made to `addBooks` can [now be seen in `main`](https://github.com/sde-coursepack/Library/commits/main). Notice how every commit from `addBooks` can be seen in main, now. That's because the merge copied all the commits over. (You can reduce the number of commits shown using a command called `squash`, but I'm not showing it here since it's beyond the scope of this lesson.)

Normally at this point, I would delete the `addBooks` branch, since I don't need it anymore. However, because this is an illustrative example, I left the `addBooks` branch on GitHub so you can see it.

## Second example: checkOut

While I won't walk through it in detail, I also created a branch called `checkOut` to implement a function called `checkOut`. That feature is not finished, and so I haven't merged it yet. [However, you can see the commit history here.](https://github.com/sde-coursepack/Library/commits/checkOut)

You can see the general pattern of commits:

1) Write a test for a particular feature in checkOut that fails
2) Write enough code to pass the test
3) Repeat

You can see my commits are on the scale of minutes (save for a two hour gap where I had office hours). I don't work for hours and then commit. I commit like I'm saving the file. The reason I can do this is because I'm working in a new feature branch, which only I'm working in. These frequent commits allow me to track my progress, but also know which features of `checkOut` are already implemented if I have to take a long gap, or I hit a roadblock.

## Cleaning up

However, the tests as they are right now are "dirty code." That is, they are very repetitive, and someone disorganized. In the Code quality unit, we will look back at this class and clean it up (albeit in another branch so this module isn't affected).

## Conclusion

You might be thinking this is overkill, but it isn't. This is how to develop code that you can be confident in, and that you can work with long term. By testing in this way, I'm catching bugs before they get introduced, and ensuring my code works **the way I intended it to**. Without testing, I only **hope** my code works. With testing, I can be confident that it actually does.