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

When unit testing, we only want to test the class we are testing! We do not want our tests to fail for any other reason, of our tests could be considered misleading, and people could ignore failures, attributing them to external dependency issues.

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

There are 3 ways we can using Test Doubles:

1. Stubs
2. Mocks
3. Fakes

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

### Stub Limitations

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

        List<NBATeam> expected = new ArrayList<>();
        expected.add(CELTICS);

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


### Mock

In a Mock, we are creating an object like a stub object to remove the need for external dependencies. However, we still want to monitor to verify that the class interacts with external dependencies as expected For example, the class `NBATeamExcelWriter` writes to an ExcelWorkbook and saves it. Instead of actually saving the file, and then reading the saved file back-in to ensure it is correct, we can instead check to see if the file "tries" to save, but block the actual file I/O operation.



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

## Why use test Doubles

You might right now be thinking that this is a lot of extra work, why not just do integration tests? After all, if the larger parts work, then the small parts must work, right?

**Remember**: debugging a little code is significantly easier than
debugging a lot of code! Using Stubs like this allows us to
dramatically shrink the code we are looking at!

Yes, this does take more code, but this isn't a problem. In fact, it's important to understand that "less code" does not mean "less work", as proper testing can drastically reduce our overall debugging work. The "code" of our stub is trivially easy to write, and it ensures that our test of `extractGoodAbbreviationTeams` is **stable** and **portable**. This test no longer relies on other classes working, no longer relies on the internet, and no longer relies on an external server. Additionally, even if our **implementation** of the class `NBATeamReader` changes, so long as the **interface** remains the same, this test will still work!



## Data Structures vs Objects

Generally, it's acceptable to use common and standardized data structures in tests without mocking them, so long as the data structure is unlikely to change. 

In our Design unit, we will talk about the difference between **objects** and **data structures**, but the key difference is that **data structures** exist only to model data. They otherwise don't *do* anything. For example, [`NBATeam`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/main/java/edu/virginia/cs/nbateams/NBATeam.java) would be considered a data structure, since it only models a collection of data that describes a single tightly cohesive idea: a team in the NBA. 

However, classes like [`BallDontLieReader`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/main/java/edu/virginia/cs/nbateams/BallDontLieReader.java) and [`NBATeamExcelWriter`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/main/java/edu/virginia/cs/nbateams/NBATeamExcelWriter.java) are decidedly *not* data structures, but rather **objects**. This is because both classes perform an action, and do not just model data.

In general, we **should** mock the behavior of **objects**, but we do not need to mock the behavior of **data structures**.

