---
Title: More Mockito Examples
---

In this unit, we'll look at some more examples of Mockito to show it's use, as well as look at some additional features.

---

* TOC
{:toc}

---

Let's consider the following Java classes:

## Transcript.java

```java
public class Transcript {
    private final Map<Section, Grade> history;

    
    public Transcript() {
        this(new HashMap<>());
    }
    
    protected Transcript(Map<Section, Grade> history) {
        this.history = history;
    }
    
    public Grade getGrade(Section section) {
        if (!history.containsKey(section)) {
            throw new IllegalArgumentException("Transcript doesn't contain section: " + section);
        }
        return history.get(section);
    }
    
    public Optional<Grade> getBestGrade(Course course) {
        return history.keySet().stream()
                .filter(section -> course.equals(section.getCourse()))
                .map(history::get)
                .max(Comparator.comparing(Grade::getPrerequisiteScore));
    }
    
    public void add(Section section, Grade grade) {
        history.put(section, grade);
    }
    
    public Set<Section> getSections() {
        return Collections.unmodifiableSet(history.keySet());
    }

}
```

Specifically, let's say I want to test my Transcript code. The methods all depend all a class called `Section`, so I could make some `Section` objects for testing. The problem? `Section` is a really big class with a lot of additional dependencies.


## Section.java

```java
public class Section {
    private final int courseRegistrationNumber;
    private final int sectionNumber;

    private final Course course;
    private final Semester semester;
    private Location location;

    private TimeSlot timeSlot;
    private Lecturer lecturer;
    private int enrollmentCapacity;
    private int waitListCapacity;

    private final Set<Student> enrolledStudents;
    private final List<Student> waitListedStudents;

    public Section(int courseRegistrationNumber, int sectionNumber, Course course, Semester semester,
                   Location location, TimeSlot timeSlot, Lecturer lecturer, int enrollmentCapacity, int waitListCapacity) {
        this.courseRegistrationNumber = courseRegistrationNumber;
        this.course = course;
        this.semester = semester;
        this.sectionNumber = sectionNumber;
        this.location = location;
        this.timeSlot = timeSlot;
        this.lecturer = lecturer;

        this.enrollmentCapacity = enrollmentCapacity;
        this.waitListCapacity = waitListCapacity;

        this.enrolledStudents = new HashSet<>();
        this.waitListedStudents = new ArrayList<>();
    }

    //...lots of getters and setters, lots of other methods
}
```

To make a `Section`, I have to make at least a `Course`, a `Semester`, a `Location`, a `TimeSlot`, a `Lecturer`, and potentially several `Student` objects. This is an unavoidable dependency, but manually building this objects, especially into the exact testable state I want them in, is going to take a lot of work. Additionally, any logic errors in `Section` or any of its myriad dependencies could cause my tests to fail even if my `Transcript` is implemented correctly.

## TranscriptTest.java with Mocks

So, instead of making these objects, I can use Mockito to mock the `Section` and `Course` I need. For instance, let's look at some existing tests for `add` and `getGrade` which mock `Section`.

```java
@ExtendWith(MockitoExtension.class)
class TranscriptTest {

    @Mock
    HashMap<Section, Grade> history;
    @Mock
    Section mockSection, sectionAlgorithmsFail, sectionAlgorithmsPass, sectionSde;
    @Mock
    Course algorithms, sde;

    @Test
    void getGrade() {
        when(history.containsKey(mockSection)).thenReturn(true);
        when(history.get(mockSection)).thenReturn(Grade.A);
        var transcript = new Transcript(history);

        assertEquals(Grade.A, transcript.getGrade(mockSection));
    }

    @Test
    void getGrade_notPresent() {
        when(history.containsKey(mockSection)).thenReturn(false);
        var transcript = new Transcript(history);

        assertThrows(IllegalArgumentException.class, () -> transcript.getGrade(mockSection));
    }

    @Test
    void add() {
        var transcript = new Transcript(history);

        transcript.add(mockSection, Grade.C);

        verify(history).put(mockSection, Grade.C);
    }
}
```

### @Mock annotation

A note that in the code snippet above, I'm using slightly difference syntax than the last unit. I'm using the `MockitoExtensions` with allows for the use of the `@Mock` annotation. Note that you *must* have `@ExtendWith(MockitoExtension.class)` before the class declaration in order to be able to use `@Mock` in this way.

