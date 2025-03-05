---
Title: Code Smells
---

# Code Smells

A code smell, a term coined by [Kent Beck](https://en.wikipedia.org/wiki/Kent_Beck), refers to problems within the design and structure of the underlying code in software. However, code smells are explicitly not bugs or defects. Rather, they are problems with the **internal** quality of the software; that is, problems with maintainability, analyzability, complexity, testability, etc.

In this module, we will look as some common examples of "smelly code," and look at solutions.

---

* TOC
{:toc}

---


## Long methods

Methods that are overly long and complicated are difficult to read and understand, difficult to use, and most importantly difficult to maintain. 

Functions, as a tool, are good at doing one thing and one thing only. If our functions are overly long, especially if they feature multiple loops and conditionals, most likely the function is doing several things. This makes the function harder to understand. If the function is harder to understand, it's harder to test, re-use, modify, remove and replace, etc. 

Consider the following example, which takes in a file of point coordinates, like:

```text
1.0, 3.5
2.7, 1.2
0.8, -3.2
5.0, 4.2
```

...and calculates the total distance between these points, printing the total distance with 3 decimal places.

```java
public class DistanceCalculator {
    public static void main(String[] args) throws IOException {
        String filename = args[0];

        FileReader fileReader = new FileReader(filename);
        BufferedReader bufferedReader = new BufferedReader(fileReader);

        List<Point> pointList = new ArrayList<>();
        for(String line = bufferedReader.readLine(); line != null; line = bufferedReader.readLine()) {
            String[] splitLine = line.split(",");
            double x = Double.parseDouble(splitLine[0].strip());
            double y = Double.parseDouble(splitLine[1].strip());
            Point point = new Point(x, y);
            pointList.add(point);
        }

        double totalDistance = 0.0;
        for (int index = 0; index < pointList.size() - 1; index++) {
            Point first = pointList.get(index);
            Point second = pointList.get(index + 1);
            double differenceX = Math.abs(first.getX() - second.getX());
            double differenceY = Math.abs(second.getX() - second.getY());
            double distance = Math.sqrt(differenceX * differenceX + differenceY * differenceY);
            totalDistance += distance;
        }

        System.out.printf("Total distance = %.3f", totalDistance);
    }
}

class Point {
    private final double x, y;

    Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }
}
```

Notice how large that `main` method is. I want to break this up into several methods. After breaking up main into several separate methods, I get the following:

```java
public class DistanceCalculator {
    public static void main(String[] args) throws IOException {
        String filename = args[0];

        BufferedReader bufferedReader = getBufferedReader(filename);
        List<Point> pointList = getPointList(bufferedReader);
        double totalDistance = getTotalDistance(pointList);
        System.out.printf("Total distance = %.3f", totalDistance);
    }

    public static BufferedReader getBufferedReader(String filename) throws FileNotFoundException {
        FileReader fileReader = new FileReader(filename);
        return new BufferedReader(fileReader);
    }

    public static List<Point> getPointList(BufferedReader reader) throws IOException {
        List<Point> pointList = new ArrayList<>();
        for (String line = reader.readLine(); line != null; line = reader.readLine()) {
            pointList.add(getPointFromLine(line));
        }
        return pointList;
    }

    public static Point getPointFromLine(String line) {
        String[] splitLine = line.split(",");
        double x = Double.parseDouble(splitLine[0].strip());
        double y = Double.parseDouble(splitLine[1].strip());
        return new Point(x, y);
    }

    public static double getTotalDistance(List<Point> pointList) {
        double totalDistance = 0.0;
        for (int index = 0; index < pointList.size() - 1; index++) {
            Point first = pointList.get(index);
            Point second = pointList.get(index + 1);
            totalDistance += first.distanceTo(second);
        }
        return totalDistance;
    }
}

class Point {
    private final double x, y;

    Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }
    public double distanceTo(Point point) {
        double differenceX = Math.abs(this.getX() - point.getX());
        double differenceY = Math.abs(this.getX() - point.getY());
        return Math.sqrt(differenceX * differenceX + differenceY * differenceY);
    }
}
```

Notice now that each method is only a few lines. Also, it becomes much easier to design a specific test-case for each method. In general, if I ask a question, like:

1) What are we using to read the file?  
2) How do we read the file into a list of points?  
3) How are we calculating the distance between two points?

