---
Title: Testing With Exceptions
---

# Testing with Exceptions

When following best practices of defensive programming, we often *want* to throw Exceptions for incorrect or invalid inputs. In this module, we will look at writing tests that *expect* an `Exception` to be thrown.

---

* TOC
{:toc}

---

Consider testing our `withdraw` function specification.

```java
    public void withdraw(double amount) { 
        //TODO: Stub
    }
```

We want to withdraw `amount` from our `BankAccount`. If the transaction completes successfully, `balance` in `BankAccount` will be reduced by `amount`. However, `amount` cannot be negative, and `amount` cannot exceed balance.

In this case, we have two preconditions:

* `amount` cannot be negative
* `amount` cannot exceed `balance`

If either condition is violated, we should throw an Exception. We can write tests
to ensure these Exceptions are thrown!

We want to test and implement our *equivalence* case first, so we write the following test.

```java
import org.junit.jupiter.api.*;

import static org.junit.jupiter.api.Assertions.*;

public class BankAccountTest {
    private BankAccount testAccount;
    
    @BeforeEach
    public void setup() {
        testAccount = new BankAccount(1234, 500);
    }
    
    @Test
    public void withdrawEquivalance() {
        testAccount.withdraw(300);
        assertEquals(200, testAccount.getBalance(),1e-4);
    }
}
```

We then implement our equivalence case:

```java
    public void withdraw(double amount) {
        balance -= amount;
    }
```

We run our test and it passes. Now we can test our **Exception** cases.

## AssertThrows

The `assertThrows` function is a JUnit 5 assertion that says "the code must throw a particular exception." For example, let's test the exception case `amount` cannot exceed `balance`. I would expect that if I tried to withdraw more money than was in the account, I would get some kind of `RuntimeException`. And so I write:

```java
    @Test
    public void withdrawInsufficientFundsException() {
        assertThrows(RuntimeException.class, () -> testAccount.withdraw(600));
    }
```