Specifically, looking at the declarations above:
```java
    @Mock
    HashMap<Section, Grade> history;
    @Mock
    Section mockSection, sectionAlgorithmsFail, sectionAlgorithmsPass, sectionSde;
    @Mock
    Course algorithms, sde;
```

This allows to define my mock objects just like I could define any test objects, but without having to explicitly add the code `history = mock(Map.class)` into a test or setup function. The object is simply assumed to be a mock. The same is true for the other mocks.

### getGrade Test

Let's look specifically at the first test for `getGrade`

```java
    @Test
    void getGrade() {
        when(history.containsKey(mockSection)).thenReturn(true);
        when(history.get(mockSection)).thenReturn(Grade.A);
        var transcript = new Transcript(history);

        assertEquals(Grade.A, transcript.getGrade(mockSection));
    }
```

Remember that `history` is a mock object, specifically a `Map<Section,Grade>`. `mockSection` is a mock of the `Section` class. Specifically, I am telling `history` to do the following:

* When you are asked if you contain `mockSection`, return `true`
* When you are asked for the grade from `mockSection`, return `Grade.A`

From there, I inject `history` into a test `Transcript` object via the constructor. Then, I call `transcript.getGrade(mockSection)`, expecting it to return A.

In this what, I have successfully tested the logic of getting a `Grade` for a Section from a particular `Transcript` without having to actually create a `Section`, along with it's `Course`, `Semester`, `Location`, etc.

Remember, *what am I testing?*. I am testing the `Transcript` method `getGrade(Section section)`. I am not testing `Section`, `Course`, or even `Grade`. I am only testing the following logic:

```java
    public Grade getGrade(Section section) {
        if (!history.containsKey(section)) {
            throw new IllegalArgumentException("Transcript doesn't contain section: " + section);
        }
        return history.get(section);
    }
```

This test ensures that my I bypass the `if` statement if the transcript contains a `section`, and returns the result of *getting* that Grade. Similarly, the next test checks the result of the `if` statement being true:

```java
    @Test
    void getGrade_notPresent() {
        when(history.containsKey(mockSection)).thenReturn(false);
        var transcript = new Transcript(history);

        assertThrows(IllegalArgumentException.class, () -> transcript.getGrade(mockSection));
    }
```

Here, I am telling `history` to say it **does not** have a grade for `mockSection`, which should result in an error being thrown. The `assertThrows` ensures that an `IllegalArgumentException` is throw when we call `transcript.getGrade()`.

### add Test

For testing `add(Section section, Grade grade)`, we use:

```java
    @Test
    void add() {
        var transcript = new Transcript(history);

        transcript.add(mockSection, Grade.C);

        verify(history).put(mockSection, Grade.C);
    }
```

Here, we are using `verify` to ensure that, as a result of adding the `mockSection` with a `Grade.C` to our Transcript, that that content is passed to our `history` dependency. Remember, we use `verify` to ensure that the correct post condition is envoked in our dependency, that is that the `history` map has the key-value `mockSection` and `Grade.C` added to it.

#### To mock or not to mock

Without mocking `history` (but still mocking the sections) we could do the following:

```java
    @Test
    void add() {
        history = new HashMap<>();
        var transcript = new Transcript(history);

        transcript.add(mockSection, Grade.C);

        assertEquals(Grade.C, history.get(mockSection));
    }
```

This is equivalent in terms of functionality as my above test where I *did* mock `history`. In both above cases, mocking still saves us the trouble of actually making the `Section` objects with all their dependencies, which is the main thing we want to avoid. It's debatable if mocking the `history` HashMap saves us much trouble, since we could initialize the object with `Map.of`. 

In practice, I would probably create real `HashMap`s in this case rather than mocking, but since this is educational, I am mainly wanting to illustrate how to use mocks, so I'm erring on the side of more mocks for this module.

The main takeaway, however, is this:

* When you are testing post-conditions of a **real** dependency, use `assertEquals` or another assert functions.  
* When you are testing post-conditions of a **mocked** dependency, use `verify` to ensure the correct function is invoked.

