---
Title: Black-Box Testing
---

# Black-Box Testing

The idea of Black-Box testing is that we test according to the *specification* of the modules we are testing (typically a function). We are testing the **interface**, not the **implementation**.

## Strategies

We will look at a couple flawed approaches to Black-Box testing first, and then consider **Equivalence Partitioning** as well as **Boundary and Exception** test cases.

### Exhaustive Testing

Exhaustive Testing is almost never feasible. Consider, for example, the function `max(int a, int b, int c)` for our previous modules. The `int` datatype supports 2<sup>32</sup> unique values. This means, to test exhaustively, we would have to test all combinations of ints, or (2<sup>32</sup>)<sup>3</sup>, which equals 2<sup>96</sup>. Just to store these test input numbers would use up approximately **3 billion times** the expected global computer storage of the entire world in 2025. And compared to data types like Strings and Lists, ints are downright simple at only 4 bytes.

### Random Testing

One idea is to pick inputs at random, then working out the expected output based on the random inputs generated. Whether we do this using an actual random number generator, or picking numbers out of thin air. A problem with this approach is that this can easily miss important test cases we should consider. For example, testing our 3-argument `max` function with 3 equal values is obviously an interesting case worth examining. However, if we are using random generation, the odds of generating this test are `n`/2<sup>64, where `n` is the number of tests we generate. Those odds are incredibly small. Instead, we want a systematic way to find a set of test scenarios that will include scenarios likely to generate defects.

## Equivalence Partitioning

The idea of **partitioning** is to break up something big into something small. In the case of black-box testing, equivalence partitioning is breaking up our **equivalence** cases into groups that largely behave the same.

In our last unit, we considered **equivalence** test cases. However, within our **equivalence** test cases, we can have several groups of test cases that behave different. Imagine a situation where you are in charge of implementing the function `Math.abs(int x)`, which is the **absolute-value** function for integers.

Both `Math.abs(4)` and `Math.abs(-4)` are **equivalence** cases: we could just as typically call absolute value on a negative number as we could on a positive number. However, this doesn't change the fact that the inputs seem to *behave* differently. That is, how we expect absolute value called on *positive* numbers to act is different from how we would expect absolute value on *negative* numbers to act. So while both are *equivalence* cases, these two cases are not *equivalent.*

In this case, we can *partition* our input space into two groups: positive numbers and negative numbers. From there, we can say:

* absolute value basically acts the same for all positive inputs
* absolute value basically acts the same for all negative inputs

This tells us we want to write **at least one test** for each partition, but we do not necessarily need to write several tests for each partition. In fact, we can cover both partitions with just one test each:

`Math.abs(4)` - covers positive partition
`Math.abs(-5)` - covers negative partition

Think about it. If we know that `Math.abs(4)` is working, do we *need* to test `Math.abs(5)`, `Math.abs(6)`, `Math.abs(327)`? Is there any reason to believe those numbers behave differently? No. So additional tests like this end up being "testing for the sake of testing", which isn't progress.

## Boundary Test Cases

However, we want to pay special attention to the **boundary** cases. In our `Math.abs` partitions, notably absent is the number `0` (zero). Zero as a number is neither positive nor negative. In fact, zero sits **on the mathematical boundary** between these two partitions. As such, we would want to consider this boundary as a
special test case.

As such, our test plan now is 3 tests:

1. `Math.abs(4)`
2. `Math.abs(-5)`
3. `Math.abs(0)`

From here, we can write these test cases in JUnit.