What this code is saying is that I expect the code `testAccount.withdraw(600)` to result in a 
`RuntimeException` being thrown. The rest of the line (`.class` and `() ->`) are syntax items. We use `RuntimeException.class` because Java needs to know what `class` of Exception is returned. The `() ->` relates to lambda bodies in Java, and we will discuss this during our [Functional Programming](https://sde-coursepack.github.io/modules/refactoring/Functional-Programming/) unit. However, for now, just assume that it needs to be there.

### Passing our test

I add the following to the implementation.

```java
    public void withdraw(double amount) {
        if (amount > balance) {
            throw new RuntimeException(getInsufficientFundsMessage(amount));
        }
        balance -= amount;
    }

    private String getInsufficientFundsMessage(double amount) {
        return "Error: insufficient funds in account #" + id + " - balance: " + balance +
        " for transaction amount: " + amount;
    }
```

Here, I am simply using the function `getInsufficientFundsMessage(double amount)` to separate throwing the exception from generating the exception message. This keeps `withdraw` shorter and more tightly focuses on the mechanisms of withdrawing from a bank account.

### Refining our test

Remember that in our [Defensive Programming](https://sde-coursepack.github.io/modules/refactoring/Defensive-Programming/) unit, we wanted to make sure that if an exception was thrown, we didn't change the state of the object in question. As such, we want to ensure that after the `RuntimeException` is thrown, the value of `balance` does not change, since no transaction should have been allowed. As such, we add to our test:

```java
    @Test
    public void withdrawInsufficientFundsException() {
        assertThrows(RuntimeException.class, () -> testAccount.withdraw(600));
        assertEquals(500, testAccount.getBalance(),1e-4);
    }
```

We run our test, and it passes.

### Using precise exceptions

In our last unit, we talked about creating an Exception called `InsufficientFundsException`

```java
public class InsufficientFundsException extends RuntimeException {
    public InsufficientFundsException(String message) {
        super(message);
    }
}
```

The point of this was to ensure that the error message thrown by `withdraw` due to incorrect usage provides the best information as to:

* Why the exception occurred  
* How to avoid it in the future  


As such, we change our generic `RuntimeException` in `withdraw` as well as `withdrawInsufficientFundsException` to the more precise `InsufficientFundsException`.

```java
    public void withdraw(double amount) {
        if (amount > balance) {
            throw new InsufficientFundsException(getInsufficientFundsMessage(amount));
        }
        balance -= amount;
    }
```

```java
    @Test
    public void withdrawInsufficientFundsException() {
        assertThrows(InsufficientFundsException.class, () -> testAccount.withdraw(600));
        assertEquals(500, testAccount.getBalance(),1e-4);
    }
```

We can then add our test for the `NegativeBalanceException` violation in the same way.

First, we write our test:

```java
    @Test
    public void withdrawNegativeTransaction() {
        assertThrows(NegativeTransactionException.class, () -> testAccount.withdraw(-300));
        assertEquals(500, testAccount.getBalance(), 1e-4);
    }
```

And then we add to our existing `withdraw` implementation.

```java
    public void withdraw(double amount) {
        if (amount < 0) {
            throw new NegativeTransactionException(getNegativeTransactionMessage("withdraw", amount));
        } else if (amount > balance) {
            throw new InsufficientFundsException(getInsufficientFundsMessage(amount));
        }
        balance -= amount;
    }
```

We run all of our tests (our equivalence and our two exception cases), and they pass, so we're good!

## The Importance of Rolling Back

We must always make sure that when exceptions are thrown, we do not create any unintended side effects.

Consider the following snippet of a class called `VoteTally` and specifically the function `addVotesFromPrecinct`:

```java
public class VoteTally {
    private final Map<String, Integer> candidateVotes;

    public VoteTally() {
        this.candidateVotes = new HashMap<>();
    }

    protected VoteTally(Map<String, Integer> candidateVotes) {
        this.candidateVotes = candidateVotes;
    }

    public int getNumCandidates() {
        return candidateVotes.size();
    }

    public Set<String> getCandidates() {
        return candidateVotes.keySet();
    }

    public int getVotesForCandidate(String candidate) {
        if (!candidateVotes.containsKey(candidate)) {
            return 0;
        }
        return candidateVotes.get(candidate);
    }

    /**
     * Adds a number of votes for each candidate.
     * @param precinctResults - the results from a precinct
     */
    public void addVotesFromPrecinct(Map<String, Integer> precinctResults) {
        for (int newVotes : precinctResults.values()) {
            if (newVotes < 0) {
                throw new IllegalArgumentException("A precinct cannot report negative votes for a candidate");
            }
        }
        for (var candidate : precinctResults.keySet()) {
            var newVotes = precinctResults.get(candidate);
            var currentVotes = getVotesForCandidate(candidate);
            candidateVotes.put(candidate, newVotes + currentVotes);
        }
    }
}
```

An example of a test for `addVotesFromPrecinct` could be:

```java
    @Test
    void addVotes_existingCandidates() {
        var testVoteTally = new VoteTally(
                new HashMap<>(Map.of("John Smith", 20, "Votey McVoteface", 10)));

        testVoteTally.addVotesFromPrecinct(Map.of("John Smith", 10, "Jane Doe", 15));

        assertEquals(3, testVoteTally.getNumCandidates());
        assertTrue(testVoteTally.getCandidates().contains("John Smith"));
        assertTrue(testVoteTally.getCandidates().contains("Jane Doe"));
        assertTrue(testVoteTally.getCandidates().contains("Votey McVoteface"));
        assertEquals(30, testVoteTally.getVotesForCandidate("John Smith"));
        assertEquals(15, testVoteTally.getVotesForCandidate("Jane Doe"));
        assertEquals(10, testVoteTally.getVotesForCandidate("Votey McVoteface"));
```

Let's also say we have the following `Exception` test (this test is insufficient as we'll show in a second).

```java
    @Test
    void addVotes_negativeVotes_Exception() {
        var testVoteTally = new VoteTally(
                new HashMap<>(Map.of("John Smith", 20, "Votey McVoteface", 10)));
        assertThrows(IllegalArgumentException.class, () ->
            testVoteTally.addVotesFromPrecinct(Map.of("Jane Doe", 5, "John Smith", -10)));
    }
```

Our test (successfully) expects an `IllegalArgumentException` to be thrown when a negative vote total for a candidate is found, since this shouldn't be possible.

But let's say when a junior developer reads the code for the function `addVotesFromPrecinct`, they might think the first loop is redundant. They think, "let's remove that unnecessary loop and just check for negative votes in the loop that already exists, so the code is easier to understand."

And so they change the code to this:

```java
    public void addVotesFromPrecinct(Map<String, Integer> precinctResults) {
        for (var candidate: precinctResults.keySet()) {
            var newVotes = precinctResults.get(candidate);
            if (newVotes < 0) {
                throw new IllegalArgumentException("A precinct cannot report negative votes for a candidate");
            }
            var currentVotes = getVotesForCandidate(candidate);
            candidateVotes.put(candidate, newVotes + currentVotes);
        }
    }
```

They run our test, it passes, so they think, "Job well done!"

**They have just created a serious bug!** This is because our test **doesn't check the expected post-conditions!**

Specifically, let's add a print-statement to our test to see what happens:

```java
    @Test
    void addVotes_negativeVotes_Exception() {
            var testVoteTally = new VoteTally(
            new HashMap<>(Map.of("John Smith", 20, "Votey McVoteface", 10)));
        assertThrows(IllegalArgumentException.class, () ->
        testVoteTally.addVotesFromPrecinct(Map.of("Jane Doe", 5, "John Smith", -10)));

        System.out.println(testVoteTally);
        }
```

What prints is tells us the candidates have the following votes:

```text
John Smith           |     20
Votey McVoteface     |     10
Jane Doe             |      5
```
(Note that, even worse, this will only happen *some* of the time, depending on how Java builds the test Map at runtime! Other times, it will appear to work fine)

Jane Doe was still added! That's because on the first iteration through our loop, we add Jane Doe, **before** we find the bad input (John Smith's -10 votes). This means while we correctly throw an exception and do not add John Smith's negative 10 votes, we have already erroneously and permanently added Jane Doe's votes!