The answer to those questions are in a specific function whose name and purpose are clear, and each function does only one thing.

We will examine this case, as well as another case, in the [Extract Method](https://sde-coursepack.github.io/modules/refactoring/Extract-Method/) module.

## Large classes

One of the most common mistakes I see in newer programmers is putting their entire program inside of a single Main.java class. This is a completely understandable mistake:

1. Newer programmers may not be as comfortable knowing when to create multiple classes
2. Newer programmers may not be comfortable with Object-Oriented programming

However, a programmer looking at a class can only keep *so many* ideas inside their head at one time. A class can therefore become hard to understand if:

* The class has too many fields that do not have much to do with each other
* The class has too many methods that do wildly different things
* The class is thousands of lines of code

There is no *ideal* class length. Some classes, like "Plain Old Java Objects" (POJOs), will inherently be short: just some fields, a constructor, and some getters and setters. An example of a POJO may be:

```java
public class Student implements Comparable<Student> {
    private final int idNumber;
    private String firstName, lastName;
    private String email;

    public static final String EMAIL_DOMAIN = "virginia.edu";

    public Student(int idNumber, String firstName, String lastName, String email) {
        if (firstName == null || lastName == null) {
            throw new IllegalArgumentException("Students names cannot be null when instantiated");
        }
        this.idNumber = idNumber;
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
    }

    public int getIdNumber() {
        return idNumber;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "example.Student{" +
                "idNumber=" + idNumber +
                ", firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                ", email='" + email + '\'' +
                '}';
    }

    @Override
    public int compareTo(Student other) {
        //sort students in ascending order by id number
        return idNumber - other.idNumber;
    }
}
```

However, many of our classes will be more complicated. For example, imagine we had a class called Registrar that handled:

* the list of all students
* the list of all sections of all courses
* all course registration, including enforcing prerequisites
* all rosters for all classes
* all professors for all classes
* all grades in all classes
* all graduation requirements
* tracking student graduation progress
* tracking faculty teaching load

All of these things in one single class can make the class difficult to understand. Instead, we want to decompose all of these different needs into several smaller classes. Each class individually exposes its behavior through a smaller interface that is easier to understand.

## Long Parameter List

To reference __Clean Code__ once again:

> "The ideal number of arguments for a function is zero (niladic). Next comes one (monadic), followed closely by two (dyadic). Three arguments (triadic) should be avoided where possible. More than three (polyadic) requires very special justification - and they shouldn't be used anyway."
>
> __Clean Code__ Chapter 3, Page 40 - Bob Martin

Let's go back to our example in the [Test-Driven-Development](https://sde-coursepack.github.io/modules/testing/Test-Driven-Development/) unit.

In this case, we wrote the function like:

```java
public class StudentFinancialRecord {
    private Student student;
    private double overdue;
    private boolean isExempt;
    
    public StudentFinancialRecord(Student student) {
        this.student = student;
        this.overdue = 0;
        this.isExempt = false;
    }

    public double calculateBill() {
        int courseCount = student.getCourseList().size();
        return 0.0; //TODO: Stub
    }
    
    public void setOverdue(double overdue) {
        this.overdue = overdue;
    }
    
    public void setExempt(boolean isExempt) {
        this.isExempt = isExempt;
    }
}
```

Note that I shortened the class above to remove any unnecessary code. 
This is an example of a **monadic** (1 argument function). However, during lecture, I used a slightly different version

```java
public class StudentFinancialRecord {
    public double calculateBill(int courseCount, double overdue, boolean exempt) {
        return 0.0; //TODO: Stub
    }
}
```

This is an example of a **triadic** (3 argument function).

Which is better? Well, from a standpoint of make the code the easiest to understand, let's consider how these are use:

To call `calculateBill` on the **monadic** version might look something like:

```java
    StudentFinancialRecord record = new StudentFinancialRecord(myStudent);
    record.setOverdue(2000);
    record.setExempt(true);
    var balanceDue = record.calculateBill();
```

To call `calculateBill` on the **triadic** version might look something like:
```java
    StudentFinancialRecord record = new StudentFinancialRecord(myStudent.getID());
    var balanceDue = record.calculateBill(myStudent.getCourseList().size(), 2000, true);
```

Which is easier to understand? Generally, the first example is. This is because we can think of the `StudentFinancialRecord` as an **object** (or record of information) that is tied to a `Student`, having some amount overdue and being either exempt or non-exempt from interest. These describe the **state** of the FinancialRecord.

It's also much easier to remember the order of arguments in a **monadic**. That's because there's only 1 order the arguments can go in. Without scrolling back up, can you remember the order of the arguments in the **triadic**? If you can't, then you can try to guess, but there are 6 possible ways to arrange the arguments.

Now consider if we had 4 arguments: 24 ways they could be organized!
Or 5 arguments! 120 ways!

In general, it's easier to make multiple **monadic** calls, where each method serves a clear individual purpose, then make one **triadic** call that combines all three purposes into one.

Now, modern IDEs help with this: they will show you the method signature as you are writing your method call, so you are less likely to get the order wrong. However, if your function is taking in and using so many inputs, it might be worth considering turning the function into a class like we did with `StudentFinancialRecord`. This allows us to more easily isolate the complicated function that requires so much information and test it independent of any tangentially related functionalities.

There is a **trade-off** here. As it stands, we have what is called *temporal coupling*, where you have to "set up" the `StudentFinancialRecord` object before calling "calculateBill". Just be aware of that trade-off when designing classes. As a general rule, your class should be built ready to be used. So it may make sense to move the setters to the Constructor. In fact, we could create overloaded constructors to help with this:

```java
public class StudentFinancialRecord {
    private Student student;
    private double overdue;
    private boolean isExempt;

    public StudentFinancialRecord(Student student, double overdue, boolean isExempt) {
        this.student = student;
        this.overdue = overdue;
        this.isExempt = isExempt;
    }

    public StudentFinancialRecord(Student student) {
        this(student, 0.0, false);
    }
}
```

This means the client code (the code using this class) has the choice to set up everything with the object at once *or* use the simpler constructor and the setter methods.

Of course, if the Constructor ends up holding every field, and our class necessarily has a complicated structure, it can end up with a large argument list. This is where something like a Factory or Builder class could come in handy, which we will discuss in design patterns.

## Boolean Parameters

Speaking of our previous example, let's consider the function:

```java
    public double calculateBill(List<Integer> registeredCourseNumbers, double overdue, boolean exempt) {
        return 0.0; //TODO: Stub
    }
```

I know for certain this function does more than one thing because of the boolean argument `exempt`. In fact, the specification tells me this:

> 3) Increase the value of the field overdue amount by 10% if exempt is false (this is done AFTER step 2) 
>    * if exempt is true, this penalty is waived, and overdue does not change.

