---
Title: Test Driven Development
---

* TOC
{:toc}

# Test Driven Development

For this module, we are going to write the tests **before** we implement the method. This practice is called Test-Driven Development, or TDD. While we will go into more specifics later, the primary idea of TDD is to write tests *first* that describe the specification **first**, and only implement the method after the code is written. There are a number of advantages to this approach, key among them being we know we're done developing when all our tests are passing.

---


## Our Test plan so far

As of yet, we haven't written any code. Instead, we are still designing our test plan.

Last time, we used the Black-Box test plan strategy of **equivalence partitioning** to design the three following unit tests.

| Test# | courseCount  |  overdue  |  exempt  |
|:------|:------------:|:---------:|:--------:|
| 1     |      2       |   2500    |  false   |
| 2     |      5       |   1500    |   true   |
| 3     |      8       |   2500    |  false   |

---


## Calculated expected return values

We now want to break out a calculator, maybe some pencil and paper, and work out what these functions should return. We want to be **very careful** during this step, as if we calculate the values incorrectly, it will cause our tests to produce incorrect output, and potentially lead us to breaking working code, or missing a defect because our test is unsound.

| Test# | courseCount | overdue | exempt | expected return |
|-------|-------------|---------|--------|-----------------|
| 1     | 2           | 2500    | false  | 20350           |
| 2     | 5           | 1500    | true   | 31500           |
| 3     | 8           | 2500    | false  | 51150           |
{:.mbtablestyle}


### Other output?

In addition to our return value, does anything else change as a result of our method call? Yes! The state of the variable `overdue` can change. Specifically in bullet point 3:

> 3) Increase the value of the field overdue amount by 10% if **exempt** is false (this is done AFTER step 2)
  >    * if **exempt** is true, this penalty is waived, and overdue does not change.

This means in addition to our return value, the ending value of `overdue` is also part of the output of this method.

| Test# | courseCount | overdue | exempt | exp. return | exp. overdue |
|-------|-------------|---------|--------|-------------|--------------|
| 1     | 2           | 2500    | false  | 20350       | 2750         |
| 2     | 5           | 1500    | true   | 31500       | 1500         |
| 3     | 8           | 2500    | false  | 51150       | 2750         |
{:.mbtablestyle}


As a result, in Test 1 and Test 3, we expect the value of overdue to change, while in test 2 we expect it to remain the same.

---


## Writing a Stub

Before we can write our tests, we need to write the method we are testing as a Stub. A stub is a method
that is intentionally incomplete. It is effectively a placeholder method signature that allows us to write
our tests so that they compile. But because we are doing TDD, we want to write our tests before we actually
implement `calculateBill`

```java
    public double calculateBill(List<Integer> registeredCourseNumbers) {
        return 0.0; //TODO: Stub
    }
```

## Writing our first test

Now, we can write Test 1. First, we need to create an instance of the class `StudentFinanicalRecord` to test with, and then we need to configure the object into it's starting state.

```java
    @Test 
    public void StudentFinancialRecordtestCalculateBill_LowCourse_BigOverdue_NoExempt() {
        //setup test object
        record = new StudentFinancialRecord(1);
        record.setOverdue(2500);
        record.setExempt(false);
	}
```

Now, we can call our the method we are testing, `calculateBill`, and check if the return value matches are expected.

```java
    @Test 
    public void StudentFinancialRecordtestCalculateBill_LowCourse_BigOverdue_NoExempt() {
        record = new StudentFinancialRecord(1);
        record.setOverdue(2500);
        record.setExempt(false);
        assertEquals(20350, record.calculateBill(generateTestCourseNumberList(2)), 1e-4);
	}
```

Remember that when comparing doubles, we have to allow for some tolerance factor. In this case, I'm using 1 ten-thousandth.

