---
Title: Integration Testing
---

# Software integration.

In our design unit, we will discuss how using interfaces and abstract classes allows us to subvert the inherent "top-down" implementation that we get with functional decomposition starting from the `main` method. Specifically, using interfaces and inheritance allow us to build a "plug-and-play" architecture that removes direct dependencies from "high-level" classes to "low-level" classes.

However, as our software system is constructed, it's not enough to know that each *module* (in Java, module typically means class, but it can also mean function) in our system works independently. We need to ensure that as we combine multiple modules together, that they are operating correctly in combination. The act of combining different software modules together is called **integration**. In this unit, we will discuss **integration** strategies and testing.

## Cautionary Tale: Mars Climate Orbiter

__Background__: The Mars Climate Orbiter was a automated satellite designed to orbit Mars, surveying the red planet for weather patterns, monitor for water vapor in the Martin atmosphere, and look for signs of past Martian climate change.

__Failure__: Because there are no brakes in space, the probe was meant to move in a precise trajectory to "skip" off the Martian atmosphere, using the atmosphere as a way to slow down. On September 23, 1999 (Earth-time, of course), the probe passed too close to Mars, going too deep into the atmosphere. All contact with the probe was lost, as it either burned up in the Martian atmosphere or re-entered heliocentric space (that is, orbiting the Sun rather than orbiting Mars). Contrary to proper retelling, it's almost certain the probe did not crash into the planet itself, although that difference is strictly academic, as in either case the project was lost.

__Details__: NASA often contracts work out to private developers. In the case of the Mars Climate Orbiter, NASA contracted a ground system (that is, software running on Earth, not on the satellite) to calculate needed thrust for the Martian insertion maneuvers to Lockheed Martin, who specializes in aerospace technology. The system was designed to calculate the thrust and amount angle needed for a successful insertion. This software system interacted with NASA's communication system that relayed instructions to the satellite in space. The satellite would then use its own NASA- built onboard systems to deliver the thrust at the correct angle and amount.

__Error__: However, there was an error in following the specification. NASA expected the thrust calculation system to return units of thrust in SI units (aka, metric units) -- specifically, Newton-seconds. However, Lockheed Martin's system produced a thrust calculation in English units, specifically pound-seconds. Because 1 pound-second equals roughly 4.5 newton-seconds, this mean the satellite vastly under-corrected its trajectory. To this day, NASA takes full responsibility, as they did not ensure the ground software was correctly producing SI units through proper testing and code review.

__Lesson__: All three systems mentioned -- 1) Trajectory Calculation system 2) Satellite Communication system 3) Satellite onboard thrust delivery program -- worked correctly in isolation. However, when combined, the three systems produced incorrect results, because information flow between these systems was incorrect. 

This speaks to the importance of integration testing: yes, we want to first make sure each module (class or function) works independently. But we also need to ensure our modules **integrate correctly**. This brings us to the need for **integration testing**.

## Integration Testing

