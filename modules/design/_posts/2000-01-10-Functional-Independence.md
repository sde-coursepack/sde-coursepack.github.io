---
Title: Functional Independence
---

# Functional Independence

When we are decomposing our system into modules, we need to consider how our modules are integrated with each other as we bring all the modules back together. We want the interactions between modules to be as simple as possible. This is true both in functional decomposition and class decompositions.

* TOC
{:toc}



## Coupling

Coupling is the degree to which two modules interact with one another. This is different from cohesion in a key way:

**cohesion:** the extent to which elements **in the same module** are dependent on one another (high cohesion is good), or **intra**dependency.

**coupling:** the extent to which elements **in interacting modules** are dependent on each other (loose coupling is good), or **inter**dependency.

## Types of Coupling

We also have different levels of coupling like we did cohesion. As before, we will arrange them from worst to best.

**WORST**
- Content coupling
- Common coupling
- Control coupling
- Stamp coupling
- Data coupling  
**BEST**


### Content Coupling

Content coupling occurs when one class modifies the inner state of a class it depends on directly (as opposed to using the public interface). For example:

```java
public class Student {
    String name, email;
    int id;
    int notificationsReceived;
}

public class StudentNotifier() {
    public notifyStudent(String message, Student s) {
        Email notificationEmail = new Email(message, s.email);
        s.notificationsReceived++;
    }
}
```

Here, one class is directly responsible for reading and writing information to another class's private data. This is bad for a number of reasons:

1) Any changes to the private implementation details of Student will cause changes to propagate to any classes that use its data.
2) Any other class could theoretically be reading and writing to the same data, making interactions with Student potentially complicated and difficult to track. If you cannot easily deduce *where* mutable data is being changed, it becomes extremely difficult to locate defective code.

Instead, we may want to relocate the `notifyStudent` function:

```java
public class Student {
    private String name, email;
    private int id;
    private int notificationsReceived;

    public notify(String message) {
        Email notificationEmail = new Email(message, email);
        notificationsReceived++;
    }
}

public class StudentNotifier() {
    public notifyStudent(String message, Student s) {
        s.notify(message);
    }
}
```

The idea here is that a Student is an entity that you can *notify*.  Let the `Student` class handle that interaction.

### Common Coupling

Two items are "content coupled" if they have **read and write** access to the same global data. An easy illustration of this is showing the use of static global variables which is used as a communication source between modules. Note that for this to be common coupling, we specifically need both **read and write** access. For example, public constant values, which cannot be changed, do not violate Common Coupling.

```java
public class Main {
    private static String filename;

    public static void main(String[] args) {
        filename = args[0];
        MyFileReader myFileReader = new MyFileReader();
        myFileReader.run();
    }
    
    public static String getFilename() {
        return filename;
    }
}

public class MyFileReader {
    public void run() {
        FileReader fileReader = new FileReader(Main.getFilename());
        BufferedReader br = new BufferedReader(fileReader);
        ...
    }
}
```

The above is bad because, by giving `public` access to the `static` field filename, **any class in our entire project** could *read* or *write* to this field. As a general rule, static values should generally be limited to constants (which cannot be changed, therefore have no risk). Instead, we can replace global variables with passed arguments (such as to a constructor or method argument):

```java
public class Main {    
    public static void main(String[] args) {
        filename = args[0];
        MyFileReader mfr = new MyFileReader(filename);
        mfr.run();
    }
}

public class MyFileReader {
    private String filename;
    
    public MyFileReader(String filename) {
        this.filename = filename;
    }
    
    public void run() {
        FileReader fileReader = new FileReader(Main.filename);
        BufferedReader br = new BufferedReader(fileReader);
        ...
    }
}
```

Note that "global data" doesn't inherently mean global variables, however. For example, if two different classes are both directly interacting with a shared database, and both classes can add, remove, and alter records in the same table, this is common coupling (note that some will call this **external coupling** and distinguish it from common coupling, but the core problem is the same). This is because if there is a data error, it is not clear which module created the corruption. However, this can be resolved by creating one module that handles database interactions with other modules that need to interact with. We will discuss this idea further when discussing the Singleton Design Pattern.

### Control Coupling

Control coupling is when one module passes some kind of flag (typically a boolean) to another module which is used in control flow. This can be `boolean` variables or `enum` (enumerated types). However, it can also be any other value sent to a function strictly and transparently for the purposes of control flow navigation.

Here is an example of control coupling making two modules more tightly coupled.

```java
public class Employee {
    private String name, jobTitle, email, homeAddress;
    
    public String getInfo(String infoType) {
        infoType = infoType.toLowerCase();
        switch(infoType) {
            case "name":
                return name;
            case "email":
                return email;
            case "title":
                return jobTitle;
            case "address":
                return homeAddress;
            default:
                throw new IllegalArgumentException();
        }
    }
}
```

