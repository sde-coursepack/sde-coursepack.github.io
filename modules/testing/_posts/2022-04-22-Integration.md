---
Title: Integration Testing
---


# Software Integration

In our design unit, we will discuss how using interfaces and abstract classes allows us to subvert the inherent "top-down" implementation that we get with functional decomposition starting from the `main` method. Specifically, using interfaces and inheritance allow us to build a "plug-and-play" architecture that removes direct dependencies from "high-level" classes to "low-level" classes.

However, as our software system is constructed, it's not enough to know that each *module* (in Java, module typically means class, but it can also mean function) in our system works independently. We need to ensure that as we combine multiple modules together, that they are operating correctly in combination. The act of combining different software modules together is called **integration**. In this unit, we will discuss **integration** strategies and testing.

---

* TOC
{:toc}


---

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

"Big-Bang" integration strategy is effectively "no plan" integration. We haphazardly integrate modules in whatever order we feel like at the time (or "all at once", aka Big Bang). This may be sufficient for small programs, especially those that are unlikely to change. However, like ad hoc development, this approach is not scalable, and can lead to conflicting strategies by the developers on the team.

In general, when a defect is discovered while integrating, it's more natural to change the most recent module you integrated, as changing one module is easier than potentially changing several.

## Integrating Around Dependencies

For the next three plans, let's consider that you are building a Yelp style app for rating restaurants around the Charlottesville area. You have the following dependencies:

* `UserInterface` uses `ApplicationController` - users use a form to query a Restaurant by name
* `ApplicationController` uses `RestaurantManager` - ApplicationController gets information about matching restaurants from the Restaurant Manager
* `RestaurantManager` uses `ReviewManager` - The RestaurantManager gets all the reviews for a given restaurant
* `ReviewManager` uses `ReviewDatabaseDriver` - The Review Manager goes to the Database Driver for reviews and gets all the reviews for restaurants in a given area.

In this example, we will also have **data structures** `Restaurant` and `Review`. These **data structures** are different from the other objects, as they exist to simply hold data. Often, these are stored as POJOs (Plain Old Java Objects) - Objects that typically have some fields, and getters and setters for those fields. In general, these objects have very limited *behavior*, and simply exist to pass information between modules. As such, we generally are treating these objects no differently than we would treat a String. As such, we generally do not need to consider **data structures** too heavily in our integration plan in terms of order of integration.

### Bottom-up

The most obvious strategy you would probably think of is a "bottom-up" approach. That is, as we develop modules, we integrate them in order from the low-level to the high-level.

Using our example:

> * `UserInterface` uses `ApplicationController` - users use a form to query a Restaurant by name
> * `ApplicationController` uses `RestaurantManager` - ApplicationController gets the reviews about a restaurant and filters it to the most recent 5.
> * `RestaurantManager` uses `ReviewManager` - The RestaurantManager gets the 10 most recent reviews for the restaurant from the ReviewManager
> * `ReviewManager` uses `ReviewDatabaseDriver` - The Review Manager coordinates with the Review Database, and can get a Map of all the reviews for a given resta

This means the first integration testing we would perform is the `ReviewManager` and the `ReviewDatabaseDriver`. Then we do integration testing between those two modules combined with `RestaurantManager`, then `ApplicationController`, then `UserInterface`.

More precisely, we define **bottom-up** integration as: 

> Integrate starting with the class **that depends on nothing**. Integrate each class after all of its dependencies have been integrated.

This more general definition helps us think about integrating classes with multiple dependencies.

#### Advantages

A clear advantage of this approach is we can use a strategy of "integrating as we build". That is, if we develop the lowest-level modules first, we can then develop each "next level up". This means that we can test our unit and integration at the same time (though whether or not this is a good idea is something we will discuss in disadvantages).

Another advantage of this approach is that, as we integrate, if a fault emerges, it becomes easier to locate that fault, as it is likely related to our most recent integration.

Additionally, as we integrate higher-level modules, we can design our tests to go as "deep" as we need to go to fully test the feature.

#### Disadvantages

Often, the most critical aspects of our program (high-level logic) get tested last by this approach. If you do not plan carefully, you can end up with the higher-level classes have to change to accommodate lower-level classes, which we generally want to avoid in software design. These high-level modules tend to be the most critical for facilitating good software design and ensure good modularity, functional independence, and abstraction. 

