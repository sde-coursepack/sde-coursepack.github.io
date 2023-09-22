---
Title: Exceptions-Best Practices
---

# Exceptions

Exceptions, or exceptional events, are situations where our code cannot meaningfully proceed in its current state. For example, a `NullPointerException` means that you tried to call instance methods on a `null` object, which cannot have any instance methods.

---

* TOC
{:toc}

---

## When to throw exceptions

When designing our class, functions, and interfaces, we should use exceptions to enforce pre-conditions and post-conditions of our function. For instance, consider `isPrime` below.

```java
    public boolean isPrime(int number) {
        if (number <= 0) {
            throw new RuntimeException("isPrime can only be called positive inputs. Was called with " + number);
        }
        for (int i = 2; i <= Math.sqrt(number); i++) {
            if (number % i == 0) {
                return false;
            }
        }
        return true;
    }
```

Here, we are throwing an exception because we are saying, as a pre-condition, non-positive numbers cannot be prime or non-prime. Therefore, it doesn't make sense to return true/false in this situation, since the pre-condition is violated.

## When not to throw exceptions

You should never throw exceptions along with try-catch as a way of handling control-flow. For example, the following code is **bad** and should not be used:

```java
    //Don't do this!
    public <E> boolean isValidIndex(List<E> list, int index) {
        try {
            list.get(index);
            return true;
        } catch (IndexOutOfBoundsException e) {
            return false;
        }
    }
```

This is overly complicated and makes the code hard to read. Alternatively, you could simply write this as a normal boolean statement:

```java
    public <E> boolean isValidIndex(List<E> list, int index) {
        return 0 <= index && index < list.size();
    }
```

## When to use try-catch

Just because there is an Exception doesn't mean you should use a try-catch! In general, a try-catch should only be used when you can meaningfully handle an exception or when dealing with checked exceptions, turning a checked exception into an unchecked exception.

### You can handle exception

Consider the following Java code:

```java
public class UserInterface {
    Scanner sc = new Scanner(System.in);

    public int promptUserForNumber() {
        System.out.println("Enter a number 1 through 5");
        String userInput = sc.nextLine();
        return Integer.parseInt(userInput);
    }
}
```

The function `promptUserForNumber` could throw a `NumberFormatException` because of it's use of `Integer.parseInt`. However, we can handle this error! That is, if a user enters something that isn't a number, rather than crashing, we can simply tell the user to try again. Using a while loop, we can force this process to be repeated until the user's input is a valid integer.

```java
public class UserInterface {
    Scanner sc = new Scanner(System.in);

    public int promptUserForNumber() {
        while (true) {
            System.out.println("Enter a number 1 through 5");
            String userInput = sc.nextLine();
            try {
                return Integer.parseInt(userInput);
            } catch (NumberFormatException e) {
                System.out.println("Invalid Entry: You must enter a number. Try again!");
            }
        }
    }
}
```

In this case, we can handle the exception. In general, when dealing with live user input, you never want to allow the program to crash. Rather, alert the user if they make a mistake and let them try again or return to the menu they were last in. Above, we used try-catch blocks to help us handle that exception, so it doesn't crash our program!

### Never catch exceptions you can't handle

There's an asterisk above that we'll address with **checked** exceptions, but it is generally good advice.

Consider this function:

```java
    public String reverseString(String string) {
        StringBuilder reverseAccumulator = new StringBuilder();
        for (int i = string.length() - 1; i >= 0; i--) {
            reverseAccumulator.append(string.charAt(i));
        }
        return reverseAccumulator.toString();
    }
```

What if someone calls this function with `reverseString(null)`? Well, in this case, this would cause a `NullPointerException` crash at the for loop, specifically with `string.length()`. 

So you might be tempted to do something like:

```java
    public String reverseString(String string) {
        StringBuilder reverseAccumulator = new StringBuilder();
        try {
            for (int i = string.length() - 1; i >= 0; i--) {
                reverseAccumulator.append(string.charAt(i));
            }
            return reverseAccumulator.toString();
        } catch (NullPointerException e) {
            //do something...
        }
    }
```

But here's the question: **can we actually *do* anything with this**? Sure, we can catch the `NullPointerException`, but there's still nothing valid we can return here. Returning `null` wouldn't make sense, since you can't reverse `null` (remember, null is *nothing*). In fact, the most logical course of action would be to throw a `NullPointerException`. Except our code already does this when the input `string` is `null`!

So, it's best to leave this function **as-is** and not even try to handle the `NullPointerException`. 

### Checked Exceptions

One annoyance of the Java programming language is Checked Exceptions: that is, exceptions that Java forces you to either handle or throw. For example:

```java
    public String readTextFile(String filename) {
        try {
            BufferedReader bufferedReader = getBufferedReader(filename);
            return getFileContents(bufferedReader);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private BufferedReader getBufferedReader(String filename) throws FileNotFoundException {
        FileReader fileReader = new FileReader(filename);
        return new BufferedReader(fileReader);
        }
    
    private String getFileContents(BufferedReader bufferedReader) throws IOException {
        StringBuilder fileContents = new StringBuilder();
        for (String line = bufferedReader.readLine(); line != null; line = bufferedReader.readLine()) {
            fileContents.append(line);
        }
        return fileContents.toString();
    }
```