You cannot meaningfully use `assertEquals` on mocked objects to check post conditions, because the `mock` objects **do not have any real behavior.** They are simply a convenient placeholder for a real object, mocking *just enough* behavior for us to test the unit we care about.

#### Never mock the class under test

As a side not, in general, you should never mock the "Class-Under-Test" (in this case, `Transcript`) because the point of these tests is to test the **real** behavior of Transcript. So while I might mock `Transcript` in other test classes, I will never mock it in `TranscriptTest.java`. There may be *some* exceptions to this rule, but they are very rare.

### getBestGrade()

Now we want to test the function `getBestGrade()`:

```java
    public Optional<Grade> getBestGrade(Course course) {
        return history.keySet().stream()
                .filter(section -> course.equals(section.getCourse()))
                .map(history::get)
                .max(Comparator.comparing(Grade::getPrerequisiteScore));
    }
```

The point of this function is to get the best grade that a student has in a `Course`. While most students only take a `Course` (that is, only one `Section` of a `Course`) one time, a student could repeat a `Course` if they failed it. At that point, we only want to count the "best grade" when looking at prerequisites. So, the above code will filter the `Transcript` `history` map to only look at `Section`s of the input `Course`. We could test this code with the following.

```java
    @Test
    void getBestGrade_multipleTakes() {
        history = new HashMap<>(Map.of(
                sectionAlgorithmsFail, Grade.F,
                sectionAlgorithmsPass, Grade.B_PLUS,
                sectionSde, Grade.A
        ));
        when(sectionAlgorithmsFail.getCourse()).thenReturn(algorithms);
        when(sectionAlgorithmsPass.getCourse()).thenReturn(algorithms);
        when(sectionSde.getCourse()).thenReturn(sde);
        
        var transcript = new Transcript(history);

        var bestGradeOptional = transcript.getBestGrade(algorithms);
        assertTrue(bestGradeOptional.isPresent());
        assertEquals(Grade.B_PLUS, bestGradeOptional.get());
    }
```

To explain this test, we have two different `Section`s that were offerings of the `Course algorithms`. Additionally, we have one `Section` that is *not* part of `algorithms` but is instead a different `Course`, `sde`. I use real course names from UVA for my test cases simply because it helps me understand the metaphor better (as opposed to `mockCourse1`, `mockCourse2`, etc.).

Be aware that by default, my `Section` mock objects have no `Course`, since they don't have any real behavior. As such, I have to *tell them* what their course is via the `when` operation.

```java
    when(sectionAlgorithmsFail.getCourse()).thenReturn(algorithms);
    when(sectionAlgorithmsPass.getCourse()).thenReturn(algorithms);
    when(sectionSde.getCourse()).thenReturn(sde);
```

Here I'm saying that `sectionAlgorithmsFail` and `sectionAlgorithmsPass` are both part of the course `algorithms`, while `sectionSde` is part of the course `sde`

In this I'm expecting the *best* grade for `algorithms` to be a `B_PLUS`, since it is the best of the two grades this `Transcript` has for `algorithms`. However, by default, the Java stream terminal operation `max` returns an `Optional<Grade>` on a `Stream<Grade>`. This is because it's possible that the stream is empty by the time we reach the terminal, meaning there is no `max`.

That said, `Optional`s are fairly straight forward, and we can use the `boolean` function `isPresent` to assert the optional actually has a grade, than then use the `get` function to actually get the `Grade` for testing:

```java
   var bestGradeOptional = transcript.getBestGrade(algorithms);
   assertTrue(bestGradeOptional.isPresent());
   assertEquals(Grade.B_PLUS, bestGradeOptional.get());
```

#### No Grade found test

Similarly, we can use a test to ensure that when there is no `Section` on the student's `Transcript` that is related to `Course`, we return no grade at all:

```java
    @Test
    void getBestGrade_notPresent() {
        history = new HashMap<>(Map.of(
                sectionAlgorithmsFail, Grade.F,
                sectionAlgorithmsPass, Grade.B_PLUS
        ));
        when(sectionAlgorithmsFail.getCourse()).thenReturn(algorithms);
        when(sectionAlgorithmsPass.getCourse()).thenReturn(algorithms);
        var transcript = new Transcript(history);

        assertEquals(Optional.empty(), transcript.getBestGrade(sde));
    }
```

