---
Title: Test Plans
---

# Test Plans

While we write each test one at a time, overall we want to ensure that our tests collectively give us confidence in our code's correctness.

For example, consider a function `public boolean isOdd(int x)`.
Would it be sufficient for us to only test this function once?

`assertTrue(isOdd(1))`

No! Of course not! We want to test this function both with
odd and even inputs! Additionally, we might want to test zero,
as well as negative numbers, to ensure our code is behaving
correctly.

So, collectively, we might test with inputs like [1, 2, 0, -4, -5], expecting outputs of [true, false, false, false, true] respectively.

This collection of tests forms a **Test Plan**.

---


## How many tests?

How much do we need to test each function? The answer is not obvious.

In fact, there is **no perfect number** of tests to write for any hypothetical function. Instead, we want to use a systematic approach so that we write *enough* tests that are diverse enough to give us broad coverage of our input space.

---


## Types of Tests

Consider the class [`MySortedList`](https://github.com/sde-coursepack/TestingIntro/blob/main/src/main/java/edu/virginia/cs/testingintro/MySortedList.java) we have looked at before. Specifically, let's consider the functions `get(int index)`, `add(int value)`, and `contains(Object target)`. Generally, we want our test plan to contain at least some tests from each type.

---


### Equivalence Tests

Equivalence test cases are "typical" test cases. That is, test cases
that show how the software behaves under normal, predictable conditions.

For example, imagine we have a `MySortedList` instance like [1, 3, 5, 7]. In this situation, the following could be considered examples of equivalence cases:

* `add(6)` - we're adding into the middle of the list between 5 and 7. Statistically, if we add random values, these values will typically go somewhere in the middle of the list.
* `get(1)` - getting a value stored at a particular index. In this case, index 1 (the value 3)
* `contains(3)` - does the list contain a 3? Yes it does. Return true

Each of these cases generally show the basic, most common operation of the function we are testing.

---


### Boundary

Boundary cases are typically cases where there is something interesting or unique about the test case to seperate it from an equivalence case. For example:

* `add(6)` called on an empty instance of `MySortedList` - adding to an empty list, depending on how we construct our data structure, could pose an interesting challenge, so we'll want to ensure it's behaving correctly.
* `add(6)` to an instance of `MySortedList` that already contains a `6`; it's worth seeing how this is handled, to ensure that we can add multiple of the same element to our object and it still behave as intended.
* `get(4)` called on an instance of `MySortedList` with 5 elements. In effect, getting the *last* element is interesting enough to warrant special consideration. Similarly, `get(0)`, or get the first element, can also be interesting.
* `contains(3)` on an empty instance of `MySortedList` is also interesting, as we want to return false. However, empty collections can always pose an issue as a crash risk.

---


#### contains quandry

Once again, imagine we have a `MySortedList` instance like [1, 3, 5, 7]. And to test `contains`, you use `contains(4)`.

Is this an **Equivalence** or **Boundary** test case? Take a second and think about the definitions and come to your own conclusion before proceeding.

---

In my opinion, this is clearly an **equivalence** case. That is, just because the function returns `false` doesn't mean it's a boundary case. Instead, we would consider this as in a separate *equivalence partition*, or group of inputs that behave the same way.

For example, consider the function `isOdd(int x)` from earlier. `isOdd(5)` should return `true`, and `isOdd(6)` should return `false`. But is either case a **boundary** case? No! Both are completely typical uses cases of our `isOdd` function.

---


#### What about find?

Now consider the function `find(int target)`. In our same `MySortedList` instance where the list is [1, 3, 5, 7], `find(3)` would be an example of an **equivalence** case.

But now consider `find(4)`. Once again, take a minute and think about it, and decide if you think this is an **equivalence** or **boundary** test case.

In this case, this is now a **boundary** test case. The reason is that `find` is based on `indexOf` in the Collections framework. `find` should return the index of the first element equal to target. However, if the element is not found in the list at all, then the function should return `-1`. The value `-1` is an example of a **sentinel value**. That is, the value returned doesn't literally mean "the item is at index -1", but rather -1 acts as a form of "special value", meaning "target not found." In general, we should actually avoid using sentinel values in our code, but `indexOf` returning -1 meaning "not found" is traditional and well understood, so it's still around.

It may seem odd that we consider `contains(4)` in the above case an **equivalence** case, while `find(4)` is a boundary case. In truth, there isn't a clear black-and-white line between the two cases. To some extent, whether a case is equivalence or boundary can be subjective. In either case, it depends *heavily* on the context of the specific function.

---


### Exception

Exception test cases are test cases that cannot be meaningfully executed correctly, and should throw Exceptions. For example:

* `get(2)` called on a `MySortedList` instance of size 2. In this case, the index 2 is **out of bounds**, and so we should expect an IndexOutOfBoundsException to be thrown.

On the other hand, `contains`, `find`, and `add` have no exception cases. This is because these functions should never throw an exception with valid syntax.

---


### Robustness

Robustness test cases are cases that are **syntactically valid** but
**semantically** meaningless. Often times in these cases, it may be unclear how they are intended to behave, and it may be worth checking the specification itself.

* `contains("Three")` is a syntactically valid way to call the `contains(Object target)` function. However, semantically, it wouldn't make sense to call `contains` with a String when our underlying ArrayList is made up of Integers. In general, Java Collections allow you to call contains on a Collection made up non-matching datatype. It just returns false. The metaphor I use is you can go to a grocery store and ask "Hey, do you have any Computers for sale?" The answer is obviously no because grocery stores sell food, not computers. But even though the answer is no, the question itself is still valid.

---


## What about when we don't know?

Consider we were writing a function on a class `BankAccount` that
withdrew funds, deducting the withdrawn amount from balance:

```java
    public void withdraw(double amount) {
        balance -= amount;
    }
```

Now consider calling this function with a negative input:

`withdraw(-50)`

What does this **mean?** Is this meant to deposit 50 dollars? Should this be accepted? In this situation, we may have found a *gap* in the specification and should work with our team to clarify the **intent** of the `withdraw` function. If may be that we should never allow a negative amount, at which point `withdraw(-50)` should be an **exception** case. OR, it may be that this is actually the intended way to handle depositing, at which point this would be an **equivalence** case. The point is: as written, with a specification that doesn't clarify intended behavior in this abnormal case, we cannot be certain.

---


## So how many tests do I need?

The next three modules should help us answer this question. Using the idea here, we will expand on our Test Plan analysis. Generally, we have two competing interests:

* We want to test our code thoroughly enough so that we are justifiably confident in its correctness
* We don't want to test **for the sake of testing**. Writing unnecessary tests that do not improve our confidence in the correctness of our code is a waste of time. We therefore want to reasonably limit the number of tests we write.

---


## "Perfect" Testing

The idea of a *perfect* test suite is ephemeral, and virtually impossible to define. It would be like asking you to imagine the perfect car. Not *your favorite car*, but the objective perfect car. Philosophically, it seems impossible, as the context of the situation around how we will use the car greatly influence our idea of perfection.

However, while I cannot confidently say what a perfect car is, I can point out faults in specific cars. For example, my first car was a 2000 Nissan Frontier pick-up truck I got from my father. This car had several notable flaws, making it hard for me as a grad student on a stipend:
* Shortly after I got it, a head gasket cracked on the car's engine, costing over $1500 to fix.
* The care got 17 miles to the gallon highway. At the time, I rarely drove it except to visit family. A round trip from South Bend, Indiana to Charleston, WV in early 2014 cost me nearly $200 just in gasoline alone.
* It was a pick-up truck with rear-wheel drive only. In the snow, since there was no weight on the rear tires, I had difficulty doing even basic driving. I was got stuck in my drive-way for 2 weeks after a 4-foot snow. My roommate had a Toyota Coralla that fared better.

The point is, while we can't easily say in the abstract what a perfect car is, we can point out flaws in a **specific** car, and note clear places where the car failed.

**We should approach test plans with the same idea.** We can't have a theoretically perfect test plan, but if we notice any glaring flaws in our test plan, we should work to fix them. If we find a bug that isn't accounted for in our tests, we should add a test to catch that bug! If we find an obvious missing test case, we should add it. If we notice a class has not been tested, we should test it!

Unlike cars, we can alter our test plan whenever we need. We don't have to just sell it and hope for a better car next time.