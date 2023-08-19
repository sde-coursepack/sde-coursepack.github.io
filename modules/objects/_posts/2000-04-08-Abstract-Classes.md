---
Title: Abstract Classes
---



# Abstract Classes
{: .no_toc }

You can think of Abstract classes as something of a half-way point between parent classes like `Clock` in the last module and an `interface`. An abstract class *is* a parent-class, and like a class it can have real data, method implementations, constructors, etc. However, it also allows us to define method abstractions like an `interface`; that is, we can define method signatures without implementing them like we do in a Java `interface`. In this module, we will explore creating and extending an `abstract` class.

---

## Contents
{: .no_toc }

* TOC
{:toc}

---

## Example problem

Let's consider a situation where we want to create a class that meets the following need:

"I need a class that allows me to read in a data source and produce a List of Students."

Some questions should immediately come to mind, though, like:
- What is the data source we're reading?
- What is the format of the data?
- Will this data source or format change in the future?

When thinking about software change, a common source of change is where data comes from and how it is formatted. If you were developing an application in the 90s, you might use something like .csv or Excel to move data. By the mid 2000s, XML became the most common file format used for sharing data over the web. As of this writing (2020s), JSON has become the most common, though there is also growth in a format called YAML.

As such, we will want to separate the interface of reading data from any particular implementation, as we may end up needing to change that over time.

Below is an example of an abstract class that defines the interface and structure of reading in data into a `List` of `Student` objects. Here, we are using something called **memoization** (this is not a typo of *memorization*, though the idea is surprisingly similar). 

```java
public abstract class StudentReader {
    private List<Student> studentList;
    
    public StudentReader() {
        studentList = null;
    }
    
    public List<Student> getStudentList() {
        if (studentList == null) {
            studentList = readStudentsIntoList();
        } else {
            return studentList;
        }
    }
    
    protected abstract List<Student> readStudentsIntoList();
    
    public void invalidate() {
        studentList = null;
    }
}
```

### Memoization

In this code, I implemented an optimization technique called memoization.

Here, the idea is that `studentList` is a **memo**, or a cached (stored in memory) result of the method `getStudentList` in our program. The first time we call `getStudentList`, it will read from the data source to create the list. This means if we need to access the list of Students multiple times, we only actually read the data source the first time. In short, we're trading memory for processing time later. The function `invalidate` exists to allow us to delete the memo, which could be useful if we know our data source has been updated, and our existing student list is out of date.

### `abstract` methods

You'll notice that the method `protected abstract List<Student> readStudentsIntoList()` is unimplemented, and specifically listed as `abstract`. The purpose of this method is to actually handle *reading* the data source and producing the List of students which will be saved into `studentList`. However, in this abstract class, we do not implement this method.

The reason is that we do not want this class to have a dependency to any particular data format or data source, since that could change. Instead, we might have a class like below, which would read in Student data from a CSV file:

```java
public class CSVStudentReader extends StudentReader {
    private String filename;
    
    public CSVStudentReader(String filename) {
        if (!filename.toLowerCase().endsWith("csv")) {
            throw new RuntimeException("Illegal filename " + filename + " - filename must end with csv");
        }
        this.filename = filename;
    }
    
    @Override
    protected List<Student> readStudentsIntoList() {
        try {
            InputStream is = new FileInputStream(filename);
            InputStreamReader isr = new InputStreamReader(is);
            BufferedReader br = new BufferedReader(isr);
            ...// read file and return List of students
        } catch (FileNotFoundException e) {
            ...// handle file not found errors
        } catch (IOException e) {
            ...// handle other io errors
        }
    }
}
```

You can think of the `abstract` method in the parent class as the same as a method in an `interface`. That is, it is a requirement that an implementing class (or, a class that `extends` the parent) is syntactically required to implement that method. That is, if we delete or in any way change the signature of the method `readStudentsIntoList` in `CSVStudentReader`, the code will no longer compile! This is a good thing, as it lets us enforce design decisions using Java's syntax checker.

In this way, let's say our data goes from CSV to JSON via a web-service. Well, now, instead of changing any existing classes, we instead can create `JSONStudentReader`:

```java
public class JSONStudentReader extends StudentReader {
    private String webServiceURL;

    public CSVStudentReader(String webServiceURL) {
        this.webServiceURL = webServiceURL;
    }

    @Override
    protected List<Student> readStudentsIntoList() {
        try {
            URL webService = new URL(webServiceURL);
            ...
        } catch (MalformedURLException e) {
            ...
        } catch (IOException e) {
            ...
        }
    }
}
```

Notice that this lets us change the data source and/or format at any time simply but creating a new class that extends `StudentReader`, without actually changing the `StudentReader`, `CSVStudentReader`, or any other extending class! Additionally, each child-class of `StudentReader` will automatically support the memoization in the parent class without having to implement it themselves, and without copying/pasting code!

## Abstract class vs Interfaces

It may seem like abstract classes and interfaces serve similar roles, and indeed they can. However, specific differences are as follows:

1) A class can only extend one other class, including abstract classes. However, a class can implement any number of interfaces.
2) An abstract class can have fields, constructors, and working code. Interfaces can have none of these
3) An abstract class can have `private` and `protected` methods, and interface can only have public methods

In general, we use interfaces to say "this class supports this feature", whereas when we extend a class, we are describing a clear type hierarchy. That is, the child class is a "type of" the parent class.