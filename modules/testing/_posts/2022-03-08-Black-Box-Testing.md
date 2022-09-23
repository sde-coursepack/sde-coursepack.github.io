---
Title: Black-Box Testing
---

The next three modules will show the construction of test cases for a single function, `calculateBill`, and how we would approach it.

# Black-Box Testing

The idea of Black-Box testing is that we select what test scenarios to write according to the *specification* of the modules we are testing (typically a function). We are testing the **interface**, not the **implementation**.

---


## Strategies

We will look at a couple flawed approaches to Black-Box testing first, and then consider **Equivalence Partitioning** as well as **Boundary and Exception** test cases.

### Exhaustive Testing

Exhaustive Testing is almost never feasible. Consider, for example, the function `max(int a, int b, int c)` for our previous modules. The `int` datatype supports 2<sup>32</sup> unique values. This means, to test exhaustively, we would have to test all combinations of ints, or (2<sup>32</sup>)<sup>3</sup>, which equals 2<sup>96</sup>. Just to store these test input numbers would use up approximately **3 billion times** the expected global computer storage of the entire world in 2025. And compared to data types like Strings and Lists, ints are downright simple at only 4 bytes.

### Random Testing

One idea is to pick inputs at random, then working out the expected output based on the random inputs generated. Whether we do this using an actual random number generator, or picking numbers out of thin air. A problem with this approach is that this can easily miss important test cases we should consider. For example, testing our 3-argument `max` function with 3 equal values is obviously an interesting case worth examining. However, if we are using random generation, the odds of generating this test are `n`/2<sup>64, where `n` is the number of tests we generate. Those odds are incredibly small. Instead, we want a systematic way to find a set of test scenarios that will include scenarios likely to generate defects.

## Equivalence Partitioning

The idea of **partitioning** is to break up something big into something small. For example, many large conference rooms can be partitioned with sliding walls, breaking one large ballroom into 2, 3, 4, or more smaller conference rooms. In the case of black-box testing, equivalence partitioning is breaking up our **equivalence** cases into groups that largely behave the same.

In our last unit, we considered **equivalence** test cases. However, within our **equivalence** test cases, we can have several groups of test cases that behave different. Imagine a situation where you are in charge of implementing the function `Math.abs(int x)`, which is the **absolute-value** function for integers.

Both `Math.abs(4)` and `Math.abs(-4)` are **equivalence** cases: we could just as typically call absolute value on a negative number as we could on a positive number. However, this doesn't change the fact that the inputs seem to *behave* differently. That is, how we expect absolute value called on *positive* numbers to act is different from how we would expect absolute value on *negative* numbers to act. So while both are *equivalence* cases, these two cases are not *equivalent.*

In this case, we can *partition* our input space into two groups: positive numbers and negative numbers. From there, we can say:

* absolute value basically acts the same for all positive inputs
* absolute value basically acts the same for all negative inputs

This tells us we want to write **at least one test** for each partition, but we do not necessarily need to write several tests for each partition. In fact, we can cover both partitions with just one test each:

`Math.abs(4)` - covers positive partition
`Math.abs(-5)` - covers negative partition

Think about it. If we know that `Math.abs(4)` is working, do we *need* to test `Math.abs(5)`, `Math.abs(6)`, `Math.abs(327)`? Is there any reason to believe those numbers behave differently? No. So additional tests like this end up being "testing for the sake of testing", which isn't progress.

---


## Boundary Test Cases

However, we want to pay special attention to the **boundary** cases. In our `Math.abs` partitions, notably absent is the number `0` (zero). Zero as a number is neither positive nor negative. In fact, zero sits **on the mathematical boundary** between these two partitions. As such, we would want to consider this boundary as a
special test case.

As such, our test plan now is 3 tests:

1. `Math.abs(4)`
2. `Math.abs(-5)`
3. `Math.abs(0)`

From here, we can write these test cases in JUnit.

---


## Multiple Inputs Examples

While this approach seems simple for `Math.abs`, it can get complicated. Consider the following class:

```java
public class StudentFinancialRecord {
	private final int studentID;
	private int classYear;
	private double overdue;
	private boolean isExempt;
	
	public StudentFinancialRecord(int studentID) {
		this.studentID = studentID;
		this.classYear = 1;
	}
    ...
    //all fields have getters, all fields except id have setters
```

Inside this class, we want to implement a method `double calculateBill(List<Integer> registeredCourseNumbers)`. 

Here is the specification for the method.

