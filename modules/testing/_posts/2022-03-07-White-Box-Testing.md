---
Title: White-Box Testing
---

# The Story so Far

In the last module we used Black-Box testing to select test cases for a function `calculateBill`.

The specification read:

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

In Black-Box testing, we select what test cases to write based on the specification. 

Our tests so far are:

| Test# | exp. return | exp. overdue | act. return | act. overdue | result |
|-------|-------------|--------------|-------------|--------------|--------|
| 1     | 20350       | 2750         | 20350       | 2750         | PASS   |
| 2     | 31500       | 1500         | 31500       | 1500         | PASS   |
| 3     | 51150       | 2750         | 51150       | 2750         | PASS   |

And good news, they're all passing now! However, in our haste to complete the test-writing from the last module, we *maybe* got a bit sloppy with our TDD practices. That is, we wrote some code that isn't actually checked by these tests.

Now that we have code that is written, we can use White-Box testing to ensure our **existing code** is reasonably tested.

# White-Box Testing

In **White-Box Testing** (sometimes called Glass-Box testing), we select cases while considering the existing **implementation** of the code. This is contrasted with **Black-Box Testing**, where we select test cases by considering the **interface** as specified for the method we are testing. As a note, because **White-Box Testing** relies on writing tests based on existing implementation, we actually cannot use White-Box testing as a starting point in Test-Driven Development. TDD is inherently **Black-Box**.

## Testing with Coverage

## Types of Coverage

### Statement of Coverage

### Branch Coverage

### Conditional Coverage

### Path Coverage