When performing integration testing, we want to ensure two or more modules are communicating correctly. These test can often be automated with JUnit tests. For example, in our [NBAExcelTeams](https://github.com/sde-coursepack/NBAExcelTeams) repository from earlier units, we can ensure that the two classes [`BallDontLieReader`](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/main/java/edu/virginia/cs/nbateams/BallDontLieReader.java) (which queries the [Ball Don't Lie public API](https://www.balldontlie.io/) for NBA team information and returns a JSONObject) and [`NBATeamReader`] (which uses BallDontLieReader for the JSONObject and builds a list of NBA Teams) are working together correctly with a [test like this one](https://github.com/sde-coursepack/NBAExcelTeams/blob/main/src/test/java/edu/virginia/cs/nbateams/NBATeamReaderIntegrationTest.java).

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

Of course, our test could be more thorough if we wanted, but for now simply ensuring that we get at least 30 teams is a good starting point. This is an **integration test**, not a unit test, because it relies on both modules working *together* correctly. It's not enough if both modules correctly work independently: if the JSON Object passed between these classes is not in an agreed upon format, this integration will fail! 

**As a side note**: you'll notice that this creates a potential problem in the future for us: namely if Ball Dont Lie API changes their data format, it can break this integration. This vulnerability is something that we would want to control for. This is also why we have two different classes. BallDontLieReader only handles the query and returning the JSONObject, while **only** NBATeamReader handles actually parsing the JSON Object. That way, if the format changes, I know I only need to edit NBATeamReader (whereas if the API went down completely, I would potentially need to edit both - something that I could fix by defining interfaces between these two classes).

## Integration Strategies

Now that we have a clear definition of integration, combining two or more modules together to ensure correct behavior of the modules collectively, we have to ask "how do we combine modules?" This isn't an easy decision. In general, we will almost always have more relationships between classes than we will classes themselves. This means we have more **integrations** than we have **units** for the sake of testing. How we go about this process can have significant ramification for our development process. Above all else, we want to avoid **"integration hell"**.

### Integration Hell

Integration Hell is state when the time and effort it takes to integrate new or changing software modules exceeds the time and effort of the new modules' development. That is, it would have been better to start from scratch and reimplement the module(s) from scratch to integrate correctly than to integrate the existing module.

Symptoms of integration hell include:
- If Class A depends on Class B, but a change to class B forces a change in Class A, this suggests a poor integration plan and can snowball dependency issues to classes that depend on Class A.

Design can help us avoid integration hell. For example, proper use of `interface`s and `abstract class`es can help us create "coding by contract", which simplifies integration by forcing developers to build according to a standardized interface. However, a standardized interface still cannot avoid all integration concerns (such as the Mars Climate Orbiter where the syntax of the interface was correct, but the semantics where incorrect). At core, we want a strategy that limits the likelihood of integration hell.

### Big-Bang Integration

"Big-Bang" integration strategy is effectively "no plan" integration. We haphazardly integrate modules in whatever order we feel like at the time. This may be sufficient for small programs, especially those that are unlikely to change. However, like ad hoc development, this approach is not scalable, and can lead to conflicting strategies by the developers on the team.

## Integrating Around Dependencies

For the next three plans, let's consider that you are building a Yelp style app for rating restaurants around the Charlottesville area. You have the following dependencies:

* `UserInterface` uses `ApplicationController` - users use a form to query a Restaurant by name
* `ApplicationController` uses `RestaurantManager` - ApplicationController gets information about matching restaurants from the Restaurant Manager
* `RestaurantManager` uses `ReviewManager` - The RestaurantManager gets the 10 most recent reviews for the restaurant from the ReviewManager
* `ReviewManager` uses `ReviewDatabaseDriver` - The Review Manager goes to the Database Driver for reviews and gets all the reviews for the requested restaurant, before filtering and returning the most recent 10

In this example, we will also have **data structures** `Restaurant` and `Review`. These **data structures** are different from the other objects, as they exist to simply hold data. Often, these are stored as POJOs (Plain Old Java Objects) - Objects that typically have some fields, and getters and setters for those fields. In general, these objects have very limited *behavior*, and simply exist to pass information between modules. As such, we generally are treating these objects no differently than we would treat a String. As such, we generally do not need to consider **data structures** too heavily in our integration plan in terms of order of integration.

### Bottom-up

The most obvious strategy you would probably think of is a "bottom-up" approach. That is, as we develop modules, we integrate them in order from the low-level to the high-level.

Using our example:

> * `UserInterface` uses `ApplicationController` - users use a form to query a Restaurant by name
> * `ApplicationController` uses `RestaurantManager` - ApplicationController gets information about matching restaurants from the Restaurant Manager
> * `RestaurantManager` uses `ReviewManager` - The RestaurantManager gets the 10 most recent reviews for the restaurant from the ReviewManager
> * `ReviewManager` uses `ReviewDatabaseDriver` - The Review Manager goes to the Database Driver for reviews and gets all the reviews for the requested restaurant, before filtering and returning the most recent 10

This means the first integration testing we would perform is the ReviewManager and the ReviewDatabaseDriver. Then we do integration testing between those two modules combined with RestaurantManager, then ApplicationController, then UserInterface.

More precisely, we define **bottom-up** integration as: 

> Integrate starting with the class that depends on nothing with the class that depends on it. Integrate each class after all of its dependencies have been integrated.

This more general definition helps us think about integrating classes with multiple dependencies.

#### Advantages

A clear advantage of this approach is we can use a strategy of "integrating as we build". That is, if we develop the lowest-level modules first, we can then develop each "next level up". This means that we can test our unit and integration at the same time (though whether or not this is a good idea is something we will discuss in disadvantages).

Additionally, as we integrate, we can design our tests to go as "deep" as we need to go to fully test the feature.

#### Disadvantages



### Top-down

### Sandwich Integration