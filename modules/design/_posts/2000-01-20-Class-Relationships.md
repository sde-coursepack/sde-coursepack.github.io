---
Title: Class Relationships
---

# Class Dependency

In this module, we'll look at 6 kinds of class relationships, and describe the design consequences of these relationships. These relationships are covered in, generally speaking, the order from tightest to least tight coupling.


* TOC
{:toc}


## Depends on (uses)

At that loosest level, one class depends on another class when it uses methods or functions from that class. Consider the following code:

```java
public class Course {
    Professor professor;
    List<Student> students;
    
    ...
    
    public EmailResponseCode emailAllStudents(String subject, String message) {
        EmailBuilder emailBuilder = new EmailBuilder();
        emailBuilder.setSender(professor.getEmailAddress())
                .setSubject(subject)
                .setMessage(message);
        for (var student : students) {
            var studentEmailAddress = student.getEmailAddress();
            emailBuilder.addBccRecipient(studentEmailAddress);
        }
        Email email = emailBuilder.get();
        return email.send();
    }
}
```

In the above function, `emailAllStudents`, we are *using* an instance of `EmailBuilder`, `Email` and
`EmailResponseCode` temporarily in this function. That is we are re-using code in these classes. Specifically in each class:

* `EmailBuilder`
  * constructor with zero arguments
  * `void setSender(String emailAddress)`
  * `void setSubject(String subject)`
  * `void setMessage(String message)`
  * `void addBccRecipient(String emailAddress)`
  * `Email get()`
* Email
  * `EmailResponse send()`

Note that while we return an `EmailResponse`, we never actually *do* anything with it here, we simply rely on the class existing. In this way, we can say that `Course` and `EmailResponse` are coupled, but very loosely. Short of renaming or deleting the class entirely, any change to `EmailResponseCode` won't directly necessitate a change to `emailAllStudents`.

`Course` and `Email` are *very slightly* more tightly coupled. That is, in addition to depending on the class existing, `Course` is also depending on a method called `send()` that returns an `EmailResponseCode`. If the interface of this method changes (that is the name, arguments, return type), this would necessitate changes to `Course.emailAllStudents`. However, if the *implementation* of this method changes, but the interface stays the same (the method signature and method behavior), then no changes should be needed in `Course.emailAllStudents`. So this is still very loose coupling.

`Course` is, again, slightly more coupled to `EmailBuilder`. This is because `Course` depends on the zero-argument constructor, as well as 5 other methods, existing. However, `Course` is simply calling these methods as provided by their interface. That is, `Course` is tied to the *interface*, but not the *implementation*. So again, if the interface stays the same, but the *implementation* changes, `Course` should be unaffected. However, any changes to the interface for *any* of these methods *will* necessitate changes in `Course`. Because there are *more* method interactions, this is higher coupling than exists for `Email` or `EmailResponse`

The key idea here is to think of each unique method call from one class to another as a thread connecting two boxes. The more threads there are between the boxes, the harder they are to separate, because it requires more work to cut or replace each individual thread.

However, the *thread* of interacting with a public interface should, when possible, be at low levels of coupling (ideally data coupling). This is because a well design interface can be receptive to change in implementation without requiring a change in the interface. Sometimes interface changes are unavoidable, and that becomes a more effort-heavy refactor.

## Aggregation

Looking at the same example, specifically note the fields:

```java
public class Course {
    Professor professor;
    List<Student> students;
    
    ...
}
```

Here, there are three classes being used as fields, `Professor`, `Student`, and `List` (specifically `List<Student>`). Looking specifically at `emailAllStudents`, we can see the following usages of these fields:

* `Professor`
  * `String getEmailAddress()`
* `Student`
  * `String.getEmailAddress()`
* `List<E>`
  * `Iterator<E> .iterator()` - used implicitly by the for-loop.

Here, we are using these three classes similar to how we used `EmailBuilder`, `Email`, and `EmailResponse`. However, a distinction is that these classes are *part of the fields of `Course`*. That is, `Course` doesn't just **use** these classes, `Course` **has** these classes, or **aggregates** these classes.

