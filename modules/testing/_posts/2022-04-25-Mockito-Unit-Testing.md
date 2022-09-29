---
Title: Testing with Mockito
---

## [Source Code Example](https://github.com/sde-coursepack/NBAExcelTeams)

# Testing more precisely

When doing unit testing, our goal is to test a single unit of
code. Now, this isn't new information. However, when developing
a module, we can often connect a module into the rest
of our system instinctively. This means that when testing, we
aren't actually doing unit testing, despite our best intentions.

Consider this test in [`NBATeamReaderIntegrationTest`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/test/java/edu/virginia/cs/nbateams/NBATeamReaderIntegrationTest.java)

```java
    @Test
    public void getsThirtyTeams() {
        NBATeamReader reader = new NBATeamReader();
        List<NBATeam> teams = reader.getNBATeams();
        assertEquals(30, teams.size());
    }
```

Is this a unit test?

Well...no, actually. However, this may not be readily apparent by
simply looking at this test. Instead, we need to dig into the
`getNBATeams()` function in [`NBATeamReader`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/main/java/edu/virginia/cs/nbateams/NBATeamReader.java)

Specifically, the function `getNBATeams()` calls the function
`getTeamsFromAPI()`, which in turn calls the function `getTeamsArrayFromAPI()`.

Here is that function:

```java
    private JSONArray getTeamsArrayFromAPI() {
        BallDontLieReader apiReader = new BallDontLieReader();
        JSONObject apiOutput = apiReader.getAllNBATeams();
        return apiOutput.getJSONArray("data");
    }
```

Here, you will see that this function has a clear **external 
dependency** to the class `BallDontLieReader`. This means
by calling `getNBATeams()` in our test, we are testing not
just if the function `getNBATeams()` works. Necessarily,
we are also reliant on the class `BallDontLieReader` working.

This means our test `getThirtyTeams()` is **not** a Unit Test
for the following reasons:

1. We are testing more than on operation, since we are also testing
`getAllNBATeams()` in the class `BallDontLieReader`. This means
if *either* function has a defect, both tests will fail.
2. `BallDontLieReader.getAllNBATeams()` could fail for reasons
**that have nothing to do with code being test!**
   1. You could be disconnected from the internet
   2. The BallDontLie API could be down for maintenance
   3. We could get an IO error when connecting to the server for any one of dozens of reasons
3. This means our test failure does not reliably tell us that there is a problem with our *code*. 

## Come to think of it...

With this in mind, even [`BallDontLieReaderTest`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/test/java/edu/virginia/cs/nbateams/BallDontLieReaderTest.java) is full of tests which **are not unit tests**. We have an external dependency to a web service API, and a failure in that communication would cause our tests to fail **even if our code is functionally correct**.

When unit testing, we only want to test the class we are testing! We do not want our tests to fail for any other reason, of our tests could be considered misleading, and people could ignore failures, attributing them to external dependency issues. Note that dealing with this in a class directly interacting with external information can be difficult. However, in a class like NBATeamReader, which only interacts with other classes in the program, we should be able isolate the class for testing with external dependencies.

## The engine light

Consider this classic scene from the TV Show the Simpsons:

<iframe width="560" height="315" src="https://www.youtube.com/embed/ddPQAJSm2cQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In the scene, Lisa notices that Homer's "Check Engine" light is on.
Homer says "Uh-oh, tape must have fallen off." Homer then puts a
piece of electric tape over the light to block it out, and says "There, problem solved!" Humorously, the care immediately breaks down due to an engine issue.

**We do not want a failing unit test to be a check engine light!** Ideally, a unit test should fail **if and only if** the code being tested has a defect. We want to avoid *false positives* (that is, a failing test indicating a defect when there is none), because if we allow too many, you and your fellow developers will begin to think that a failed test "probably isn't your fault", and that way madness lies.

## Don't delete integration tests!

To be clear, integration tests are still valuable. We shouldn't
delete the existing tests we have. However, integration tests are
not a substitute for effective unit tests. So, without deleting
these existing tests, lets work on writing Unit tests for classes that use `BallDontLieReader` and `NBATeamReader`, while severing the reliance on any external dependencies.

If you wish to disable a test from automated testing because it relies on external dependencies, like a Web-API, you can use the `@Disabled(String message)` tag on the test method. The Disabled message will prevent the test from running automatically with the rest of the tests. You can also break tests up into groups to run separately, but that's beyond the scope of this course.