This step alone means we have two different functions here:

* one for calculating the bill when the student **is** exempt from interest
* one for calculating the bill when the student **is not* exempt from interest

As such, we can separate this into two different functions:

```java
    public double calculateBillWithInterest(List<Integer> registeredCourseNumbers, double overdue) {
        return 0.0; //TODO: Stub  - exempt is false
        }

public double calculateInterestExemptBill(List<Integer> registeredCourseNumbers, double overdue) {
        return 0.0; //TODO: Stub - exempt is true
        }
```

The reason we want to remove boolean arguments is that it requires whoever is calling the function to know at least some information about how the function works. This means understanding the function requires **more knowledge**. Instead, we want our functions to be as simple to use as possible. So instead of:

```java
   public void orderHamburger(boolean hasCheese);
```

...do...

```java
   public void orderHamburger();

   public void orderCheeseburger();
```

While the latter has more functions, both functions are clearly communicating their intent and are **niladics**, and therefore cannot be called with incorrect arguments!

### Replace booleans with polymorphism

On top of removing the boolean, we may want to consider whether a student who is interest exempt should be handled by another class entirely. For example:

```java
public class StudentFinancialRecord {
    protected double overdue;

    public double calculateBill(List<Integer> registeredCourseNumbers, BillCalculator billCalculator) {
        return billCalculator.calculate(registeredCourseNumbers);
    }
}
```

...and...

