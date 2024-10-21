---
Title: Types of Testing
---

# Types of Testing

What we focused on in the last unit was a type of testing
called **unit testing**. In this module, we'll discuss
different kinds of testing, and what they are used for.

* TOC
{:toc}



## From Small to Large in Scope

The first kind of *verification* testing (that is, assuring the
software works as specified) typically scales from small-scale
to large-scale.

### Unit Testing

**Unit testing** is testing individual modules (typically, individual)
functions. Unit testing is an incredibly powerful tool, as it
lets us catch many "silly" bugs immediately. Typically, in a given
class, we want to unit test all non-trivial public methods. Specifically,
non-trivial in this case means "not simple getters and setters."

The advantage of unit testing is that, when a unit test fails, we can
typically isolate the bug in the unit test to the method (that is, assuming we
have managed to isolate the method from dependencies, which we
will talk about later with Mockito). This dramatically reduces the
amount of code we need to search through for a bug. Additionally, if
we can use tests to differentiate when code works and when it doesn't, that
extra information can further help our debugging.

The biggest advantage of unit testing is how easy it is to automate,
ensuring that running the existing test suite (the group of tests written
to verify the software meets the specification) is a simple, automatic process.

In the next unit, we will use JUnit 5 to write Unit Tests.

### Integration Testing

Where unit testing is used to verify that individual modules are
operating correctly by themselves, **integration testing** is testing
when modules are combined.

For example, consider, in our [NBAExcelTeams project from the
poi Demo](https://github.com/sde-coursepack/NBAExcelTeams).

Specifically, consider the test class [NBATeamReaderIntegrationTest](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/test/java/edu/virginia/cs/nbateams/NBATeamReaderIntegrationTest.java).

```java
public class NBATeamReaderIntegrationTest {
    @Test
    public void getsThirtyTeams() {
        NBATeamReader reader = new NBATeamReader();
        List<NBATeam> teams = reader.getNBATeams();
        assertEquals(30, teams.size());
    }
}
```

While not immediately visible in this test, the function `getNBATeams()`
uses another class:

```java
    private JSONArray getTeamsArrayFromAPI() {
        BallDontLieReader apiReader = new BallDontLieReader();
        JSONObject apiOutput = apiReader.getAllNBATeams();
        return apiOutput.getJSONArray("data");
    }
```

This means what we are testing here is the **combination**
of the classes `NBATeamReader` and `BallDontLieReader`. That is,
we aren't just testing `NBATeamReader` by itself, we are testing
it **within the a larger operation** that combines the class with
another class.

Integration tests can also typically be automated. At this point, it may
seem like it isn't possible to test `NBATeamReader` **without also testing**
`BallDontLieReader.` However, in a later unit (Unit Testing with Mockito),
we will show you how to use mocking to create a "slice" of a program
that does not contain any other classes so we can test `NBATeamReader`
in isolation.

Much integration testing can be automated with tools like JUnit5.
However, some integration testing, especially those dealing with
User Interface or database connections, can require more significant
testing construction and specialized libraries/architecture.

For example, when testing with a real database
connection, developers will often make a test database which does
not contain real application data, but mocked-up application data.


When testing integration between the user interface and the rest
of the system, a real user interface may be generated during testing
with an automatic entity "pressing" buttons. 

These systems can be computationally expensive to run, as 
well as time-consuming to set up. As such, we still want to 
rely primarily on **unit-testing** as our front-line defense 
against bugs, due to the relatively low cost of setup and execution.
Integration testing, however, is still a vital step in any software
development process. 

A classic integration testing *failure* is the
Mars Climate Orbiter. Two software systems, one to calculate thrust and
the second to activate the rockets to deploy that thrust, both worked
independently. However, the thrust calculation unit returned a floating
point number calculated in English units (Pound-seconds), while the
system that deployed the thrust expected the number in SI units (Newton-seconds).
Thus, while both system worked independently, a critical defect
existed in their communication that was only manifest when the two 
units were integrated.

### System Testing

System testing involves testing the entire system as a whole *from
the user perspective*. For example, if you were building a course
registration website, you would do system testing by logging into
the website and attempting to perform tasks as an end-user would: you
would use a Web Browser to access the website (or a testing version
of the website), and interact with the website as, say, a student would.

System tests typically involve following a script to use a specific
feature or sets of features within an application. For example,
if you were testing a course registration system, you would likely try logging in as a student and adding a course to that student through the same web interface you expect students to use. You likely would connect your website to a separate testing data source with fake student accounts and fake courses, rather than testing on the actually data-source that students would use. But the key insight is that your interactions with the website testing will be the same as the interactions by students when they are actually *using* the website.

System testing is, of course, vital to ensuring the final product
meets the specification. However, because system testing relies on
interacting with the whole software system, it can make identifying the specific
source of failures (for example, a crash when loading a particular view)
very, very difficult! This is because if the system fails in some way (crashing, bug, incorrect behavior), that could occur *anywhere* in the executing code  in the software system. This means the defect found during system testing is hard to *trace* (find where in the source code the defect is caused to fix it).

If a defect is **first** found during *system testing*,
this likely means there was a lack of thoroughness in unit and integration
testing. This creates a difficulty, because finding a defect in an entire
software system is far harder than finding a defect in one or two modules.
As such, in system testing, we often want to use the stack trace
to identify where a defect occurs, and then take a step back and write
unit and integration tests to specifically find and correct the underlying
source of the defect.

Many students early on in programming
rely on system testing as *their only form of testing!*. That is, 
students view "writing code" and "testing and debugging" as two separate
steps that occur in order.

In reality, the best approach to testing is iterative: **test your code
as you write your code!** The key to having effective system testing is
finding "silly" defects early through unit testing and integration testing.
If you find that you are spending an inordinate amount of time system testing,
consider taking a step back and testing your individual modules first!

## Regression Testing

When writing and testing your code, have you ever fixed a bug, only
to find your fix introduced a new bug? And then you fix the new bug,
only to reintroduce the old bug? I personally have been caught in that
tug-of-war for hours before. The reason was that I was only testing
one thing at a time: the thing I was working on at that moment.

Regression testing is the idea of keeping all of your old tests around.
After all, if you have a good test that could identify a defect, why ever
throw it away? A good test is good for as long as the tested module
isn't radically changed.

The idea of regression testing is to ensure we do not reintroduce old
defects into our code, or break portions of our code that are already
working. Gradle actually performs regression testing for us using the
`gradlew test` command, which is part of the `gradlew build` command.

Typically, in regression testing, we run our entire suite of **automated
tests** for our software system. If any tests fail, we alert the developer.
This is valuable, because it acts as a line of defense against changes
that break seemingly unrelated sections of code.

### Continuous Integration

Regression testing is so valuable, that there are entire software
services related to regression testing. Continuous Integration, or CI,
is the practice in software development where, when code is merged
into or added to a "protected branch", like `main` or `development`, 
the code is run against the existing testing suite to ensure known
defects are not accepted into these protected branches. For example,
if you want to "hot-fix" a website (that is, commit a change directly
to the `main` branch which is actively hosted on the internet), CI
can notify you "Hey, your code fails these tests," and **reject** the
change, protecting the live website from being broken.

GitHub offers their own continuous integration service, GitHub
Actions, which uses their own servers to build and execute your
software tests. However, like everything else on the internet, 
nothing is truly free. GitHub actions, depending on your account
status, typically give ~2000 free minutes of CI time per month.
That is, while your tests are running on GitHub servers, that 
time is recorded and counted against this limit. After ~2000 minutes
are used in a given month, you are charge per minute for all
remaining usage.

This means that GitHub Actions CI is a great tool to use for your
personal projects. However, as your project gets larger and
more complicated, with more and more tests, eventually you may
want to consider other premium options depending on your scale.
Many private companies will host their own continue integration servers,
or will use external services like GitHub Actions, Travis CI, or Amazon
Web Services (AWS) depending on their resources and needs.

---

## Based on Audience

So far, we have focused on **internal** testing with developers.
The other tests change the audience of the tests. Generally, these
tests are *system tests*, but done with an external audience.

### Internal Testing

As we mentioned, during development, developers will write their
own tests (unit tests, integration tests). The tests are
designed to verify the development's functional completeness (that is,
that all specified features are added) and technical correctness (that
is, that the features are working as intended).

