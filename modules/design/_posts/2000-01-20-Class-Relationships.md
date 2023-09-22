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

## Conclusion

In this module, we looked at different types of commonly discussed class relationships in Java. Of course, there is naturally nuance and semantics in some of these differences. However, understanding these relationships, how they are implemented, and the consequences that create in regard to coupling is critical to making strong design decisions.