Specifically, `Optional.empty()` is the result of an `Optional` that doesn't contain anything. In this case, because I asked for the best `sde` grade, and there is no `sde` related `Section` in `history`, this returns `Optional.empty()`.

Both of my tests pass in this case. Again, notice I never use `verify`. This is because this method should have no post-conditions, or side-effects. This method simply queries the data to get the best grade. It doesn't *change* any data.

## Prerequisite.java

Now that we have this function in place, we can look at the `Prerequisite` class. Each `Course` has a `Prerequisite`, which lists zero or more required classes that must have been taken before you can register for a course, as well as the minimum grade in that course.

```java
public class Prerequisite {
    private final Map<Course, Grade> requiredCourses;
    
    public Prerequisite() {
        this(new HashMap<Course, Grade>());
    }
    
    public Prerequisite(Map<Course, Grade> requiredCourses) {
        this.requiredCourses = requiredCourses;
    }
    
    public void add(Course course, Grade minimumGrade) {
        requiredCourses.put(course, minimumGrade);
    }
    
    public Grade getMinimumGrade(Course course) {
        if (!requiredCourses.containsKey(course)) {
            throw new IllegalArgumentException("Prerequisites doesn't include course: " + course);
        }
        return requiredCourses.get(course);
    }
    
    public void remove(Course course) {
        if (!requiredCourses.containsKey(course)) {
            throw new IllegalArgumentException("Course: " + course + " is not in the prerequisites: " + this);
        }
        requiredCourses.remove(course);
    }

    public Set<Course> getPrerequisiteCourses() {
        return Collections.unmodifiableSet(requiredCourses.keySet());
    }

    public boolean isMetBy(Transcript transcript) {
        for (Course course: requiredCourses.keySet()) {
            var minimumGrade = requiredCourses.get(course);
            var optionalTranscriptGrade = transcript.getBestGrade(course);
            if (optionalTranscriptGrade.isEmpty()) {
                return false;
            }
            var transcriptGrade = optionalTranscriptGrade.get();
            if (!transcriptGrade.greaterThanOrEqualTo(minimumGrade)) {
                return false;
            }
        }
        return true;
    }
}
```

### isMetBy test

For now, let's focus on the method `isMetBy(Transcript transcript)`. This method checks whether a particular `Transcript` has met the prerequisites. This means the student has taken every class in the set of `requiredCourse` and has received *at least* the minimum required grade for that course.

#### true case

At UVA, our algorithms course, CS 3100 (aka, DSA2) has two prerequisites: CS 2100 (aka dsa1) and CS 2120 (aka dmt1, or Discrete Math) with a C- or better. We design the test case to follow this idea below:

```java

class PrerequisiteTest {
    @Mock
    private Course dsa1, dmt1;
    @Mock
    Map<Course, Grade> requiredCourses;

    private Prerequisite prerequisite;

    @Test
    void isMetBy_true() {
        var transcript = mock(Transcript.class);
        
        when(transcript.getBestGrade(dsa1)).thenReturn(Optional.of(Grade.A));
        when(transcript.getBestGrade(dmt1)).thenReturn(Optional.of(Grade.C_MINUS));
        
        requiredCourses = Map.of(
                dsa1, Grade.C_MINUS,
                dmt1, Grade.C_MINUS);
        
        prerequisite = new Prerequisite(requiredCourses);

        assertTrue(prerequisite.isMetBy(transcript));
    }
}
```

In this above test, `dsa1` and `dmt` are mock `Course`. We create our test `prerequisite` object using `requiredCourses`. We used a real `Map` for the `requiredCourses`, stating both mock courses require a `Grade.C_Minus` or better.

Our `Transcript` is itself a mock object. In this case, we are mocking the results of the `getBestGrade` method, saying that, for this transcript, the best grade for `dsa1` is an A, and the best grade for `dmt1` is a C-. 

The method under test, `isMetBy` is called in the last line, `assertTrue(prerequisite.isMetBy(transcript))` since this Transcript meets the prerequisites.

#### false case - Low Grade

Now let's look at one false case where the `false` return is caused specifically by a student's best grade being *below* the minimum required grade:

