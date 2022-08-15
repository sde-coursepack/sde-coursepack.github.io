---
Title: Command Line Arguments
---

## **[Source Code Examples](https://github.com/sde-coursepack/commandline)**


# Command Line Arguments

This unit will teach you the basics of writing programs that use command line arguments.

## main(String[] args)

We've all typed it more times than we can count:

```public static void main(String[] args)```

This is the method that is automatically called when you run a Java program. We know that the method is `public`
because it has to be called externally. It's `static` because the method is called without needing to make an instance
of the class the main method is in. It's `void` because the method doesn't return anything. The method must be
called `main` because that's the name of the method Java will look for when we run.

But what about `String[] args`? Obvious this is an Array of String objects with the variable name `args`. But barring
making you think about the noises angry pirates make, what does it do?

Well, this array is use for **command-line arguments**.

Let's consider an example of a program we run when making jar files:

```jar cfe MyJar.jar example.Main example/*.class```

In this case, `jar` is the name of a program. And just like a function with input parameters, this function can take
in arguments. In this case, the arguments are `cfe MyJar.jar example.Main example/*.class`

Well, when the program is run, Java converts this text into an array of `String` objects (`String[]`). If we assume the
variable name is `args`, then we get:

* `args[0]` &#8594; `"cfe"`
* `args[1]` &#8594; `"MyJar.jar"`
* `args[2]` &#8594; `"example.Main"`
* `args[3]` &#8594; `"example/*.class"`

You'll notice that Java splits the arguments up by whitespace, and we start with index `0` being the first argument
**after the program name**. To wit, `jar` above is not included in the command-line arguments array.

Now, it should be noted that we need to run our programs with something like:

`java myPackage.myClassName`

But from there, we can add arguments the same way. For example:

`java myPackage.myClassName these are my lovely arguments`

And we get:

* `args[0]` &#8594; `"these"`
* `args[1]` &#8594; `"are"`
* `args[2]` &#8594; `"my"`
* `args[3]` &#8594; `"lovely"`
* `args[4]` &#8594; `"arguments"`

### Package Reminder

Reminder that when we want to run class files from command line, we need to include the package name. Refer back
to the module on packages if you feel you still need a reminder.

### Why do we want these?

Command-line arguments give us a way for users to add information to the program without having to edit
the source code of the program, and without us having to worry about dealing with something like `System.in`.
This is important, because we may be writing a program for something that doesn't have an easily accesible user console.
In fact, *our "users" may not even be human at all.* Many large software suites rely on programs communicating directly,
which may often by done *by* command line arguments.