This is very tightly coupled, because in order to use the function `getInfo` correctly, you **have** to understand that function's structure and options. This is bad because changes to the method `getInfo` can and often will necessitate changes to *how `getInfo` is called by the caller.* This is the reason Control Coupling is bad: a change to the called will likely require changes to the caller.

By contrast, simply having `getName()`, `getJobTitle()`, etc. as getting functions would be significantly better and simpler to understand. Thus, in this case, even though the input is a String, it still effectively a "flag" value that is only used by control flow.

Now, in some cases, it is not possible to completely avoid having some degree of conditional logic in handling an input. However, we can isolate that control coupling to only a single method using a Factory Method Pattern, and use polymorphism to keep the interactions with the product class abstract and not reliant on the underlying class type.

```java
public enum FileType { CSV, XML, JSON, TXT };

public class FileWriterFactory {
    public FileWriter getFileWriter(String filename) {
        String fileExtension = getFileExtension(filename);
        return switch(fileExtension) {
            case ".csv" -> new CSVFileWriter(filename);
            case ".xml" -> new XMLFileWriter(filename);
            case ".json" -> new JSONFileWriter(filename);
            case ".txt" -> new TXTFileWriter(filename);
            case default -> throw new IllegalArgumentException();
        };
    }

    /**
     * Takes in a file name and returns the extension. Example:
     * "myFile.txt" -> ".txt"
     */
    private String getFileExtension(String filename) {
        
    }
}
```

In the above example, assume that all the file writers specified are concrete implementations of some abstract `FileWriter` interface that has a `writeData(Data data)` method. This is better because the caller doesn't actually have to know which filename connects to which `FileWriter`:

```java
public class Example {
    public static void main(String[] args) {
        String outputFilename = args[0];
        Data data = getData();
        FileWriterFactory factory = new FileWriterFactory();
        FileWriter fileWriter = factory.getFileWriter(outputFilename);
        fileWriter.writeData(data);
    }
    
    private Data getData() {
        //gets data from somwhere
    }
}
```

Now, we have **encapsulated** the logic of "which output gets selected" *entirely* inside `FileWriterFactory`. This means changes, such as new output formats, could be localized into the `getFileExtension` function in that class. The caller (`main`) would be completely unaffected if we added new FileWriter implementations that only required a filename to initialize.


### Why Boolean arguments are bad

Boolean arguments are often a source of control-coupling. Another reason boolean arguments are bad is that usage of functions with boolean arguments often lacks understandability.

```java
    public void makeBurger(boolean hasCheese) {
        if (hasCheese) {
            cookBurger();
            addCheese();
            assembleSandwich();
        } else {
            cookBurger();
            assembleSandwich();
        }
    }   
```

A reason that this is a bad function is that any function using it must know the **intent** of the boolean variable used. In a vaccuum, a call to this function may look like `makeBurger(true)`. It's incredibly unclear what the purpose of `true` is, here. 

One option that could be better is using an enum:

```java
    public enum BurgerType = {CHEESEBURGER, HAMBURGER};
    public void makeBurger(BurgerType burgerType) {
        if (burgerType == CHEESEBURGER) {
            cookBurger();
            addCheese();
            assembleSandwich();
        } else {
            cookBurger();
            assembleSandwich();
        }
    }  
```

At least now, to call this function to make a cheeseburger, we call `makeBurger(CHEESEBURGER)` which is better. However, we can remove the control coupling entirely by simply making two functions.

```java
    public void makeCheeseBurger() {
        cookBurger();
        addCheese();
        assembleSandwich();
    }

    public void makeHamBurger() {
        cookBurger();
        assembleSandwich();
    } 
```

### Stamp Coupling

Stamp Coupling exists when a module passes a data structure to another module, when the entire data structure is not needed.

```java
public class Student {
    private String name, email;
    private StudentRecord record;
    
    public StudentRecord getRecord() { return record; }
}

public class StudentRecord {
    private int credits;
    private List<Grade> gradeList;
    
    public List<Grade> getGradeList() { return gradeList; }
}

public class GPACalculator() {
    public double calculateGPA(Student s) {
        StudentRecord record = s.getRecord();
        List<Grade> gradeList = record.getGradeList();
        ...
        
        return gpa;
    }
}

```

