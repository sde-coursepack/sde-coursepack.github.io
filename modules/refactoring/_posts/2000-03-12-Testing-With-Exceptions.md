---
Title: Testing With Exceptions
---

* TOC
{:toc}

# Testing with Exceptions

When following best practices of defensive programming, we often *want* to throw Exceptions for incorrect or invalid inputs.

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
`RuntimeException` being thrown. The rest of the line (`.class` and `() ->`) are syntax items. We use `RuntimeException.class` because Java needs to know what `class` of Exception is returned. The `() ->` relates to lambda bodies in Java, and we will discuss this during our Functional Programming unit. However, for now just assume that it needs to be there.

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

Remember that in our Defensive Programming unit, we wanted to make sure that if an exception was throw, we didn't change the state of the object in question. As such, we want to ensure after the `RuntimeException` was thrown, that the value of `balance` did not change, since no transaction should have been allowed. As such, we add to our test:

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