This also often reverse the most natural way to think about our software process: we want to start with high-level abstractions, and decompose those into more concrete features. Here, our integration strategy is starting with low-level concrete classes and building upwards.

This can also make prototyping difficult, as it's much easier to show a useful prototype to customers with a real interface, but fake data, than the other way around.

### Top-down

Top-down integration reverses the order of bottom-up integration.

> Integrate starting with the class **on which nothing depends**. Integrate classes that are depended upon by your integrated classes.

So, building from our previous example, we would first integrate `User Interface` with `ApplicationController`. Then, we integrate `RestaurantManager`, Then `ReviewManager`, then `ReviewDatabaseDriver`.

But how can we test our integration if we have unimplemented dependencies? With stubs!

#### Top-Down mocking

Consider that we are integrating `RestaurantManager`. In this example, we may are testing the feature "The User Interface shows the 5 most recent reviews for the restaurant".

**Note that the below code is a hypothetical example, but is intended to show working code as best as possible.**

```java
public class UserInterface {
    private ApplicationController controller;
    
    public void displayRecentReviews(Restaurant restaurant) {
        List<Review> reviewList = controller.getRecentReviews(restaurant);
        System.out.println(formatReviewListForRestaurant(reviewList, restaurant));
    }
}
```

Note that in the above, this display method could also be used to produce an HTML page for a website, or send JSON to a phone app to display, or update some Widget, etc. The point is, we have some function communicating restaurant reviews to a user through an interface with the ApplicationController.

```java
public class ApplicationController {
    private RestaurantManager restaurantManager;
    
    public List<Review> getRecentReviews(Restaurant restaurant) {
        Set<Review> reviews = restaurantManager.getReviewsForRestaurant(restaurant);
        return reviews.stream()
                .sorted(Comparator.comparing(Review::getTimestamp).reversed())
                .limit(5)
                .toList();
    }
}
```
In this method, we use `RestaurantManager` to get all of the `Review`s for a given restaurant, and then filter that to the 5 most recent reviews.

Now we add in the class we are integrating: `RestaurantManager`, which has a dependency to `ReviewManager`

```java
public class RestaurantManager {
    private ReviewManager reviewManager;
    
    public Set<Review> getReviewsForRestaurant(Restaurant restaurant) {
        int restaurantZipCode = restaurant.getZipCode();
        HashMap<Restaurant, Set<Review>> reviewsInZipCode = reviewManager.getRestaurantReviewsMap(restaurantZipCode);
        return reviewsInZipCode.get(restaurant);
    }
}
```

You might be thinking "how can we run this code without implementing `ReviewManager`?" And the answer is: we can't! But we *can* test it using mocking and stubs so long as we have a clear interface (in this case one that contains the method ``. The below example is a **unit test** for Restaurant Manager using a library called `mockito` which we will dive deeper into in the next unit.

```java
import org.junit.jupiter.api.*;
import java.util.*;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

public class RestaurantManagerTest {
    private static final int TEST_ZIP_CODE = 22903;
    
    private RestaurantManager testRestaurantManager;
    private ReviewManager reviewManager;
    
    private static final Restaurant bodos = new Restaurant("Bodo's Bagels", 22903, FoodCategory.BREAKFAST);
    private static final Review testReview1 = new Review(bodos, "2022-07-23 10:37:00", 5, "Great bagels");
    private static final Review testReview2 = new Review(bodos, "2022-05-16 08:16:15", 4, "They should reopen their drive through");
    private static final Review testReview3 = new Review(bodos, "2022-08-04 16:23:17", 2, "My afternoon breakfast sandwich was good, but it had a hold down the middle, wtf!?!?!");
    private Review testReview1, testReview2, testReview3;
    
    private static final Restaurant chipotle = new Restaurant("Chipotle", 22903, FoodCatogory.MEXICAN);
    private static final Review testReview4 = new Review(chipotle, "2022-08-05 17:37:58", 5, "No holes in their burritos here, unlike bodos");
    
    @BeforeEach
    public void setup() {
        reviewManager = mock(ReviewManager.class);
        testRestaurantManager = new RestaurantManager();
        testRestaurantManager.setReviewManager(reviewManager);
        
        test
    }
    
    @Test 
    public void testGetReviewsForRestaurant() {
        when(reviewManager.getRestaurantReviewsMap(TEST_ZIP_CODE)).thenReturn(getTestReviewMapTwoRestaurants());
        
        Set<Review> actualOutput = testRestaurantManager.getReviewsForRestaurant(bodos);
        
        assertEquals(actualOutput, Set.of(testReview1, testReview2, testReview3));
    }
    
    private Map<Restaurant, Set<Review>> getTestReviewMapTwoRestaurant() {
        Map<Restaurant, Set<Review>> restaurantReviewMap = new HashMap<>();
        Set<Review> bodosReviews = Set.of(testReview1, testReview2, testReview3);
        Set<Review> chipotleReviews = Set.of(testReview4);
        restaurantReviewMap.put(bodos, bodosReviews);
        restaurantReviewMap.put(chipotle, chipotleReviews);
        return restaurantReviewMap;
    }
}
```