In the above case, any `Client` class that calls calculateGPA is passing unnecessary information, like a students name, email, and credits. This is problematic because this means that the `GPACalculator` class is now coupled to and dependent on the interfaces of both `Student` and `StudentRecord`, in addition to its necessary Dependency on the class `Grade`. This means now if any of the three classes change, this change will propagate to GPACalculator. On the other hande, if instead, `calculateGPA` only accepted a `List<Grade>`, now it is a much more stable interface, as only changes to the interface of `Grade` and `List` could affect it.

In short, if you find your function is throwing away a significant part of the input data, consider if it might be more tightly defined.

### Data Coupling

The gold standard: all communication between modules is done via passing the minimum amount of data as arguments and returning exactly the data needed. A good example of this is `Math.sqrt(double number)`, which takes in a number, and returns its square root. The modules otherwise do not share any data. Additionally, data passed is not mutated by the call. 

This insures that any action taken by the "called" method, with the exception the data it returns, will not affect the execution of the calling method. Additionally, the calling method only needs to understand the intent of the parameters at a functional interface level.

This allows us to develop both modules independently, without worrying about behavior of one module complicating the behavior of another.


## Mutability and Coupling

One issue of coupling relates to mutability, that is objects where the State can change. For example:

```java
    List<Integer> myList = new ArrayList<>(List.of(8, 6, 7, 5, 3, 0, 9));
    Collections.sort(myList);
    System.out.println(myList); // prints 0, 3, 5, 6, 7, 8, 9
```

Here, we are sorting the variable `myList`. The module `Collections` has a sorting method. However, this sorting is done **in-place**. That means that we are actually modifying the value of `myList` *in another module*.

This is worse than *data coupling*, because the relationship is more complicated. Rather than handling side effects locally, we are relying on another module to handle our side effects. While this is a *built-in* Java function, it would be easier to understand and use this method correctly if this method returned a *new* List, rather than modifying our existing one. Of course, this creates a trade-off: if we make the usage of `sort` create a new list, it now uses twice as much memory for our list. This reveals something worth noting: there is often a trade-off between easily understandable/maintainable code and optimization. At times, we will sacrifice simplicity for performance.


### Temporal Coupling

Here is one example of temporal coupling, which can emerge when dealing with modules that are Procedurally cohesive.

```java
public class GuessResult {
    private String guess;
    private String answer;
    
    public String getGuess() {
        return guess;
    }

    public void setGuess(String guess) {
        this.guess = guess.toUpperCase();
    }

    public String getAnswer() {
        return answer;
    }

    public void setAnswer(String answer) {
        this.answer = answer.toUpperCase();
    }

    public LetterResult[] getGuessResult() { ... }
}
```

In order to use the above code, you would have to do something like:

```java
    GuessResult gr = new GuessResult();
    gr.setGuess("BOXER");
    gr.setAnswer("MATCH");
    LetterResult[] result = gr.getGuessResult();
```

That is, you specifically have to call `setAnswer` and `setGuess` **before** you can call `getGuessResult`, otherwise you get an exception. This means that the interface of this module is **more complicated than it needs to be**. Consider the following change:

```java
public class GuessResult {
    private String guess;
    private String answer;

    public GuessResult(String guess, String answer) {
        this.guess = guess.toUpperCase();
        this.answer = answer.toUpperCase();
    }

    public LetterResult[] getGuessResult() { ... }
}
```

Now what does the usage look like?

```java
    GuessResult gr = new GuessResult("BOXER", "MATCH");
    LetterResult[] result = gr.getGuessResult();
```

In this case, we have turned **temporal coupling** (with *procedural cohesion*) into **data coupling** (with *sequential cohesion*), as it is no longer possible to get the function calls out of order. You have to call the constructor first anyways to instantiate the object. But after that, the object cannot be in an incorrect state. If you want to call this function again, simply create a new `GuessResult` object. A note that there is a trade-off here, as instantiating a new object, as opposed to reusing an older one, comes with a memory and time cost. However, from a maintainability perspective, the second approach is more maintainable.

**How does code like the first example happen?** It happens due to an over-adherence to simple-sounding principles. This particular example emerged because of trying to follow principles in __Clean Code__. Specifically, from Chapter 3 where Bob Martin argues that Monadic functions (functions with one argument) are generally better than Dyadic functions (functions with two arguments). If you take that idea out of context and try to force one-argument functions, that's how you end up with setups like:

```java
    GuessResult gr = new GuessResult();
    gr.setGuess("BOXER");
    gr.setAnswer("MATCH");
    LetterResult[] result = gr.getGuessResult();
```

## Conclusion

In the last two units, we talked about *cohesion* and *coupling*. We want our classes to be *highly cohesive* (that is, parts of the modules highly **intra**dependent to describe one behavior) and *loosely coupled* (that is, relationships to other modules are as simple as possible, ideally just simple functional calls).