## Test Doubles

Of course, the obvious question is "how do we test an API reader
without an API"? The answer is **Test Doubles*. 

**Test Doubles** are classes that replace *external dependency* objects with replacement  objects that imitate the behavior of the external dependency they are replacing. In this case, an external dependency is any code used by a class that isn't part of the class.

## Stubs

A Stub object is an object that replaces an object with another object of the same interface. However, the Stub class's implementation is typically hardcoded to return specific values. For example, if we were to use the Stub of [`NBATeamReader`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/main/java/edu/virginia/cs/nbateams/NBATeamReader.java), we may write something like this:

```java
public class StubNBATeamReader extends NBATeamReader {
    public static final NBATeam LAKERS = new NBATeam(1,"Lakers","Los Angelos","LAL",
            Conference.WESTERN, Division.PACIFIC);
    public static final NBATeam CELTICS = new NBATeam(2, "Celtics", "Boston", "BOS",
            Conference.EASTERN, Division.ATLANTIC);

    public List<NBATeam> getNBATeams() {
        return generateFakeNBATeamList();
    }

    private List<NBATeam> generateFakeNBATeamList() {
        return List.of(LAKERS, CELTICS);
    }
}
```

This code is in the class [`StubNBATeamReader`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/test/java/edu/virginia/cs/nbateams/StubNBATeamReader.java). Please be aware
that the `List.of` function was introduced in Java 9, and will not
work in earlier versions of Java. 

We would save this class **in our test folder**, not in our
src folder, as we only use this class while testing.
Then, if we were testing a function that used the class `NBATeamReader`, we would simply use an instance of `StubNBATeamReader` instead.

For instance, say we wanted to test the class [`GoodAbbreviations`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/main/java/edu/virginia/cs/nbateams/GoodAbbreviations.java), specifically the method `extractGoodAbbreviationTeams(NBATeamReader nbaTeamReader)`. If we followed the pattern of our earlier tests in this project, we would test this by creating an `NBATeamReader` object that interacts with `BallDontLieReader` that interacts with the external Web API. However, *that's not a unit test*! Instead, let's test using our fake class.

### Testing with our stub class

A reminder that in our earlier unit on Poi, we described a
"good abbreviation" as one where the first three letters of the
team's city matches the team's name. As such, our output from
`getNBATeams()` in `StubNBATeamReader` is perfect for this test,
as it gives us one team with a good abbreviation (Boston Celtics)
and on team without a good abbreviation (LA Lakers). Thus, we know
our **input** (our `StubNBATeamReader`), and are expected output (a list containing only the Boston Celtics).