I know it looks like a lot here, but we'll break this down in general.

What we are doing here is using a library called Mockito to create an "on-the-fly" (that is, at runtime) stub for the class ReviewManager. Rather than implement ReviewManager, we are simply (for the sake of this test) hard coding the output to the function to be a map that allows us to test our basic functionality.

We hardcode a map of restaurants in the `private` method `getTestReviewMapTwoRestaurant()` to generate a simple example of what a map returned from `ReviewManager.getRestaurantReviewsMap(int zipcode)`. We are saying we want it to be a map with two restaurants, "Bodo's Bagles" and "Chipotle". Thus, we aren't really using (or implementing) `ReviewManager`, but rather using a "mock". The syntax for the `when` and `thenReturn` will be covered in more detail in the next unit. They key takeaway, however, is that this mocking **allows us to test, even when we have unintegrated, or even unimplemented, dependencies, so long as we know their interface**.

In the same way we wrote the above **unit test**, In this way, we can easily create prototypes based around this hard coded data that we can take as far as needed. Obviously, we won't use this data long-term, but you'll notice this data is only created in our **testing environment**, and thus safely won't affect our **main** source code environment.

#### Advantages of Top-Down

The biggest advantage is that this lets us integrate our system in the same we want to design our system: from the high-level to the low-level. This means we are able to make desicions about the high-level classes early before we integrate low-level classes, allowing us to ensure our high-level design is effective and flexible.

You may think writing up all these mocks is time-consuming. In truth however, with practice it can be a pretty quick process. In fact, most likely even if you were testing without a mock, you would still need to build the objects you are testing with during the test setup. Arguably, these mocks make it easier, as you aren't working with a real class. And in actuality, we are better off testing with mocks anyway, especially during **unit testing**. We will discuss the reasons why in the next unit (Mockito Unit Testing).

#### Disadvantages of Top-Down

One disadvantage, however, can be the difficulty in testing user-interfaces directly. This can be tricky, as mocking user input can be difficult. Additionally, the lowest-level resources are often interacting with files, databases, web-services, etc., and interactions with real external resources can be hard to mock (for instance, the most common advice on Stack Overflow for "How can I mock a File class in Java" is "do not try to mock a File in Java").

Another disadvantage is that lower-level modules are often integrated last, and are typically tested in fewer integration tests as a result (as compared to Bottom-Up) since in most integration tests, the lower levels are either absent or mocked.

Additionally, in smaller apps, the mocking may be overkill if there are only a small number of total modules to integrate.

### Sandwich Integration

Also called hybrid integration testing, this combines bottom-up and top down. Generally, in sandwich integration, we will often start with a **target layer**. This could include class in the middle (like `ApplicationController` or `RestaurantManager`, for example). In this case, we can integrate both up and down, in the order we choose, testing as we go. In this case, we use Bottom-Up integration strategies to test **upwards** from our target layer, and Top-Down integration strategies to test **downwards** from our target layer.

A major advantage of sandwich testing is that it can be very valuable in larger applications, as it allows for more scalability as new modules and subprojects are integrated. We are often able to isolate the User Interface (which is typically the most volatile part of our application), while still being able to test by decomposing features into individual modules.

One major disadvantage is that this approach struggles where there are a large number of interdependencies between modules, as isolate a "target layer" to integrate up and down from becomes difficult.