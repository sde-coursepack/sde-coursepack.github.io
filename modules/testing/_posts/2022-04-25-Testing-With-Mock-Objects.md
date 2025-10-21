---
Title: Mocking Objects
---

# Mocking Objects

In this unit, we can show how mock objects can be used to help us more precisely test specific functions *without* testing or needing to use their dependencies directly. This can help us with testing higher-level code without also testing or setting up their low-level dependencies.

---

* TOC
{:toc}

---

## [Source Code Example](https://github.com/sde-coursepack/NBAExcelTeams)

## Testing more precisely

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

1. We are testing more than one operation, since we are also testing
`getAllNBATeams()` in the class `BallDontLieReader`. This means
if *either* function has a defect, both tests will fail.
2. `BallDontLieReader.getAllNBATeams()` could fail for reasons
**that have nothing to do with the code being tested!**
   1. You could be disconnected from the internet
   2. The BallDontLie API could be down for maintenance
   3. We could get an IO error when connecting to the server for any one of dozens of reasons
3. This means our test failure does not reliably tell us that there is a problem with our *code*. 

## Come to think of it...

With this in mind, even [`BallDontLieReaderTest`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/test/java/edu/virginia/cs/nbateams/BallDontLieReaderTest.java) is full of tests which **are not unit tests**. We have an external dependency to a web service API, and a failure in that communication would cause our tests to fail **even if our code is functionally correct**.

When unit testing, we only want to test the class we are testing! We do not want our tests to fail for any other reason, or our tests could be considered misleading and people could ignore failures, attributing them to external dependency issues. Note that dealing with this in a class directly interacting with external information can be difficult. However, in a class like NBATeamReader, which only interacts with other classes in the program, we should be able isolate the class for testing with external dependencies.

## The engine light

Consider this classic scene from the TV Show the Simpsons:

<iframe width="560" height="315" src="https://www.youtube.com/embed/ddPQAJSm2cQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In the scene, Lisa notices that Homer's "Check Engine" light is on.
Homer says, "Uh-oh, tape must have fallen off." Homer then puts a
piece of electric tape over the light to block it out, and says "There, problem solved!" Humorously, the car immediately breaks down due to an engine issue.

**We do not want a failing unit test to be a check engine light!** Ideally, a unit test should fail **if and only if** the code being tested has a defect. We want to avoid *false positives* (that is, a failing test indicating a defect when there is none), because if we allow too many, you and your fellow developers will begin to think that a failed test "probably isn't your fault," and that way madness lies.

## Don't delete integration tests!

To be clear, integration tests are still valuable. We shouldn't
delete the existing tests we have. However, integration tests are
not a substitute for effective unit tests. So, without deleting
these existing tests, lets work on writing Unit tests for classes that use `BallDontLieReader` and `NBATeamReader`, while severing the reliance on any external dependencies.

If you wish to disable a test from automated testing because it relies on external dependencies, like a Web-API, you can use the `@Disabled(String message)` tag on the test method. The Disabled message will prevent the test from running automatically with the rest of the tests. You can also break tests up into groups to run separately, but that's beyond the scope of this course.

## Test Doubles

Of course, the obvious question is "how do we test an API reader
without an API"? The answer is **Test Doubles**. 

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
and one team without a good abbreviation (LA Lakers). Thus, we know
our **input** (our `StubNBATeamReader`), and our expected output (a list containing only the Boston Celtics).

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

What we need is a way to create a Stub class without actually creating a class. We will cover that in the next unit with [Mockito](https://sde-coursepack.github.io/modules/testing/Mockito/).