In practice, this almost certainly means that every, or nearly every, public method of `Course` will interact in some way with `Professor`, `Student`, and `List`. These repeated interactions across several methods means a tighter coupling between `Course` and these classes, compared to its coupling with `Email`.

This doesn't mean that this is bad. If, logically, a course aggregates a `Professor` and a `List<Student>`, we should expect that we will necessarily have more interactions between these classes, since that is necessary for a cohesive module.

We *need* aggregation to describe the state of an object. However, this doesn't change the fact that we want all interactions between `Course` and other classes to be at the interface level, and not dependent on implementation details. We still want these interactions to be at the data coupling level.


## Composition

Composition and Aggregation will look the same if you simply look at a narrow code view, as they both involve one class possessing another class as a field. The difference is often determined by semantics of intent rather than true syntax. In general:


* **Aggregation** "A aggregates B" - refers to an instance of class, A, "has" one or more instances of Class B. 

* **Composition** - "A is composed of B" refers to a relationship where one or more instances of B are intrinsically part of another class, A. In general, B does not  such that they often cannot be meaningfully seperated. The **composing** class brings together these items, and the items by themselves don't have a meaning outside of their attachment to the composing class. **Composition** relationships are a *subset* of Aggregation relationships.

A way to think of it is that I **aggregate** clothes, but I'm **composed** of body parts. My shirts can be resold and re-used with minimal effort. Both I and my shirts are separable entities that can interact with the world around me separate from each other. However, my body parts cannot be easily removed and attached to others. You wouldn't cut off my arm and try to open a door with *just* my arm. It only functions when attached to the rest of my body. Of course, this is a metaphor, and not actual code, but when designing software models, we often lean on such metaphors.

### Aggregation vs Composition Example

For instance, consider the following:

```java
public class MailingList {
    private String name;
    private String listServAddress;
    private MailingListHistory history;
    private Set<User> subscribers;
    
    public MailingList(String name, String listServAddress) {
        this.name = name;
        this.listServAddress = listServAddress;
        this.history = new MailingListHistory();
        this.subscribers = new HashSet<>();
    }
    
    //getters here
    
    public boolean addSubscriber(User newSubscriber) {
        return subscribers.add(newSubscriber);
    }
    
    public boolean removeSubscriber(User subscriber) {
        return subscribers.remove(subscriber);
    }
    
    public void sendEmail(Email email) {
        for(User user : subscribers) {
            user.sendEmail(email);
        }
        history.add(email);
    }
}
```

In the above, I would say this class **aggregates** `User`s (who come and go, and have broader meaning in the system). That is, a `User` doesn't exist solely as a part of `MailingList`. On the other hand, `MailingListHistory` only makes sense as part of a `MailingList`. This means `MailingList` is **composed** of a `MailingListHistory`.

Again, there are subjective semantics at play here, especially given only a single snapshot of a class. The key takeaway is that both aggregation and composition refer to a one-way relationship where one class possesses one or more instances of another class.

## Association

When discussing relationship naming, the word *association* has different meanings to different authors. Often this breaks down into two definitions, a general definition, and a specific definition.

* General definition - any interacting between classes is an association. That is, it is a synonym for relationship as used in this article
* Specific definition - sometimes called "bidirectional association" two classes are connected in such a way that interaction may be needed both ways. Often, this can be expressed as a mutual aggregation.

Example:

```java
public class Author {
    private List<Book> booksWritten;
    
    public List<Book> getBooksWritten() {
        return new ArrayList<Book>(booksWritten);
    }
}

public class Book {
    private List<Author> authors;
    
    public List<Author> getAuthors() {
        return new ArrayList<Author>(authors);
    }
}
```

Note that an association doesn't require a `List` or any Collection, I'm just semantically accounting for the idea that one book may be written by multiple people, and one author may write multiple books.

The key here is that we may want to travel both directions. I have an `Author`, I want their `Book`s, and I have a `Book`, I want its author(s). Here, neither is specifically *the* owner of the other, but the relationship is mutual.

Once again, differentiating this from *aggregation* and *composition* often relies on semantics, which depend on context and have a degree of subjectivity. In our `MailingListHistory` class above, for example, that class may have a reference to `MailingList` for the sake of referencing the list it is a history for.

