---
Title: OO Design Principles
---

* TOC
{:toc}

# Design Principles

In this module, we will focus on some additional design principles. First, we look at some general design principles, and then second we focus on Object-Oriented Design Principles. These principles are designed to help us achieve modular, functionally independent code that adheres to proper use of abstraction and information hiding.

## KISS Principle

__Keep it simple, stupid!__

A simpler design is better than a complicated one. The best design is the simplest design that meets the need. Whenever possible, we should avoid adding un-need complexity. To that end, having several well-named single purpose modules is easier to understand than having one big multi-purpose module.

## DRY Principle

__Don't Repeat Yourself__

In general, don't copy-and-paste code. If you are copying several lines of code, chances are you are describing something that should be a function (or possibly a class). Additionally, if two classes are performing the same actions, it's possible that those classes should both be extending another class, or at least implementing a shared interface.

Specifically, we should use functions or polymorphism when we are sharing the *same knowledge.*

```java
public class GregorianDateValidator {
    public boolean isValidDate(int year, int month, int day) {
        return (isValidYear() && isMonthValid() && isDayValid());
    }
    
    private boolean isValidYear(int year) {
        return year != 0; //year 0 doesn't exist on Gregorian Calendar
    }
    
    private boolean isMonthValid(int month) {
        return 1 <= month && month <= 12;
    }
    
    private boolean isDayValid(int year, int month, int day) {
        return 1 <= day && day <= daysInMonth(year, month);
    }
    
    private int daysInMonth(int year, int month) {
        return switch (month) {
            case 1, 3, 5, 7, 8, 10, 12 -> 31;
            case 4, 6, 9, 11 -> 30;
            case 2 -> getDaysInFebruary(year);
            default -> throw new IllegalArgumentException("Error: invalid month" + month);
        };
    }
    
    private int getDaysInFebruary(int year) {
        if (isLeapYear(year)) {
            return 29;
        } else {
            return 28;
        }
    }
    
    private boolean isLeapYear(int year) {
        if (year % 400 == 0) {
            return true;
        } else if (year % 100 == 0) {
            return false;
        } else if (year % 4 == 0) {
            return true;
        } else {
            return false;
        }
    }
}
```

Now, imagine we also wanted to make `JulianDateValidator`. And you note that, functionally, this is doing the same task as `GregorianDateValidator`. Only,  One possibility would be something like:

```java
public abstract class DateValidator {
    public abstract boolean isValidDate(int year, int month, int day);
}
```

And then from there, we could simply change `GregorianDateValidator` to:

```java
public class GregorianDateValidator extends DateValidator {
    @Override
    public boolean isValidDate(int year, int month, int day) {
        return (isValidYear() && isMonthValid() && isDayValid());
    }
    ...
}
```

But now, if we were to write `JulianDateValidator`:

```java
public class GregorianDateValidator extends DateValidator {
    @Override
    public boolean isValidDate(int year, int month, int day) {
        return (isValidYear() && isMonthValid() && isDayValid());
    }

    private boolean isValidYear(int year) {
        return year != 0; //year 0 doesn't exist on Gregorian Calendar
    }

    private boolean isMonthValid(int month) {
        return 1 <= month && month <= 12;
    }

    private boolean isDayValid(int year, int month, int day) {
        return 1 <= day && day <= daysInMonth(year, month);
    }
    
    ... //assume we kept going
}
```

You'll note that we are duplicating a lot of code. This is to be avoided because nearly all of these functions do the same thing! In fact, the only exception is `isLeapYear(int year)`, as in the Julian calendar, the rule is more simply "every 4 years is a Leap year." So instead, a better approach might be to **raise up** the methods that share knowledge, and then the children only keep the methods which rely on different knowledge:

```java
public abstract class DateValidator {
    public boolean isValidDate(int year, int month, int day) {
        return (isValidYear() && isMonthValid() && isDayValid());
    }
    
    private boolean isValidYear(int year) {
        return year != 0; //year 0 doesn't exist on Gregorian Calendar
    }
    
    private boolean isMonthValid(int month) {
        return 1 <= month && month <= 12;
    }
    
    private boolean isDayValid(int year, int month, int day) {
        return 1 <= day && day <= daysInMonth(year, month);
    }
    
    private int daysInMonth(int year, int month) {
        return switch (month) {
            case 1, 3, 5, 7, 8, 10, 12 -> 31;
            case 4, 6, 9, 11 -> 30;
            case 2 -> getDaysInFebruary(year);
            default -> throw new IllegalArgumentException("Error: invalid month" + month);
        };
    }
    
    private int getDaysInFebruary(int year) {
        if (isLeapYear(year)) {
            return 29;
        } else {
            return 28;
        }
    }
    
    protected abstract boolean isLeapYear(int year);
}
```

You'll note here we only made `isLeapYear` protected, and all the other methods are still private. This was because I don't *want* either child class overriding the above methods. If, later on, I add new implementations which may need to override additional functions, then I can do that later. By now, the concrete child classes are:

```java
public class GregorianDateValidator extends DateValidator {
    @Override
    protected boolean isLeapYear(int year) {
        if (year % 400 == 0) {
            return true;
        } else if (year % 100 == 0) {
            return false;
        } else if (year % 4 == 0) {
            return true;
        } else {
            return false;
        }
    }
}

public class JulianDateValidator extends DateValidator {
    @Override
    protected boolean isLeapYear(int year) {
        return year % 4 == 0;
    }
}
```