Below is our [`GoodAbbreviationsTest`](https://github.com/sde-coursepack/NBAExcelTeams/blob/d0482f054179498a2b0c643546757d04fd9a5665/src/test/java/edu/virginia/cs/nbateams/GoodAbbreviationsTest.java) class.

```java
package edu.virginia.cs.nbateams;

import org.junit.jupiter.api.*;
import java.util.*;

import static org.junit.jupiter.api.Assertions.*;
public class GoodAbbreviationsTest {
    @Test
    public void testGoodAbbreviations() {
        GoodAbbreviations abbreviations = new GoodAbbreviations();
        NBATeamReader reader = new StubNBATeamReader();
        List<NBATeam> goodAbbreviationsTeams = abbreviations.extractGoodAbbreviationTeams(reader);
        List<NBATeam> expected = List.of(StubNBATeamReader.CELTICS);
        assertEquals(expected, goodAbbreviationsTeams);
    }
}
```

Using the above, we are able to test the class `GoodAbbreviations` without needing to rely on any external dependencies. The class `StubNBATeamReader` is used instead via polymorphism (since `StubNBATeamReader` extends our real class `NBATeamReader`). Because the result is hard-coded, it allows us to return a value that is tailor-made to test our method (in this case, we test both the case of a team with a "good" Abbreviation and a team without one). 

### Manual Stub Limitations

One limitation of this stub is that we only designed it for testing one method. What if we wanted to test a method for team's whose "city" value is actually a state, like the Utah Jazz or Minnesota Timberwolves? Now this stub isn't as flexible. We have two options in order to maintain this class:

1. We go back and change our existing Stub class every time we want to test a new method. This approach is bad because every time we change our existing Stub class, any existing tests might have to change. For example, adding Utah Jazz to our stub class to test the state name would require us to update our `GoodAbbreviationsTest`

2. Create a new stub class for each test. But this means creating a lot of classes that are only used in a single class, polluting the global name space.

What we need is a way to create a Stub class without actually creating a class.

---

## Stubs with mockito

Enter [`mockito`](https://site.mockito.org/), a framework for improving unit testing. Mockito a can support "on the fly" stubbing within the test. Here is [`GoodAbbreviationsTest`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/test/java/edu/virginia/cs/nbateams/GoodAbbreviationsTest.java), but using mockito.

```java
package edu.virginia.cs.nbateams;

import org.junit.jupiter.api.*;
import java.util.*;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class GoodAbbreviationsTest {
    private NBATeamReader reader;

    private static NBATeam LAKERS, CELTICS;

    @BeforeAll
    public static void init() {
        LAKERS = new NBATeam(1,"Lakers","Los Angelos","LAL",
                Conference.WESTERN, Division.PACIFIC);
        CELTICS = new NBATeam(2, "Celtics", "Boston", "BOS",
                Conference.EASTERN, Division.ATLANTIC);
    }

    @BeforeEach
    public void setup() {
        reader = mock(NBATeamReader.class);
    }

    @Test
    public void testGoodAbbreviations() {
        when(reader.getNBATeams()).thenReturn(List.of(LAKERS, CELTICS));

        GoodAbbreviations abbreviations = new GoodAbbreviations();

        List<NBATeam> expected = List.of(CELTICS);

        List<NBATeam> goodAbbreviationsTeams = abbreviations.extractGoodAbbreviationTeams(reader);
        assertEquals(expected, goodAbbreviationsTeams);
    }
}
```

### Stub Test Double

Look at the line:

`reader = mock(NBATeamReader.class);`

What we are saying here is that we want to create a **Test Double object** that has the same interface as `NBATeamReader`. This creates our mock object that we will use in this test class.

### Writing a stub with mockito

Look at the first line of our test function:

`when(reader.getNBATeams()).thenReturn(List.of(LAKERS, CELTICS));`

What does this line mean? Well, in prose, we would read it as "**When** `reader.getNBATeams()` is called, **then return** a list of the LAKERS and CELTICS teams".

In short, we are creating a stub-function here! We no longer need the heavy-weight namespace consuming class `StubNBATeamReader`.
Instead, we are creating our stub right here!

## Installing mockito

Installing mockito in a gradle project, like any other library is
simple. You can simply add:

`testImplementation group: 'org.mockito', name: 'mockito-core', version: '4.7.0'`

To your dependencies. You can, of course, use a more recent version
number if you wish (at the time of this writing, 4.7.0 is the
most recent version)

### Import statements

`import static org.mockito.Mockito.*;`

This gives you access to the functions in mockito we want to use.

## Top-down integration

Let's now consider that we want to integrate `GoodAbbreviations` with `NBATeamReader` and test to ensure it works. Now, instead of mocking the behavior of `NBATeamReader`, we instead mock the behavior of `BallDontLieReader`, and use the actual `NBATeamReader` class.

### Example

```java
package edu.virginia.cs.nbateams;


import org.junit.jupiter.api.*;
import org.json.*;
import java.util.*;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class GoodAbbreviationsIntegrationTest {
    private GoodAbbreviations goodAbbreviations;
    private NBATeamReader nbaTeamReader;
    private BallDontLieReader mockBDLReader;

    private static final NBATeam CELTICS = new NBATeam(2, "Celtics", "Boston", "BOS",
            Conference.EASTERN, Division.ATLANTIC);

    @BeforeEach
    public void setup() {
        goodAbbreviations = new GoodAbbreviations();

        mockBDLReader = mock(BallDontLieReader.class);
        nbaTeamReader = new NBATeamReader(mockBDLReader);
    }

    @Test
    public void testGoodAbbreviationsFromNBATeamReader() {
        JSONObject mockJSONObject = getMockJSONObject();
        when(mockBDLReader.getAllNBATeams()).thenReturn(mockJSONObject);

        List<NBATeam> goodAbbrvTeams = goodAbbreviations.extractGoodAbbreviationTeams(nbaTeamReader);
        assertIterableEquals(goodAbbrvTeams, List.of(CELTICS));

    }

    private JSONObject getMockJSONObject() {
        String mockJSONString = """
                {
                  "data":[
                     {"id":2,"abbreviation":"BOS","city":"Boston","conference":"East","division":"Atlantic",
                             "full_name":"Boston Celtics","name":"Celtics"},
                     {"id":14,"abbreviation":"LAL","city":"Los Angeles","conference":"West","division":"Pacific",
                             "full_name":"Los Angeles Lakers","name":"Lakers"}
                    ]
                }
                """;

        JSONObject teamsJSONObject = new JSONObject(mockJSONString);
        return teamsJSONObject;

    }
}
```

While this may look like a lot, realize that `getMockJSONObject` is only used to create the mock `JSONObject` that is in the same format of a JSONObject we would expect from `BallDontLieReader`. However, from here, no mocking is done inside of the classes `NBATeamReader` or `GoodAbbreviations`. In this way, we have gone one step in a top-down integration, downwards from GoodAbbreviation (because GoodAbbreviation could itself be used by another class, this could be an example of part of a sandwich integration where `GoodAbbreviations` is the target layer).

## Wait, what about JSON?

At this point, you may notice that I haven't focused on the
external dependency of JSONObject and JSONArray, and this is
a fair criticism. However, JSONObject and JSONArray are, in
effect, data structures that are no dissimilar from Maps and Lists. While we have a unit on the org.json library
later, it's worth looking at the structure of a JSON
file for now:

```json
{"professor" : {
    "name" : {
      "first" : "Paul",
      "last" : "McBurney",
      "preferred" : "Will"
    },
    "degrees" : [
      { "degree" : "BS", 
        "subject" : "Computer Science",
        "year" : 2010,
        "institution" : "West Virginia University"
      },
      { "degree" : "MS",
        "subject" : "Computer Science",
        "year" : 2012,
        "institution" : "West Virginia University",
        "advisor" : "Tim Menzies"
      },
      { "degree" : "Ph.D.",
        "subject" : "Computer Science",
        "year" : 2016,
        "institution" : "University of Notre Dame",
        "advisor" : "Collin McMillan"
      }
    ]
  }
}
```

This structure, fundamentally, is a combination of Maps and Lists. For example:

* "professor" is the key to a **map** that has keys "name" and "degrees"
    * "name" is the key to a **map** that has keys:
        * "first" - value "Paul"
        * "last" - value "McBurney"
        * "preferred" - value "Will"
    * "degrees is the key to a **list** of degrees:
        * index 0 is my Bachelor's degree
        * index 1 is my Master's degree
        * index 2 is my Ph.D.

Because the JSON format is highly standardized and stable, I can
still feel comfortable using real JSON objects in testing, and
therefore the `org.json` library. There are purists who will
say we should unit-test by mocking the behavior of that library,
and there are fair reasons for that argument. For example, if
the JSON format were less well-establish and the structure of files
could change, or org.json were still new and the interfaces could change, then our tests could potentially fail because of things outside of our code. However, to me, it's fundamentally no different than testing using ArrayLists, HashMaps, or Strings.

## Mocks

A **mock** is very similar to a stub. We are creating an object like a stub object to remove the need for external dependencies. However, we still want to monitor to verify that the class interacts with external dependencies as expected For example, the class `NBATeamExcelWriter` writes to an ExcelWorkbook and saves it. Instead of actually saving the file, and then reading the saved file back-in to ensure it is correct, we can instead check to see if the file "tries" to save, but block the actual file I/O operation.

Below is an example of using mocking a class that relies heavily on an internal `Map`. One limitation of testing our classes with abstract data types is that, typically, we have to pick a concrete implementation (like `TreeMap` or `HashMap`) to do testing. This can cause problems, because it makes our test inflexible, since they are tied to specific concrete types. If the underlying concrete type used by the object changes, this could break our tests.

Consider the [`Patron` class](https://github.com/sde-coursepack/Library/blob/main/src/main/java/Patron.java) from our `Library` example below. For the sake of brevity, I only included the constructor we plan to use and method we plan to test.

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
        if (booksCheckedOut.contains(b)) {
            throw new IllegalArgumentException("Already have copy checked out");
        }
        booksCheckedOut.add(b);
    }
}
```

In this case, we can test our method `addBookToCheckedOut` method by **mocking the List**. That's right, we can mock the abstract data type `List`, meaning we can test our class while keeping our test **independent of the concrete List implementation used.**

Here is an example of using a mocked List in [`PatronTest.java`](https://github.com/sde-coursepack/Library/blob/6dba0b86e97dd0be78fd19864b2ed2f8ba0ebdd3/src/test/java/PatronTest.java), and it will show us the same `when-then` syntax as before, as well as a new piece of Mockito syntax, `verify`:

```java
import org.junit.jupiter.api.*;
import java.util.*;
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class PatronTest {
    private Patron testPatron;
    private List<Book> mockList;

    private static final Book mistborn = new Book(3, "The Final Empire", "Brandon Sanderson");
    
    @SuppressWarnings("unchecked") // used because we cannot mock List<Book>, only List, which produces warning
    @BeforeEach
    public void setup() {
        mockList = mock(List.class);
        testPatron = new Patron(1, "Jane", "Doe", mockList);
    }

    @Test
    public void testCheckOutSuccessful() {
        //when you ask the mocklist if it contains Mistborn, it says "no"
        when(mockList.contains(mistborn)).thenReturn(false);

        //call the method addBookToCheckedOut(mistborn)
        testPatron.addBookToCheckedOut(mistborn);

        //verify that the mock list had add(mistborn) called on it
        verify(mockList).add(mistborn);
    }
}
```

### Mockito `verify` function

`verify` is used to verify **that a particular function, with a particular argument, was called on our `mockList` object.
This happens in our `Patron`'s `addBookToCheckedOut` when we execute:

`booksCheckedOut.add(b);`

Because `booksCheckedOut` in our `testPatron` instance **is** the same as `mockList`, and `b` is the same book as `mistborn` in our function call, this line does the same thing as saying:

`mockList.add(mistborn)`

This means, because we `did` have our test try to add `mistborn` to `mockList`, we can say that the function behaved *as expected.* Now, this may look odd, as we have a test with no explicit `assert` statements. You may even think we should check to make sure `booksCheckedOut` is in the state it should be. However, remember, **we don't have a real list**. This is a mock list. However, the simple fact that the method tried to add `mistborn` to the mocked `checkOutBookList` is sufficient for our testing purposes.

For a second example, let's now consider the false case:

```java
    @Test
    public void testCheckOutFailure_alreadyHaveBook() {
        //when you ask the mocklist if it contains Mistborn, it says "yes"
        when(mockList.contains(mistborn)).thenReturn(true);

        //call the method addBookToCheckedOut(mistborn), should get exception
        assertThrows(IllegalArgumentException.class, () ->
                testPatron.addBookToCheckedOut(mistborn));

        //verify that the mock list has never had add(mistborn) called on it
        verify(mockList, times(0)).add(mistborn);
    }
```

In this case, we use our old friend `assertThrows` to ensure the method throws an exception. However, we also want to `verify` that we have tried to add `mistborn` to our `mockList` zero times. The `times(0)` argument is useful for saying how many times a function should have been called. If we don't include a `times` argument, then it defaults to `times(1)` - that is, we call the function exactly one time. In the same way, you can use `times(2)`, `times(x)` where x is an int, etc.

Another way we could write this test is to say that, after our call to `contains`, **nothing should happen to this list at all**. This is the same test, but with our last statement changed.

```java
    @Test
    public void testCheckOutFailure_alreadyHaveBook() {
        //when you ask the mocklist if it contains Mistborn, it says "yes"
        when(mockList.contains(mistborn)).thenReturn(true);

        //call the method addBookToCheckedOut(mistborn), should get exception
        assertThrows(IllegalArgumentException.class, () ->
        testPatron.addBookToCheckedOut(mistborn));

        //verify that the mock list has never had add(mistborn) called on it
        verify(mockList).contains(mistborn);
        verifyNoMoreInteractions(mockList);
    }
```

`verifyNoMoreInteractions` means that no methods, other than the ones we have called `verify` with, have been called. In this case, we have to include `contains` since we did use it in the if-statement in Patron.


## Why use test Doubles

You might right now be thinking that this is a lot of extra work, why not just do integration tests? After all, if the larger parts work, then the small parts must work, right?

**Remember**: debugging a little code is significantly easier than debugging a lot of code! Using Stubs like this allows us to dramatically shrink the code we are looking at! We can even use this to customize our level of integration.

Yes, this does take more code, but this isn't a problem. In fact, it's important to understand that "less code" does not mean "less work", as proper testing can drastically reduce our overall debugging work. The "code" of our stub is trivially easy to write, and it ensures that our test of `extractGoodAbbreviationTeams` is **stable** and **portable**. This test no longer relies on other classes working, no longer relies on the internet, and no longer relies on an external server. Additionally, even if our **implementation** of the class `NBATeamReader` changes, so long as the **interface** remains the same, this test will still work!