```java
public interface BillCalculator {
    public double calculate(List<Integer> registeredCourseNumbers);
}

public class TaxExemptBillCalculator {
    public double calculate(List<Integer> registeredCourseNumbers) {
        // implement tax exempt approach
    }
}

public class NonExemptBillCalculator {
    public double calculate(List<Integer> registeredCourseNumbers) {
        // implement tax exempt approach
    }
}
```

Now, we calculate *at run time* the correct amount due based on whether the Student is exempt. This has the advantage of moving the logic of calculating the bill outside of the class that simply keeps a record of the information.

This is called a Strategy Pattern, and is a very popular design pattern that lets us separate data from how the data is used.

## Primitive Obsessions

Consider the following class:

```java 
public class Building {
    private String name;
    private String streetName;
    private int streetNumber;
    private int zipCode;
    private String city;
    private String state;
    private double latitude;
    private double longitude;
}
```

This class has 8 different fields! Imagine if each of these fields had getters/setters. That's 16 different methods!

But consider that five of the fields are simply related to the address of the building. Two of the fields relate to the geographic coordinates of the building. In effect, we actually have three different ideas here:

1. The idea of a building with a name, address, and geolocation
2. The address of the building
3. The geolocation of the building

This is an example of primitive obsession (for the sake of design, we typically consider the Java `String` class to be in effect a primitive, even though it is technically a class). We are modeling complex, unrelated data in one class directly with primitive datatypes. This means the `Building` class is more complicated than it needs to be.

Instead, we can combine the primitives relevant to specific behaviors together:

```java 
public class Building {
    private String name;
    private Address address;
    private Coordinate coordinates;
}

public class Address {
    private String streetName;
    private int streetNumber;
    private int zipCode;
    private String city;
    private String state;
}

private class Coordinates {
    private double latitude;
    private double longitude;
}
```

Now, we can shorten our 16 getters and setters in Building to just 6:

* `getName`/`setName`
* `getAddress`/`setAddress`
* `getCoordinate`/`setCoordinate`

Additionally, because we can think about an Address of a building as just an `Address` and the Coordinate as just a `Coordinate`, it makes our code more modular and easier to understand and describe.

## Duplicate Code

Consider the following code from a SudokuSolving program I wrote. While there are other functions called by this method, for the sake of space, I'm only highlighting this function.

```java
public class BoxLineReductionSolver implements OneStepSolver {

    @Override
    public String solveOneStep(SudokuGrid g) {
        for (RowContainer row : g.getRows()) {
            for (int missingValue : row.getMissing()) {
                List<Cell> cells = getCellsWithPossibility(row, missingValue);
                if (inSameBox(cells)) {
                    int boxNumber = cells.get(0).getBoxNumber();
                    if (removePossibilitiesExcludingRow(missingValue, row.rowNumber, g.getBoxes().get(boxNumber))) {
                        return "BoxLineReduction: " + missingValue + " in Box " + boxNumber + " must be in Row " + row.rowNumber + "; removed " +
                                missingValue + " from all other cells in the Box.";
                    }
                }
            }
        }

        for (ColumnContainer col : g.getColumns()) {
            for (int missingValue : col.getMissing()) {
                List<Cell> cells = getCellsWithPossibility(col, missingValue);
                if (inSameBox(cells)) {
                    int boxNumber = cells.get(0).getBoxNumber();
                    if (removePossibilitiesExcludingColumn(missingValue, col.colNumber, g.getBoxes().get(boxNumber))) {
                        return "BoxLineReduction: " + missingValue + " in Box " + boxNumber + " must be in Column " + col.colNumber + "; removed " +
                                missingValue + " from all other cells in the Box.";
                    }
                }
            }
        }
        return null;
    }
}
```

This function has a number of problems, and violates many rules of design and code quality that we have and will discuss. However, the biggest problem here is **duplicate code**.

Look at the two for loops that start with:

* `for (RowContainer row : g.getRows()) {`
* `for (ColumnContainer col : g.getColumns()) {`

These two for-loops are nearly completely identical! In fact, the only difference is in the `return` String at the end. This is particularly bad because `RowContainer` and `BoxContainer` both extend the same class,  `SudokuContainer`, so I've already done the design work to avoid this repetition.