## String Arguments
Example: [HelloWho.java](https://github.com/sde-coursepack/commandline/blob/main/src/main/java/edu/virginia/cs/commandline/HelloWho.java)

If you compile this program into a class file, you can run it as follows:

`java edu.virginia.cs.commandline.HelloWho Will`

And the program will output:

`Hello, Will`

<img alt="alt" src="{{site.baseurl}}/modules/java/images/6/helloWho.png"/>

How does this work at a code level? Well, let's look:

```java
    public static void main(String[] args) {
        String who = args[0];
        System.out.println("Hello, " + who);
    }
```

If you look at **_line 2_**, you can see that his is where I tell Java **"Take the first argument and store it in a `String` variable."**

Now, let's try:

`java edu.virginia.cs.commandline.HelloWho Will McBurney`

If I run this, I still get:

`Hello, Will`

That's because the String "McBurney" is stored in `args[1]`, due to the space between it and "Will". Note that, as
general practice, **We can ignore extra and unnecessary arguments**.

### Arguments with spaces

If I want to include the space, then I can run the program with the arguments:

`java edu.virginia.cs.commandline.HelloWho "Will McBurney"`

If I do this, I'll get the desired result:

`Hello, Will McBurney`

<img alt="alt" src="{{site.baseurl}}/modules/java/images/6/helloWho2.png"/>

This is because the quotations marks in the argument tell Java "this is all one argument."

### Missing Arguments
If we run the program without arguments with the command below:

`java edu.virginia.cs.commandline.HelloWho`

The program will crash with an `ArrayOutOfBoundException`:

<img alt="alt" src="{{site.baseurl}}/modules/java/images/6/helloWhoError.png"/>

There's really nothing we can do about this error: If our program needs an argument, the best we can do is inform
the user. That said, we could improve the error message clarity. To fix this, I changed the main method:

```java 
    public static void main(String[] args) {
        try {
            String who = args[0];
            System.out.println("Hello, " + who);
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Error: This program requires a command line argument for a name. format:\n" +
            "\t  java edu.virginia.cs.commandline.HelloWho name");
        }
    }
```

This will result in the following error message if arguments are missing:

```
Error: This program requires a command line argument for a name. format:
    java edu.virginia.cs.commandline.HelloWho name
```

This better error message is more useful to the user. The stacktrace we had earlier implies that the programmer
has a bug in their code, and that is how the user will interpret it very often. This error message makes it clear
that the error was actually a *user error*, and the user can correct their mistake.

## Numeric Arguments
Example: [HelloNTimes.java](https://github.com/sde-coursepack/commandline/blob/main/src/main/java/edu/virginia/cs/commandline/HelloNTimes.java)

The HelloNTimes program takes in a number via command line and prints "Hello" that many times. Example:

`java java edu.virginia.cs.commandline.HelloNTimes 5`

Will result in the console printing:

```
Hello!
Hello!
Hello!
Hello!
Hello!
```

One limitation that we have to contend with is that the arguments array is an array of `String` objects. What if we want
a number? Well, in that case, it's as simple as using `Integer.parseInt`

```java
    public static void main(String[] args) {
        try {
            int count = Integer.parseInt(args[0]);
            printHelloNTimes(count);
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Error: Must include a number argument. Example:\n"+
                    "\tjava edu.virginia.cs.commandline.HelloNTimes 5");
        } catch (NumberFormatException e) {
            System.out.println("Error: First argument was not an integer. Example:\n"+
                    "\tjava edu.virginia.cs.commandline.HelloNTimes 5");
        }
    }
```

Focusing on the error messages, you now see we have two different errors to account for. We still have to consider the
`ArrayIndexOutOfBoundsException` if no argument is given at all. However, another problem is that `Integer.parseInt` can
throw a `NumberFormatException` if the value of the String cannot be converted to an integer. For example, if we called:

`java edu.virginia.cs.commandline.HelloNTimes Steve`

OR

`java edu.virginia.cs.commandline.HelloNTimes 3.6`

then Integer.parseInt would throw a `NumberFormatException`. Once again, we want to give the user a *useful* error
message. A useful error message is a *precise* error message that tells the user **what they did wrong** and **how to 
fix it**.

**Another consideration** could be to consider how to handle the user entering a negative number. In my program, I just
ignore it, because my `printHelloNTimes` function will end up producing the same output as if the user entered `0`.

However, if I wanted to prevent a user entering negative numbers, I would want to make sure to add a meaningful
error message once again.

## Multiple Arguments
Example: [MyGCD.java](https://github.com/sde-coursepack/commandline/blob/main/src/main/java/edu/virginia/cs/commandline/MyGCD.java)

I don't include robust error checking in these last two examples. I just want to briefly show a trivial example of multiple
command line arguments.

`java edu.virginia.cs.commandline.MyGCD 144 81`

The program then assigns 144 to the variable `a` and 81 to the variable `b`.

```java
    public static void main(String[] args) {
        int a = Integer.parseInt(args[0]);
        int b = Integer.parseInt(args[1]);
        int gcd = gcd(a,b);
        String toPrint = String.format("GCD(%d, %d) = %d", a, b, gcd);
        System.out.println(toPrint);
    }
```

Thus, the run above simply prints:

`GCD(144, 81) = 9`

## Optional Arguments
Example: [IsLeapYear](https://github.com/sde-coursepack/commandline/blob/main/src/main/java/edu/virginia/cs/commandline/IsLeapYear.java)

In the isLeapYear example, I wanted to allow the user to ask if a year was a Leap Year by either the Gregorian
calendar or the Julian Calendar. If you are unfamiliar, with the difference, years divisible by 100 like 1900 and 2100 are *not* Leap years in the
Gregorian calendar, while both *are* in the Julian calendar (years divisible by 400 like 2000 and 2400 are leap years in both).

Because we use the Gregorian calendar today, I'm going to assume the user is asking about whether or not the
Gregorian calendar has a leap year in the given year. However, if the user wants, they can add `-j` or `--julian`
to the arguments to specify they want to use the Julian calendar.

As such, this program allows the user to run the program as:

`java edu.virginia.cs.commandline.isLeapYear 1900`

Which will say 1900 *is not* a leap year, while:

`java edu.virginia.cs.commandline.IsLeapYear 1900 -j`

or

`java edu.virginia.cs.commandline.IsLeapYear 1900 -julian`

Will both say that 1900 *is* a leap year. 

However, because this argument is optional, I do not want to *assume* that it is at a particular position.

This is how this is handled at the code level:

```java
    public static void main(String[] args){
        List<String> argList= Arrays.asList(args);
        boolean isJulian=checkForJulianFlag(argList);
        ...
    }

    private static boolean checkForJulianFlag(List<String> argList) {
        return argList.contains("-j") || argList.contains("-julian");
    }
```

If you are unfamiliar with `Arrays.asList`, it is a function in the `java.util.Arrays` library that produces
a List representation of an Array. It should be noted, however, that this list is **immutable**, meaning you
can't use functions like `add` and `remove` on it. However, the benefit is that it lets me use the list
method `contains` so I don't have to write my own for-loop: I can simply leverage the built-in Java List
interface.

As a consequence, you'll note that I do not assume that `-j` or `-julian` must be at index 1. So, as a result:

`java edu.virginia.cs.commandline.IsLeapYear 1900 here are some unused arguments -j`

Would result in the program saying 1900 *is* a leap year, since the `-j` being *anywhere* in there is enough for me
to assume the user wants me to check against the Julian calendar.

### Flag Argument Conventions

While you won't be expected to handle or enforce this, when we have optional arguments, we generally have two forms:

* **Short Form** such as `-j`
* **Long Form** such as `--julian`

As a general convention, the short form has one hyphen, and the long form has two hyphens.

### Optional Parameter Arguments

Note that you can also have option arguments with parameters. For example, let's say instead of have `-j`, I have `-c`
which lets the user specify any one of several calendars (including the Julian), but I still default to Gregorian because
it is the most widely used calendar in the world today. At that point, I may have a command lined like:

`java edu.virginia.cs.commandline.IsLeapYear 1900 -c Julian`

By convention, the `-c` in this context would be used to mean "set the calendar option to the next argument", which
in this case is Julian. We would then implement that logic within our code. In this way, you can think of `-c Julian`
or `--calendar Julian` as a pair of arguments that are related.

## Tips for handling command line arguments

These are some general tips for building programs that use command line arguments.

### Useful error messages

As we showed in the first two code examples we looked at (HelloWho and HelloNTimes), we want to generate meaningful
error messages that tell the user when they enter the command line arguments incorrectly. If a user sees a stacktrace without
a clear error message explaining their error,
they will think that the programmer
has a bug in their code, rather than the correct interpretation that it was the **user** who made an error.
A clear error message makes it clear
that the error was actually a *user error*.


### Identify required and optional arguments

When building your program, required arguments should have very specific positions. For example, in IsLeapYear, the
year (required int argument) is first (index 0). The optional arguments do not have required positions, and shouldn't.
After all, the very fact that the argument is optional means it isn't always there.

Required arguments should generally come first in order.

### ```Arrays.asList(args)``` is your friend

Working with raw arrays is annoying to work with. Lists, on the other hand, are not annoying, and much easier to work with. 

```java
    List<String> argsList = Arrays.toList(args);
```

Now you can use List functions like ```contains```, ```indexOf```, and ```get```, rather than having to write your own
logic that does this same thing.

### Fail fast

As much as possible, check all command line arguments before you enter the rest of the program. For example, if
the user enters a file name, ensure that file exists before you execute any time-consuming setup tasks for the program.
If you expect an argument to be an integer, and it isn't, fail before you start doing database connections, etc.

Note this may not always be possible. For example, if a program were to read a formatted file, and the user specifies
a valid filename in the command line arguments, we won't know about the bad format until we start reading the file. But
whenever we can, we want the program to fail quickly.


### Complicated Arguments

If you have a list of large, complicated Arguments, you should create a separate class to handle parsing the Arguments.
We will talk about an example of this when we get to Design.