Follow the specification steps IN ORDER
1) First, calculate total:
   * All courses cost the same, and cost per course is determined by the number of courses taken
   * $8000/ course if less than 3 course 
   * $6000/ course if 3-6 course 
   * $5500/course if greater than 6 courses
2) Increase the value total by 10% if overdue is greater than 2000
3) Increase the value of the field overdue amount by 10% if **exempt** is false (this is done AFTER step 2)
   * if **exempt** is true, this penalty is waived, and overdue does not change.
4) Return the sum of total and overdue

Using this specification, let's design some test cases we can use to test our function as we write it.

---


### Inputs

What are the inputs to this function?

#### registeredCourseNumbers

Obviously the input `List<Integer> registeredCourseNumbers` is a parameter input. However, do the contents of the list matter? No! In this case, it turns out the only thing that matters is the size. We don't care about the actual numbers in the List. This means when generating our input for testing, we can use a private helper method:

```java
    private List<Integer> generateTestCourseNumberList(int length) {
		List<Integer> courseNumbersList = new ArrayList<>();
		for (int i = 0; i < length; i++){
			courseNumbersList.add(12345 + length);
		}
		return courseNumbersList;
	}
```

This method generates a list that "looks like" CRNs numbers (in my experience typically 5-digit numbers). The size of
the list is equal to the number of courses I need. *By realizing we only need to consider the size of the input list, and not the lists specific contents, we have made our tests easier to design*.

Focusing only on the size of this List, what are our **equivalence partitions**? Well, focusing on bullet point 1 of the specification, we have 3 partitions. For convenience, we'll call them **"Low"**, **"Mid"**, and **"High"**:
1. **"Low"** - 1 or 2 courses - we'll use 2
2. **"Mid"** - 3 to 6 courses, inclusive - we'll use 5
3. **"High"** - 7 or more courses - we'll use 8

Since we have 3 partitions here, we want to test each partition **at least once**.

#### What fields are included in our inputs?

Our class has four fields: `id`, `classYear`, `overdue`, `isExempt`. Are all of the fields relevant inputs to our function? Well, if we look at the specification, clearly we are referencing the values of `overdue` and `isExempt`. From this, we can assume these two fields are inputs to our function, since their values will affect how the function behaves. Note that these fields are inputs due to controlling the state of our class, specifically the parts of the state of the class relevant to the `calculateBill` function. You'll also notice, however, that `id` and `classYear` are not relevant to our `calculateBill` method. These two fields do not impact the function behavior or output. As such, we **do not** consider `id` and `classYear` inputs.

#### `overdue`

What are our partitions of `overdue`? If we look at bullet point 2 of the specification, we see that the function behaves different if the value of `overdue` is greater than 2000 (specifically, it causes us to change the value of total, which directly affects the function output). So, we have two partitions to consider:

1. "Small" - `overdue` > 2000 - we'll use 2500
2. "Big" - `overude` <= 2000 - we'll use 1500

#### `isExempt`

The boolean value `isExempt` will determine whether or not we apply interest to the `overdue` amount when `calculateBill` is called. Unsurprisingly, our boolean value has two partitions:

1. "YesExempt" - true
2. "NoExempt" - false

---


### Combining our partitions

In total, we now have several partitions for each field:

* `registeredCourseNumbers` - **3 partitions** - "Low", "Mid", "High"
* `overdue` - **2 partitions** - "Small", "Big"
* `isExempt` - **2 partitions** - "YesExempt", "NoExempt"

Now, as a **bare minimum**, we want to test every individual partition at least once. Given this, what is the **bare minimum** number of tests we need?

We only need 3. Each test will test a different partition of `registeredCourseNumbers`, while at least one test will each partition of `overdue` and `exempt`. We can list our tests in a table:

| Test# | courseCount | overdue | exempt |
|-------|-------------|---------|--------|
| 1     | 2           | 2500    | false  |
| 2     | 5           | 1500    | true   |
| 3     | 8           | 2500    | false  |

Now, you may have combined partitions differently, but we will start working with these tests in this example.

### A warning about this approach

I highlighted **bare minimum** in the last second because this should be seen as the bare minimum. A fault with this approach is assuming that the values of each input are independent of one another. However, **certain combinations of partitions** may be specifically important! When reading a specification, you should consider how partitions relate to one another. In fact, we will show in our next unit how these tests are insufficient. Just keep that in the back of your mind for now.

---


## Next Steps

Be aware that as of yet, we have not written `calculateBill` or any JUnit tests. This is intentional! This is
because we are practicing **Test Driven Development**. 

**Continued in the next module**