```java
public class MailingListHistory {
    private MailingList mailingList;
    
    public MailingList getMailingList() {
        return mailingList;
    }
}
```

However, this relationship is not symmetrical semantically. `MailingListHistory` is still a sub-part of `MailingList` that only makes sense tied to a `MailingList`. So I would still call this composition.

In general, with bidirectional associations, the semantics generally refer to a relatively symmetrical relationship in the sense that neither class possesses the other, but they do reference each other.

There may not always be an exact 100% agreed upon answer to whether a particular relationship is bidirectional association, aggregation, or composition. However, try to keep in mind the semantics of these relationships, and not just the syntax, when describing how two classes relate.


## Realization

A class realizes an interface or abstract by implementing/extending it. Where dependency describes a "uses" relationship, and aggregation describes a "has-a" relationship, realization is an "is-a" relationship. In a "realization", one class (the parent) describes the *behaviors* (in Java, an `interface` or `abstract class`), and the other (the child) implements that behavior.

For example,

```java
public interface Runnable {
    void run();
}

public class MyRunnable implements Runnable {
    @Override
    public void run() {
        //do something
    }
}
```

In this example, `MyRunable` **realizes** `Runnable`. That is, `MyRunnable` "is-a" `Runnable`; it is a concrete implementation of the `Runnable` interface. This also applies to abstract classes as well as interfaces. In this relationship, we would call `Runnable` (the interface) the *parent*, and `MyRunnable` (the concrete implementation) the child.

A realization relationship is tighter coupling, since any changes to the parent's interface will affect all implementations. Because of this, we generally want small, stable interfaces, and rarely want to change their interface. A change, for example, to `Runnable`, like:

```java
public interface Runnable {
    void run(Routine routine);
}
```

...would require changes to `MyRunnable`, as well as *every other implementation* of `Runnable` in our code. Additionally, any class that *uses* an object that implements `Runnable` would also need to change. So introducing an interface/abstact class with multiple implementations does add coupling.

### Abstract Class coupling

Specifically, abstract classes will typically cause even more coupling than interfaces, since abstract classes can have:
* their own fields inherited by the child (which means the child is coupled not just to the interface of the abstract class, but the implementation)
* their own implemented methods (which the child can either re-use or override)
* their own constructors (which a child class *must* invoke in its constructor)

Also, a child class can only implement (*extend*) one class (which it can implement multiple interfaces). This means a child class cannot be coupled to another other classes via extension.

In general, favor small interfaces that describe one only one behavior or a few very cohesive set of behaviors. The larger and interface/abstract class, or the more implementation details in an abstract class, the more likely you will inevitably have to change. And such changes are frequently time-consuming, meaning expensive.

If you find you are implementing abstract classes where *most* of the behavior is concrete, and only one feature of a class is abstract, that's a sign that you probably don't want to use inheritance, and instead want to use aggregation (see below).

### Generalization

Generalization encapsulates all "is-a" relationships. Realization is actually a subset of *generalization*. That is, all realizations are generalizations. However, I listed *realization* first because it typically results in less coupling than other forms of generalization.

Specifically in generalization, a parent describes a *general* behavior, and a child describes a specific behavior, or specialization of the parent. That is, the parent *generalizes* the child. Realization specifically refers to *generalizations* where the parent is an abstract description of behavior and the child an implementation of that behavior.

For example, imagine we design something like Piazza. Here, all users have a log-in action, a list of courses they are enrolled in, and a list of posts they have made. However, Users that are students users can ask Questions, and post in a student answers section, while users that are Professors can post announcements, delete posts from their course, and add and remove users to and from their course.

```java
public abstract class User {
    protected String username;
    protected String passwordHash;
    protected List<Course> courses;
    protected List<Post> posts;

    //getters and setters

    public LogInResponse logIn() { ...}

    public List<Post> getVisiblePosts(Course course) {...}

    public List<Course> getActiveCourses() { ...}

    public void postFollowup(Post post) {...}

    public abstract boolean canViewPost(Post post);
}

public class Student extends User {
    @java.lang.Override
    public boolean canViewPost(Post post) {
        return (this == post.getAuthor() || !Post.isPrivate()) && 
                courses.contains(post.getCourse());
    }

    public void postQuestion(Course course, String question);
    public void postStudentAnswer(Post question);
}

public class Professor extends user {
    @java.lang.Override
    public boolean canViewPost(Post post) {
        return courses.contains(post.getCourse());
    }
    
    public void postNote(Course course, String post);
    public void deletePost(Post post);
    public void addToCourse(User user);
    public void removeFromCourse(User user);
}
```