Now, the shared logic is all in one place.

---

### Don't be too DRY

However, his principle shouldn't necessarily be taken as a hard-and-fast rule as it may be stated. Some new programmers interpret this as "if there are any two places in your entire project that repeat any code whatsoever, you should fix that".

Consider the following code modification of the above. I'll ask you: is this code better?

```java

public abstract class DateValidator {
    private static int SMALLEST_VALID_DAY_AND_MONTH = 1;
    private static int MOST_COMMON_MONTH_DAYS = 31;
    private static int USUAL_DAYS_IN_FEBRUARY = 28;
    private static int LARGEST_VALID_MONTH = 12;
    
    private static int USUAL_LEAP_YEAR_INTERVAL = 4;
    
    public boolean isValidDate(int year, int month, int day) {
        return andGate(
                    andGate(isYearValid(day),isMonthValid(month)),
                    isDayValid());
    }

    private boolean isYearValid(int year) {
        return !isZero(year); //year 0 doesn't exist on Gregorian Calendar
    }
    
    private isZero(int number) {
        return number == 0;
    }
    
    private boolean isMonthValid(int month) {
        return firstNumberBetween(month, SMALLEST_VALID_DAY_AND_MONTH, LARGEST_VALID_MONTH);
    }
    
    private boolean isDayValid(int year, int month, int day) {
        return firstNumberBetween(day, SMALLEST_VALID_DAY_AND_MONTH, daysInMonth(year, month));
    }
    
    private int daysInMonth(int year, int month) {
        return switch (month) {
            case 1, 3, 5, 7, 8, 10, 12 -> MOST_COMMON_MONTH_DAYS;
            case 4, 6, 9, 11 -> MOST_COMMON_MONTH_DAYS - 1;
            case 2 -> getDaysInFebruary(year);
            default -> throw new IllegalArgumentException("Error: invalid month" + month);
        };
    }
    
    private int getDaysInFebruary(int year) {
        if (isLeapYear(year)) {
            return USUAL_DAYS_IN_FEBRUARY + 1;
        } else {
            return USUAL_DAYS_IN_FEBRUARY;
        }
    }
    
    private boolean firstNumberBetween(int month, int low, int high) {
        return andGate(low <= month, month <= high);
    }
    
    private boolean andGate(boolean a, boolean b) {
        return a && b;
    }
    
    private boolean isDivisibleBy(int number, int divisor) {
        return isZero(number % divisor);
    }
    
    protected abstract boolean isLeapYear(int year);
}

public class GregorianDateValidator extends DateValidator {
    @Override
    protected boolean isLeapYear(int year) {
        if (isDivisibleBy(year, 400)) {
            return true;
        } else if (isDivisibleBy(year, 100)) {
            return false;
        } else {
            JulianDateValidator temp = new JulianDateValidator();
            return temp.isLeapYear();
        }
    }
}

public class JulianDateValidator extends DateValidator {
    @Override
    protected boolean isLeapYear(int year) {
        return isDivisibleBy(year, USUAL_LEAP_YEAR_INTERVAL);
    }
}
```

The code above really isn't better. And the reason is that despite being more DRY, it is **less readable and understandable**. Probably the worst example of this is:

```java
public class GregorianDateValidator extends DateValidator {
    @Override
    protected boolean isLeapYear(int year) {
        if (isDivisibleBy(year, 400)) {
            return true;
        } else if (isDivisibleBy(year, 100)) {
            return false;
        } else {
            JulianDateValidator temp = new JulianDateValidator();
            return temp.isLeapYear();
        }
    }
}
```

Here, we are creating an instance of an otherwise un-used module and using it's **implementation** of `isLeapYear`. This means that `GregorianDateValidator` is now **dependent on** knowledge in `JulianDateValidator`, when adding this dependency wasn't necessary. The previous method:

```java
public class GregorianDateValidator extends DateValidator {
    @Override
    protected boolean isLeapYear(int year) {
        if (year % 400 == 0) {
            return true;
        } else if (year % 100 == 0) {
            return false;
        } else if (year % 4 == 0) {
            return true;
        } else {
            return false;
        }
    }
}
```

...was completely understandable.

In short, never unnecessarily sacrifice **understandability** and **functional independence** for DRY-ness. Certainly, if there are functions that are more complicated, it makes sense to encapsulate that as a class or method and re-use it. But you shouldn't overdo it.

---

## YAGNI Principle

__"You ain't gonna need it"__

The YAGNI Principle states that you should only add features to your code when required. The short version of this principle is "don't try to future-proof your code!" 

This was specifically a common problem in **plan-driven** software development (as opposed to *agile*). Because most major design decisions had to be made before implementation in a *plan-driven* approach. It was necessary to try to anticipate changes **and** develop the infrastructure to support those changes.

Because agile promotes add features iteratively, with an emphasis on refactoring often, it's often better to wait until you *know* what new features you need before you start designing and implementing. A reason for this is that if we anticipate a feature will be needed in the future, we could be wrong! Now any design changes we have made, code and tests we've written, etc. are useless. The time spent on those features was wasted.

Additionally, over design of our software system can make it **harder** to understand, even if the software is theoretically easier to change. Remember, the first step to any software change is **understanding** the software. So avoiding over-design is just as important as avoiding under design!

> “Such is the vastness of his genius that he can outwit even himself.”
> Stephen Erickson, Deadhouse Gates, Book 2 of the Malazan Book of the Fallen