Finally (**don't forget our other output**), we can add our assertion that checks that the value of `overdue` either changed for stayed the same correctly.

```java
    @Test 
    public void StudentFinancialRecordtestCalculateBill_LowCourse_BigOverdue_NoExempt() {
        record = new StudentFinancialRecord(1);
        record.setOverdue(2500);
        record.setExempt(false);
        assertEquals(20350, record.calculateBill(generateTestCourseNumberList(2)), 1e-4);
        assertEquals(2750, record.getOverdue(), 1e-4);
    }
```

We can write our other two tests this way. 

---


### Warning - TDD Misuse!

A quick note that, generally, in Test Driven Development standard practice, you are supposed to write **one test** at a time, and for each test write "just enough" code to make the last test pass. For now, though, let's assume we have written all three tests. We will go over more rigorous practice in the TDD Workflow Unit.

From there, we can **run** our tests and...they all fail! Which shouldn't surprise us. Remember, we haven't implemented our method yet. So, our current test table is:

| Test# | exp. return | exp. overdue | act. return | act. overdue |
|-------|-------------|--------------|-------------|--------------|
| 1     | 20350       | 2750         | 0           | 2500         |
| 2     | 31500       | 1500         | 0           | 1500         |
| 3     | 51150       | 2750         | 0           | 2500         |
{:.mbtablestyle}

Note that for space, I have removed the inputs from this and future tables. However, the inputs of each test have not changed and will not change for the remainder of this module.

From there, we can implement the function. Let's say we wrote the following:

```java
    public double calculateBill(List<Integer> registeredCourseNumbers) {
		int courseCount = registeredCourseNumbers.size();
		double total = 0;
		if (courseCount < 3) {
			total = 8000 * courseCount;
		} else if (courseCount >= 3 && courseCount <= 6) {
			total = 6000 * courseCount;
		} else {
			total = 5500 * courseCount;
		}
		if (overdue <= 2000 && isExempt) {
			return total + overdue;
		} else if (overdue > 2000) {
			if (isExempt) {
				return overdue * 1.1 + total;
			} else {
				return (total + overdue) * 1.1;
			}
		} else {
			return total + overdue * 1.1;
		}
	}
```
Please be aware that the above code is intentionally written in a way to be confusing. This way, we have to rely on our tests to tell us if it works or not! Also, just like any paper, the first-draft of any method is typically going to be ugly and hard to read. We'll edit this later in our Code Quality unit.

Now, we run our tests, and we get:

| Test# | exp. return | exp. overdue | act. return | act. overdue | result |
|-------|-------------|--------------|-------------|--------------|--------|
| 1     | 20350       | 2750         | 20350       | 2500         | FAIL   |
| 2     | 31500       | 1500         | 31500       | 1500         | PASS   |
| 3     | 51150       | 2750         | 51150       | 2500         | FAIL   |
{:.mbtablestyle}

Oh no! Two of our tests failed! Why did this happen! Well, if we look, our actual return matches are expected return in all three tests. However, anytime we expected overdue to change, it didn't! You realize that while you multiplied `overdue` by `1.1` when you were supposed to, you never actually stored the new value of `overdue`. So,
you write a quick fix, simply copying and pasting `overdue = overdue * 1.1;` before each time you did the multiplication, and then delete the `* 1.1` in the return statement.

```java
    public double calculateBill(List<Integer> registeredCourseNumbers) {
		int courseCount = registeredCourseNumbers.size();
		double total = 0;
		if (courseCount < 3) {
			total = 8000 * courseCount;
		} else if (courseCount >= 3 && courseCount <= 6) {
			total = 6000 * courseCount;
		} else {
			total = 5500 * courseCount;
		}
		if (overdue <= 2000 && isExempt) {
			return total + overdue;
		} else if (overdue > 2000) {
			if (isExempt) {
				overdue = overdue * 1.1;
				return overdue + total;
			} else {
				overdue = overdue * 1.1;
				return (total + overdue);
			}
		} else {
			overdue = overdue * 1.1;
			return total + overdue;
		}
	}
```

And run your tests again, and you get:

| Test# | exp. return | exp. overdue | act. return | act. overdue | result |
|-------|-------------|--------------|-------------|--------------|--------|
| 1     | 20350       | 2750         | 18750       | 2750         | FAIL   |
| 2     | 31500       | 1500         | 31500       | 1500         | PASS   |
| 3     | 51150       | 2750         | 46750       | 2750         | FAIL   |
{:.mbtablestyle}

Oh no! We fixed the overdue issue, but broke the return value! Well, the culprit is actually when we changed:

```java
        } else {
            return (total + overdue) * 1.1;
        }
```

into

```java
        } else {
            overdue = overdue * 1.1;
            return (total + overdue);
        }
```

Because we didn't think carefully while changing our code, we made a mistake. While `overdue` is correctly, `total` is no longer increasing by 10% as specified! So, we make one more change:

```java
    public double calculateBill(List<Integer> registeredCourseNumbers) {
		int courseCount = registeredCourseNumbers.size();
		double total = 0;
		if (courseCount < 3) {
			total = 8000 * courseCount;
		} else if (courseCount >= 3 && courseCount <= 6) {
			total = 6000 * courseCount;
		} else {
			total = 5500 * courseCount;
		}
		if (overdue <= 2000 && isExempt) {
			return total + overdue;
		} else if (overdue > 2000) {
			if (isExempt) {
				overdue = overdue * 1.1;
				return overdue + total;
			} else {
				overdue = overdue * 1.1;
				total = total * 1.1;
				return total + overdue;
			}
		} else {
			overdue = overdue * 1.1;
			return total + overdue;
		}
	}
```

And now we run our tests:

| Test# | exp. return | exp. overdue | act. return | act. overdue | result |
|-------|-------------|--------------|-------------|--------------|--------|
| 1     | 20350       | 2750         | 20350       | 2750         | PASS   |
| 2     | 31500       | 1500         | 31500       | 1500         | PASS   |
| 3     | 51150       | 2750         | 51150       | 2750         | PASS   |
{:.mbtablestyle}

All right! Now our three of our tests are passing!

![](../images/black-box/anakin_padme.png)

Remember, **tests only help us find defects and improve our confidence.** Tests cannot prove the absence of bugs.
In fact, in the next unit, we may find some defects in this code...

*To be continued...*