The idea here is two handle the *general* behavior of a User in the `User` class, and then the specific behaviors of `Student`s and `Professor`s in child classes. This why you only write the general behavior once, and decompose the specific behavior into separate classes. Specifically, when discussing *generalizations* that are not *realizations*, we think of the parent class as either concrete, or a "nearly complete" abstract class (that is, it has fields, implemented methods, etc.)

This is the tightest form of coupling. In fact, in the case where protected variables are shared by the parent and children, this is a form of **common coupling**. Specifically, a field protected field could be changed by a parent, and child, or any class that extends the child. Additionally, changes to a child class may necessitate changes to the parent class, and changes to the parent class typically propagate to all the children (and, when present, any class that extends the children). So, in the above example, `Student` is directly tightly coupled to `User`. However, it is also indirectly coupled to `Professor`, since a change to Professor may end up propagating to `User`, which would propagate down to `Student`. In this way, inheritance, especially of a class with any existing concretions, creates very tight coupling, and is very resistant to change.

## Prefer Aggregation over Inheritance

Polymorphism is a powerful tool, as we discussed in [Benefits of Polymorphism](https://sde-coursepack.github.io/modules/objects/Benefits-of-Polymorphism/). It allows us to create "plug-in" type systems where we can hide which of several implementations of a particular feature were are using, and gives us flexibility. A **caller** can invoke a behavior, without having to be aware of specific implementation details. However, polymorphism requires introducing some form of abstraction, either an `interface` or a parent class (often, but not necessarily, abstract). And this creates a tight coupling:

* The **caller** is coupled to the abstraction as either an aggregation or a dependency
* Any **implementators** are coupled to the abstraction in a realization way

This means that changes to the abstraction's *interface* (using the generic definition of interface, not java 'interface') will have significant ramifications for the **caller** *and* the implementors. As such, we want to make these interfaces as small and stable as possible.

### Example

Consider the following example where there is no inheritance.

```java
public class State{
    private String name;
    private int population;
    private String postalCode;
    private String capitalCity;
}

public class CSVStateReader {
    private List<State> states;
    
    public List<State> getStatesFromCSVFile(String filename) {
        states = new ArrayList<State>();
        FileReader fileReader = new fileReader(filename);
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        String fileContents = getFileContents(bufferedReader);
        states.addAll(parseCSVData(jsonData));
        return states;
    }
    
    public List<State> parseCSVData(String fileContents) {
        String[] lines = lines.split("\n");
        ... //get states from CSV files
        return csvStates;
    }
    
    public String getFileContents(BufferedReader bufferedReader) { ... };
}

public class JSONStateReader {
    private List<State> states;
    
    public List<State> getStatesFromJSONFile(String filename) {
        states = new ArrayList<State>();
        FileReader fileReader = new fileReader(filename);
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        String fileContents = getFileContents(bufferedReader);
        states.addAll(parseJSONData(fileContents));
        return states;
    }
    
    public List<State> parseJSONData(String fileContents) {
        JSONObject jsonObject = new JSONObject(fileContents);
        ... //parse json
        return jsonStates;
    }

    public String getFileContents(BufferedReader bufferedReader) { ... };
}

public class StatePrinter {
    public void printStatesByPopulationDescending(String stateFilename) {
        List<States> states = null;
        if (filename.endsWith(".csv")) {
            var csvReader = new CSVStateReader();
            states = csvReader.getStatesFromCSVFile(stateFilename);
        } else if (filename.endsWith(".json")) {
            var jsonReader = new JSONStateReader();
            states = csvReader.getStatesFromJSONFile(stateFilename);
        } else {
            throw new UnsupportedFileFormatException(stateFilename);
        }
        states.sort((a, b) -> (b.getPopulation() - a.getPopulation));
        states.forEach(System.out::println);
    }
}
```

The above code works, but you notice something. *There's redundant code!* Both `JSONStateReader` and `CSVStateReader` have a lot in common. In fact, the only obvious difference is in their `parseData` function. So, you create an abstraction:

```java
public abstract class StateReader {
    protected List<State> state;
    
    public void getStatesFromFilename(String filename) {
        states = new ArrayList<State>();
        FileReader fileReader = new fileReader(filename);
        BufferedReader bufferedReader = new BufferedReader(fileReader);
        String fileContents = getFileContents(bufferedReader);
        states.addAll(parseData(fileContents));
        return states;
    }

    public String getContents(BufferedReader bufferedReader) { ... };
    
    public abstract List<State> parseData(String fileContents);
}

public class CSVStateReader extends StateReader {
    @Override
    public List<State> parseData(String fileContents) {
        String[] lines = fileContents.split("\n");
        ... //get states from CSV files
        return csvStates;
    }
}

public class JSONStateReader extends StateReader {
    @Override
    public List<State> parseData(String fileContents) {
        JSONObject jsonObject = new JSONObject(fileContents);
        ... //parse json
        return jsonStates;
    }
}

public class StatePrinter {
    public void printStatesByPopulationDescending(String stateFilename) {
        StateReader reader = null;
        if (filename.endsWith(".csv")) {
            reader = new CSVStateReader();
        } else if (filename.endsWith(".json")) {
            reader = new JSONStateReader();
        } else {
            throw new UnsupportedFileFormatException(stateFilename);
        }
        List<State> states = reader.getStatesFromFilename(stateFilename);
        states.sort((a, b) -> (b.getPopulation() - a.getPopulation));
        states.forEach(System.out::println);
    }
}
```

This is an abstraction! Specifically, we took all the *shared* code between `JSONStateReader` and `CSVStateReader` and put it in an abstract parent called `StateReader`. Now, `StatePrinter` can use the abstraction.

But, did this really help us *that* much?

Compare these two implements of `StatePrinter`:

**Without abstraction**

```java
public class StatePrinter {
    public void printStatesByPopulationDescending(String stateFilename) {
        List<States> states = null;
        if (filename.endsWith(".csv")) {
            var csvReader = new CSVStateReader();
            states = csvReader.getStatesFromCSVFile(stateFilename);
        } else if (filename.endsWith(".json")) {
            var jsonReader = new JSONStateReader();
            states = csvReader.getStatesFromJSONFile(stateFilename);
        } else {
            throw new UnsupportedFileFormatException(stateFilename);
        }
        states.sort((a, b) -> (b.getPopulation() - a.getPopulation));
        states.forEach(System.out::println);
    }
}
```

**With abstraction**

```java
public class StatePrinter {
    public void printStatesByPopulationDescending(String stateFilename) {
        StateReader reader = null;
        if (filename.endsWith(".csv")) {
            reader = new CSVStateReader();
        } else if (filename.endsWith(".json")) {
            reader = new JSONStateReader();
        } else {
            throw new UnsupportedFileFormatException(stateFilename);
        }
        List<State> states = reader.getStatesFromFilename(stateFilename);
        states.sort((a, b) -> (b.getPopulation() - a.getPopulation));
        states.forEach(System.out::println);
    }
}
```

Is this really *that* different? This really only saves us one line.

*Ah*, you think, *but look at how we removed duplicate code from our two implementors*.

Here is the duplicate code that we extracted to the parent class:

```java
    public void getStatesFromFilename(String filename){
        states=new ArrayList<State>();
        FileReader fileReader=new fileReader(filename);
        BufferedReader bufferedReader=new BufferedReader(fileReader);
        String fileContents=getFileContents(bufferedReader);
        states.addAll(parseData(fileContents));
        return states;
    }
```

Okay, but is this code really that complicated? Do we really *need* to only have one place in our code responsible for opening a data file, getting the file contents as a String, passing the String to another function, and then returning the output of that function? Notice that the actual *interesting* logic (actually *parsing* the data) is still handled by the sub-class. The part we are duplicating is a much lower complexity.

Also notice that before we introduced our abstraction, `CSVStateReader` and `JSONStateReader` had no interactions with each other whatsoever. Changes to one class would have no bearing at all on how the other class was use. Now, however, they coupled to the same interface:

```java
    public void getStatesFromFilename(String filename);
    public List<State> parseData();
```

But now imagine that instead of reading a local JSON file, we want to support *either* reading JSON from a local file *or* the internet (via a `URL` object). So we now need to either abandon our abstraction, or generalize our abstraction to incorporate the URL input. Our existing interface cannot support that, so we have to change our `getStates` method to take in an `Object` instead of a `String filename`, and use `instanceOf` to check the type. If we want to preserve our abstraction, we have to do something like:

```java
public abstract class StateReader {
    protected List<State> states;
    
    public abstract List<State> getStates(Object resource);
    public abstract List<State> parseData(String contents);
    public String getContents(BufferedReader bufferedReader) { ... };
}

public abstract class StateFileReader extends StateReader {
    public List<State> getStates(Object resource) {
        if (!resource instanceOf String) {
            throw new UnsupportedOperationException("StateFileReaders can only take in String file names");
        }
        String filename = (String) resource;
        states=new ArrayList<State>();
        FileReader fileReader=new fileReader(filename);
        BufferedReader bufferedReader=new BufferedReader(fileReader);
        String fileContents=getContents(bufferedReader);
        states.addAll(parseData(fileContents));
        return states;
    }
    
    public abstract List<State> parseData(String contents);
}

public abstract class StateURLReader extends StateReader {
    public List<State> getStates(Object resource) {
        if (!resource instanceOf URL) {
            throw new UnsupportedOperationException("StateURLReaders can only take in a URL");
        }
        Url url = (URL) resource;
        states=new ArrayList<State>();
        BufferedReader bufferedReader=new BufferedReader(url.openStream());
        String urlContents=getContents(bufferedReader);
        states.addAll(parseData(fileContents));
        return states;
    }

    public abstract List<State> parseData(String contents);
}
```

Notice all three of these classes are *just* abstractions. We haven't even written our concrete classes yet. Here they are:


```java
public class CSVStateReader extends StateReader {
    @Override
    public List<State> parseData(String fileContents) {
        String[] lines = fileContents.split("\n");
        ... //get states from CSV files
        return csvStates;
    }
}

public class JSONStateReader extends StateReader {
    @Override
    public List<State> parseData(String fileContents) {
        JSONObject jsonObject = new JSONObject(fileContents);
        ... //parse json
        return jsonStates;
    }
}

public class JSONWebReader extends StateReader {
    @Override
    public List<State> parseData(String fileContents) {
        JSONObject jsonObject = new JSONObject(fileContents);
        ... //parse json
        return jsonStates;
    }
}

```

First, take in just how complex and fragmented our code now is. We have three layers of abstraction.

* the top Layer has the `protected List<State> states` field and implements `String getContents(BufferedReader bufferedReader)`
* the middle layer has the `getStates` code which creates the `bufferedReader`, calls the`parseData` function, and returns `states`
* the bottom layer implements the `parseData` function.

Even worse, now in our class `JSONFileReader` and `JSONWebReader`, we have duplicate code! Both are separately implementing how the data is parsed. But if the data format is the same, this is redundant code, the very thing we were trying to avoid in the first place! Maybe we can introduce an abstraction to handle parsing the JSON....

Oh, and we haven't taken into account that now we need to write `StatePrinter` to take in an object, and we have to rewrite that if-statement, and..

### It's time to stop

When dealing with inheritance structures, because of the tight coupling between classes, changes produce *substantial* changes. As a general rule, if I find I need to go two layers deep in my inheritance structure to understand what's going on, or even deeper, I start to get confused and find things hard to track. As such, if I'm tempted to build that deep an abstraction that requires that level of decomposition, I stop myself and ask "Is there a way to avoid this."

What if, instead of trying to preserve our abstraction as is, we just reconsider what our abstraction needs to be.

Instead of `List<State> getStates(Object resource)`, what if we completely strip where the data comes from out of our abstraction. We also strip *all* implementation details out of the abstraction completely. No fields, no code, just the interface. Well, now instead of an `abstract class`, we can use an interface. That is, `List<State> getStates()`

How can we do this?

```java
public interface StateSupplier {
    List<State> getStates();
}
```

And now, we create our three implementors:

```java 
public class CSVStateReader implements StateSupplier{
    private String filename;
    
    public CSVStateReader(String filename) {
        this.filename = filename;
    }
    
    public List<State> getStates() {
        //open file, parse data, return list
    }
}
```

```java 
public class JSONFileStateReader implements StateSupplier{
    private String filename;
    
    public CSVStateReader(String filename) {
        this.filename = filename;
    }
    
    public List<State> getStates() {
        //open file, parse data, return list
    }
}
```

```java
public class JSONURLStateReader implements StateSupplier{
    private URL url;
    
    public CSVStateReader(URL url) {
        this.url = url;
    }
    
    public List<State> getStates() {
        //open url, parse data, return list
    }
}
```

And now, let's re-write StatePrinter to simply take in the Reader we need rather than a filename or URL:

```java
public class StatePrinter {
    public void printStatesByPopulationDescending(StateSupplier stateSupplier) {
        List<State> states = supplier.getStates();
        states.sort((a, b) -> (b.getPopulation() - a.getPopulation));
        states.forEach(System.out::println);
    }
}
```

But where do we handle selecting the `StateSupplier` we need? Well, let's make a class who provides all the options as methods which build the objects for us (aka, a **factory**)

```java
public class StateSupplierFactory {
    public StateSupplier getStateSupplierForFilename(String filename) {
        if (filename.endsWith("csv")) {
            return new StateCSVReader(filename);
        } else if (filename.endsWith("json")) {
            return new JSONFileStateReader(filename);
        } else {
            throw new UnsupportFileFormatException(filename);
        }
    }
    
    public StateSupplier getStateSupplierFromURL(URL url) {
        return new JSONURLStateReader(url);
    }
}
```

Now, if we add new file, or new web, formats, we can simply implement the class and add it to an appropriate function (or add a new function) in `StateSupplierFactory`. `StatePrinter` no longer cares *where* the data comes from, it just gets the data from whatever `StateSupplier` is injected.

### Dependency Injection

Looking at `StatePrinter` again:

```java
public class StatePrinter {
    public void printStatesByPopulationDescending(StateSupplier stateSupplier) {
        List<State> states = supplier.getStates();
        states.sort((a, b) -> (b.getPopulation() - a.getPopulation));
        states.forEach(System.out::println);
    }
}
```

Using the arguments, we have directly injected the dependency `stateSupplier`. Thus, rather than asking `StatePrinter` to both "get" it's data *and* "use" it's data, we now ensure the function only does the latter. 

This also gives us benefits with testing. Before, we couldn't mock any of the behavior of our function easily, since we needed to create and use several classes inside of the method. However, but passing in stateSupplier, we can easily mock `stateSupplier` to return a pre-determined hard-coded List of `State` objects in order to test the specific logic of our sorting and printing.

Specifically, rather than using a large inheritance structure, I am using an `interface` that hides only the narrow functionality I care about. And since this interface is no longer tied to where the data is coming from, our interface is more flexible. Additionally, the functions in `StatePrinter` no longer have any dependency on where the data comes from, only the fact that there's a thing call `StateSupplier` that gives them states.

In essence, we have replaced an inheritance structure with a `dependency`. We could also make it an aggregation:

```java
public class StatePrinter {
    private StateSupplier supplier;
    
    public StatePrinter(StateSupplier supplier) {
        this.supplier = supplier;
    }
    
    public void printStatesByPopulationDescending() {
        List<State> states = supplier.getStates();
        states.sort((a, b) -> (b.getPopulation() - a.getPopulation));
        states.forEach(System.out::println);
    }
}
```

In either case, this still allows us to do dependency-injection.

## Conclusion

Inheritance structures are resistant to change because they create tight-coupling between the parent and child classes. Thus, any interface changes create *significant* refactoring problems. In many cases, it would be better to leave classes separate than force an abstraction that combines them. However, when an abstraction is desirable, using a minimal interface to describe the functionality is preferred.

While this may lead to some repetition of boilerplate code (such as several classes opening a BufferedReader), this code is loosely coupled with how it is used, lending itself to be quick to build, and easy to replace.