```java
    @Test
    void isMetBy_false_lowGrade() {
        var transcript = mock(Transcript.class);
        when(transcript.getBestGrade(dsa1)).thenReturn(Optional.of(Grade.D_PLUS));
        lenient().when(transcript.getBestGrade(dmt1)).thenReturn(Optional.of(Grade.A));

        requiredCourses = Map.of(
                dsa1, Grade.C_MINUS, 
                dmt1, Grade.C_MINUS);
        prerequisite = new Prerequisite(requiredCourses);

        assertFalse(prerequisite.isMetBy(transcript));
    }
```

This test is nearly identical to our true test case, with the exception being that the best grade for `dsa` is now a D+, which is worse than the required C-. As such this should return false, as you can see at the end of the method.

##### `lenient()` keyword

I want to quickly address the `lenient()` keyword in the above test:

`lenient().when(transcript.getBestGrade(dmt1)).thenReturn(Optional.of(Grade.A));`

Here, `lenient()` does not directly affect the test case. This is basically the same thing as saying:

`when(transcript.getBestGrade(dmt1)).thenReturn(Optional.of(Grade.A));`

However, a rule of `mockito` usage is that you should **only** `when-then` things that the method under test actually does. You should never add unnecessary `when-then`. 

When I remove this `lenient()` from this line:

```java
    var transcript = mock(Transcript.class);
        when(transcript.getBestGrade(dmt1)).thenReturn(Optional.empty());
        when(transcript.getBestGrade(dsa1)).thenReturn(Optional.of(Grade.A));
```

the logic of the test doesn't change. However, mockito may crash with the following error message:

```java
org.mockito.exceptions.misusing.UnnecessaryStubbingException: 
Unnecessary stubbings detected.
Clean & maintainable test code requires zero unnecessary code.
Following stubbings are unnecessary (click to navigate to relevant line of code):
  1. -> at sde.virginia.edu.hw4.PrerequisiteTest.isMetBy_false_missingClass(PrerequisiteTest.java:112)
Please remove unnecessary stubbings or use 'lenient' strictness. More info: javadoc for UnnecessaryStubbingException class.
```

The reason this happens is that, when the test is running, it is **possible**, depending on the order `isMetBy` loops through the `requiredCourses` Map that we never actually call: `transcript.getBestGrade(dsa1)`. This is because if we check the grade for `dmt1` first, then we have already failed the meet the pre-requisite, and so the function returns false without every checking `dsa1`. The `MockitoExtensions` sees this, and intentionally causes the test to fail, since there was a `when-then` we didn't use.

Since we cannot easily force the order that the course grades are checked in (and we wouldn't want to, since that's an implementation detail that could easily change), instead, we use `lenient`

```java
var transcript = mock(Transcript.class);
        when(transcript.getBestGrade(dsa1)).thenReturn(Optional.of(Grade.D_PLUS));
        lenient().when(transcript.getBestGrade(dmt1)).thenReturn(Optional.of(Grade.A));
```

This tells the `MockitoExtensions` that "Hey, it's find if we never actually check `dmt1`. Don't fail the test over it." Thus, `lenient()` is a way to say "it's possible this when-then isn't needed."

However, we **do not** want to add `lenient()` to the `dsa1` when-then, since we _absolutely_ want to ensure that `isMetBy` checks the `dsa1` grade (the grade that causes the `false` return). In general, I only use lenient when I need to, and this is typically caused by data being stored in `Set` or `Map` where the iteration order is intentionally unpredictable.

Annoyingly, without `lenient()`, the test will not always fail. This is because roughly half the time, `dmt1` will be checked **before** `dsa1`, meaning that there were no unnecessary when-thens. However, if `dsa1` is checked *before* `dmt1`, then `dmt1` is never actually checked because we return `false` without finishing the loop.


#### isMetBy false - No grade

We also want our `isMetBy` method to return false if the student has no grade for a specific course (implying they have never taken it).


```java
    @Test
    void isMetBy_false_missingClass() {
            var transcript = mock(Transcript.class);
        when(transcript.getBestGrade(dmt1)).thenReturn(Optional.empty());
        lenient().when(transcript.getBestGrade(dsa1)).thenReturn(Optional.of(Grade.A));

        requiredCourses = Map.of(dsa1, Grade.C_MINUS, dmt1, Grade.C_MINUS);
        prerequisite = new Prerequisite(requiredCourses);

        assertFalse(prerequisite.isMetBy(transcript));
        }
```