### Alpha Testing

The terms Alpha and Beta testing have become marketing terms of
late. For example, you may hear phrases like "Public Alpha", or
"Open Beta", etc. For instance, "Beta" has often become
the term for "Demo" or "Early public access" in the gaming world.
When defining these terms, try to ignore how you have seen the 
terms used in popular culture.

Alpha Testing is system testing carried out *internally*, that is,
by the developing organization. In alpha testing, members of the 
organization test the software, trying to simulate the behavior of
real users. The goal is to identify any existing issues in the software,
as well as try to ensure sufficient quality to move to showing
the software to end users.

Whereas in system testing, mock users typically follow type script,
in alpha testing, mock users are encouraged to experiment with the
system. In addition to using the system for typical
use-cases, often testers will try to find failures through abnormal usage
of the system.

### Beta Testing

The key distinction between alpha and beta testing is who the testing users
are. In a beta test, the goal is to find actual customers, or *external* people
who would be likely to use the system *after it releases.* In a beta
test, users are closely monitored for how they use the software, and
often their usage is logged extensively. This is so that if a bug
is found, or the user reports and issue with the system, it can
be recorded an acted upon before the product is shipped. Beta
versions of software are typically released to a limited number of
potential end users for the purposes of providing feedback.

### Acceptance Testing

Both Alpha and Beta testing are parts of the overall process of
Acceptance Testing. While our *internal* testing is primarily used
for **verification** (the software works as specified), the primary
purpose of acceptance testing is **validation** (the software satisfies
the customer's needs). Remember, while *verification* and *validation*
are related, they are not the same thing! The goal of Acceptance
testing is to ensure that the customer or other stakeholders are
satisfied with the final software.