This is why the test we had was insufficient. A better test would be:

```java
@Test
    void addVotes_negativeVotes_Exception() {
        var testVoteTally = new VoteTally(
                new HashMap<>(Map.of("John Smith", 20, "Votey McVoteface", 10)));
        assertThrows(IllegalArgumentException.class, () ->
            testVoteTally.addVotesFromPrecinct(Map.of("Jane Doe", 5, "John Smith", -10)));

        assertEquals(2, testVoteTally.getNumCandidates());
        assertTrue(testVoteTally.getCandidates().contains("John Smith"));
        assertFalse(testVoteTally.getCandidates().contains("Jane Doe"));
        assertTrue(testVoteTally.getCandidates().contains("Votey McVoteface"));
        assertEquals(20, testVoteTally.getVotesForCandidate("John Smith"));
        assertEquals(10, testVoteTally.getVotesForCandidate("Votey McVoteface"));
    }
```

Now, our test will fail if Jane Doe's votes are erroneously added! But because our test that *was* in place didn't check the post-conditions and just assumed "Exception means pass," we failed to catch this bug being injected!

This is also why we always want to explicitly check pre-conditions first, and separate that logic entirely from the actual running of the method!

```java
public void addVotesFromPrecinct(Map<String, Integer> precinctResults) {
        for (int newVotes : precinctResults.values()) {
            if (newVotes < 0) {
                throw new IllegalArgumentException("A precinct cannot report negative votes for a candidate");
            }
        }
        
        for (var candidate: precinctResults.keySet()) {
            var newVotes = precinctResults.get(candidate);
            var currentVotes = getVotesForCandidate(candidate);
            candidateVotes.put(candidate, newVotes + currentVotes);
        }
    }
```

This can never produce the bug, because we would have failed before any existing data was changed/mutated.