In the above, you can see that in `readTextFile(String filename)`, we have a try-catch block to handle any `IOException`, as we are syntactically forced to either handle or declare a throw in the method signature.

However, in `getBufferedReader(String filename)` and `getFileContents(BufferedReader bufferedReader)`, we have no try-catch block. Instead, we declare in the method signature what checked-exceptions these methods can throw.

#### Handle checked exceptions in one place

The reason I wrote this code this way is that while each of these methods *can* throw some kind of `IOException` (remember, `FileNotFoundException` is a sub-type `IOException`), we only want to actually handle the checked exception in one place. If we have try-catch blocks in every single function that interacts with this text file, our code gets dramatically harder to read and understand.

As such, as a general rule, anytime I write a method that requires handling checked exceptions, I try to do so with only 1 try-catch block for all possible checked exceptions.

#### Throw checked exceptions as unchecked exceptions

While there was good intent behind the idea of Checked Exceptions, ultimately they can be difficult to work with. For example, consider the client function in `Main` that calls `readTextFile(String filename)`. Do we want that function to have to worry about `FileNotFoundException`s or any other `IOException` issues? No! We want to encapsulate all the File I/O behavior and error handling in our function `readTextFile(String filename)`. 

As such, if we need to throw an Exception in `readTextFile`, we can catch the Checked Exception (`IOException`) and simply throw it as an unchecked `RuntimeException`. The specific error message of the `IOException` is preserved in the `RuntimeException` because we pass the original exception into the constructor of `RuntimeException`.

If a client *must* handle exceptions created by a called method, that increases *coupling*. However, we should only do this when the client can *meaningfully handle* the exception. In this case, can `Main` really **do** anything with `readTextFile` if the File doesn't exist? In this example, no. If, however, we rewrote main to handle, say, a `FileNotFoundException` (such as allowing a user to prompt a different filename), well, then we might use `throws FileNotFoundException` in the `readTextFile` signature.

So, for example, if I run this code with a file that doesn't exist:

```java
    public static void main(String[] args) {
        ExceptionExample ee = new ExceptionExample();
        String filename = "FileThatDoesntExist.txt";
        ee.readTextFile(filename);
    }
```

I still get a meaningful error message:

```shell
Exception in thread "main" java.lang.RuntimeException: java.io.FileNotFoundException: FileThatDoesntExist.txt (The system cannot find the file specified)
	at ExceptionExample.readTextFile(ExceptionExample.java:37)
	at ExceptionExample.main(ExceptionExample.java:57)
Caused by: java.io.FileNotFoundException: FileThatDoesntExist.txt (The system cannot find the file specified)
```

## Writing your own Exceptions

Writing your own Exceptions can be a valuable way to communicate information unique to your context. For example, we can use the method name to **communicate intent** of why an Exception was thrown.

For instance, if we were writing software for Bank transactions, we might want an Exception that communicate "Insufficient Funds". This way, when a transaction is rejected because of an Exception, that **reason why** the transaction was rejected is clear.

Writing your own Exception classes can be very simple:

```java
public class InsufficientFundsException extends RuntimeException {
    public InsufficientFundsException(String message) {
        super(message);
    }
}
```

In general, the only unique aspect here is the name of the Exception. It is worth noting that we extend the `RuntimeException` here. In general, this is a good class for your custom-made exceptions to extend. However, depending on the situation, you may want to extent more specific classes, like `IllegalArgumentException` when your exception is related to errors in function arguments.

Be aware that if you extend a checked exception, then your custom exception will also be a checked exception.

### Exception error messages

The error messages for Exceptions should also meaningfully communicate:

a) What caused the exception
b) How to avoid it in the future.

Consider a function that generates an error message for `InsufficientFundsException`

```java
    private String getInsufficientFundsMessage(BankAccount account, double amount) {
        return "Error: insufficient funds in account #" + id + " - balance: " + balance +
        " for transaction amount: " + amount;
    }
```

This function generates a String like:

`Error: insufficient funds in account #1234 - balance: 500.0 for transaction amount 700.0`

And so, if we `throw new InsufficientFundsException` with this message, we would see:

`Exception in thread "main" edu.virginia.cs.exceptions.InsufficientFundsException: Error: insufficient funds in account #1234 - balance: 500.0 for transaction amount 700.0`

This exception clearly communicates to whoever caused it what the problem is (performing a transaction with an account that cannot cover the amount of the transaction). They would like adapt by adding in their own check, something like:

```java
    try {
        account.withdraw(amount);
    } catch (InsufficientFundsException e) {
        screen.display("We're sorry, but you have insufficient funds for that transaction. The transaction has been canceled.");
    }   
```

However, it's important to note that handling the exception is the **client classes** responsibility. Throwing the exception to alert the client to mis-use is your classes responsibility.