The problem with this duplication is that this is a bug waiting to happen. Imagine if I need to update this function, for example, to remove this violation of *abstraction* (we will discuss this later)

`int boxNumber = cells.get(0).getBoxNumber();`

Well, because that line of code is in two different places, I have to change it *twice*. If I change it in one place, but not the other, I now have two pieces of code that in theory should do the same thing, but are implemented differently.

We always want to be **DRY** (Don't Repeat Yourself), and we never want to be **WET** (Write Everything Twice). To fix this, I can use my polymorphic structure:

```java
    public String solveOneStep(SudokuGrid g) {
		String result = checkContainer(g, g.getRows());
		if (result != null) return result;

		result = checkContainer(g, g.getColumns());
		if (result != null) return result;
		return null;
	}

    private String checkContainer(SudokuGrid g, List<? extends SudokuContainer> containers) {
        for (SudokuContainer container : containers) {
            String result = checkSudokuContainerForBoxLineReduction(g, container);
            if (result != null) {
                return result;
            }
        }
        
        return null;
    }

	private String checkSudokuContainerForBoxLineReduction(SudokuGrid g, SudokuContainer container) {
		for (int missingValue : container.getMissing()) {
			List<Cell> cells = getCellsWithPossibility(container, missingValue);
			if (inSameBox(cells)) {
				int boxNumber = cells.get(0).getBoxNumber();
				if (removePossibilitiesExcludingColumn(missingValue, container.getContainerID(), g.getBoxes().get(boxNumber))) {
					return "BoxLineReduction: " + missingValue + " in Box " + boxNumber + " must be in Column " + container.getContainerID() + "; removed " +
							missingValue + " from all other cells in the Box.";
				}
			}
		}
		return null;
	}
```

This is still not perfect (the functions are too long and complicated), but it is an improvement in that our code is now less repetitive.


## Message Chains

It's important to note that **fewer lines of code** aren't inherently better. That is, doing things to shorten the number of lines of code without actually changing the code complexity isn't beneficial.

Consider the following code:

```java
    public String getFirstValueFromCSVFileAsUppercase() {
        return new BufferedReader(new FileReader(new File("filename.csv")))).readline().strip().split(",")[0].strip().toUpperCase();
    }
```

This one line of code is completely unreadable. However, instead, consider the following:

```java
    public String getFirstValueFromCSVFileAsUppercase() {
        File myCSVFile = new File("filename.csv");
        FileReader fileReader = new FileReader(myCSVFile);
        BufferedReader reader = new BufferedReader(fileReader);
        
        String firstLine = reader.readLine();
        firstLine = firstLine.strip();
        String[] firstLineSplit = firstLine.split(",");
        String firstValue = firstLineSplit[0].strip();
        return firstValue.toUpperCase();
    }
```

These two pieces of code do the same thing, but the second is more readable and maintainable, because it's much more clear what the purpose of each step is in our code.

Additionally, we can now extract these steps into individual methods:

```java
    public String getFirstValueFromCSVFileUppercase() {
        BufferedReader reader = getBufferedReaderFromFilename("filename.csv");
        String[] firstRowArray = reader.getNextRowAsStringArray(reader);
        return getStrippedStringAsUppercase(firstRowArray[0]);
    }

    public BufferedReader getBufferedReaderFromFilename(String filename) {
        File myCSVFile = new File("filename.csv");
        FileReader fileReader = new FileReader(myCSVFile);
        return new BufferedReader(fileReader);
    }
    
    public String[] reader.getNextRow(reader) {
        String firstLine = reader.readLine();
        firstLine = firstLine.strip();
        return firstLine.split(",");
    }

    public String toStrippedUppercase(String string) {
        String strippedString = strip.strip();
        return strippedString.toUppercase();
    }
```

Does this take up more space? Yes. However, now the code is easier to understand, as each individual function has a single clear purpose. Additionally, each function may be more re-usable, meaning we won't have to rewrite repeated functionality later as needed. Each function is also individually testable.

## Conclusion

The above was just a sampling of Code Smells. You can find a more thorough  and larger list at [Refactoring Guru](https://refactoring.guru/refactoring/smells).