In this case, we use `Optional.empty()` to signal that this transcript has no grade for `dmt1`. Once again, it's possible this method returns false without ever checking `dsa1` due to not being able to predict the order the courses are iterated through, so we use the `lenient` keyword to let mockito know not to fail the test if that when-then isn't used.

## Dependency injection

Switching gears to some unrelated code, I want to consider the following code adapted from the [Three Layer Architecture](https://sde-coursepack.github.io/modules/patterns/Three-Layer-Architecture/) module:

```java
public class BestSellersService {
    public Book getLongestCurrentBestSeller(ListName listName) {
        var dataManager = new BestSellersDataManager();
        var bestSellers = dataManager.getCurrentBestSellerList(listName);
        var longestCurrentBook = bestSellers.getBooks().stream()
                .max(Comparator.comparing(Book::getWeeksOnList));
        if (longestCurrentBook.isEmpty()) {
            throw new EmptyBestSellerListException(bestSellers.toString());
        }
        return longestCurrentBook.get();
    }
}
```

For the sake of testing the logic of this method which is getting the longest running `Book` on a `BestSellerList`, we might start thinking of writing a test like this:

* Okay, so I'll need a list_name  
* Get that list from the `BestSellersDataManager`

But wait...you remember that the `BestSellersDataManager` connects to the internet to retrieve data from the New York Times API. This means this isn't really a unit test, this is an integration test.

So, okay, let's mock the `BestSellersDataManager`. The problem is that...well...you can't. Not without dipping into some more advanced features that come with a whole host of their own considerations that could make testing harder. This is because the instance `BestSellersDataManager` is created inside of the same method that uses it. Mocking only allows you to control the inputs to the method and how they behave. 

The solution? **Dependency Injection**. Simply take the creation `BestSellersDataManager` out of the function, and create it somewhere else. In this case, we create an injectable-constructor that populations a field.

```java
public class BestSellersService {
    private final BestSellersDataManager bestSellersDataManager;

    public BestSellersService() { //default constructor
        this(new BestSellersDataManager());
    }

    protected BestSellersService(BestSellersDataManager bestSellersDataManager) { //injectable constructor
        this.bestSellersDataManager = bestSellersDataManager;
    }

    public Book getLongestCurrentBestSeller(ListName listName) {
        var bestSellers = bestSellersDataManager.getCurrentBestSellerList(listName);
        var longestCurrentBook = bestSellers.getBooks().stream()
                .max(Comparator.comparing(Book::getWeeksOnList));
        if (longestCurrentBook.isEmpty()) {
            throw new EmptyBestSellerListException(bestSellers.toString());
        }
        return longestCurrentBook.get();
    }
}
```

Now, in our test, we can **inject** our mock bestSellersDataManager.

```java
@ExtendWith(MockitoExtension.class)
public class BestSellersServiceTest {
    private BestSellersService bestSellersService;

    @Mock
    private BestSellersDataManager mockDataManager;

    @Mock
    private BestSellersList mockBestSellersList;

    @BeforeEach
    public void setup() { //Dependency Injection here
        bestSellersService = new BestSellersService(mockDataManager); 
    }

    @Test
    void getLongestCurrentBestSeller() {
        Book gardensOfTheMoon = new Book("553812173","Gardens Of The Moon","Steve Erickson", 10); //10 weeks on list
        Book deadhouseGates = new Book("0553813110","Deadhouse Gates","Steve Erickson", 5); //5 weeks on list

        when(mockDataManager.getCurrentBestSellerList(ListName.COMBINED_FICTION)).thenReturn(mockBestSellersList);
        when(mockBestSellersList.getBooks()).thenReturn(List.of(gardensOfTheMoon, deadhouseGates));
        assertEquals(gardensOfTheMoon, bestSellersService.getLongestCurrentBestSeller(ListName.COMBINED_FICTION));
    }
}
```

Dependency injection is necessary for us to be able to mock the behavior of the `BestSellersDataManager` to allow us to force the method to return testable data without actually using the New York Times API. In short, we have created a real unit-test that only depends on the logic in our `BestSellersService` class and our `Book` data structure.

Dependency injection is a powerful design tool for a number of reasons, but it is also a powerful testing tool as we see here, especially when we use mockito!
