---
Title: Abstraction
---

# Abstraction

The idea of abstraction is that all behavior that belongs to a particular module should be handled through that modules interface.

## Interface vs Implementation

We can see the idea of implementing in modern electric cars. For instance, consider the picture below of the cabin of a Tesla:

![img.png](../images/concepts/tesla_cabin.png)

In this image, you can see a steering wheel. Below the steering wheel, you can see a brake pedal on the left, and an accelerator on the right. Often, the right pedal is referred to as the "gas pedal". This is because in an internal combustion engine car, the pressing the pedal down causes more gasoline ("gas" being an American parlance for gasoline) and oxygen into the engine, allowing for a more powerful explosion that causes the engine to turn faster. However, in an electric car, there is no "gas". However, for most American drivers, if I asked "which pedal is the gas pedal", they would say the pedal on the right. That's because we don't think of the pedal based on **how it works**; instead, we think of the pedal based on **what it does**. That is, we care about the **interface**, not the **implementation**.

That is, modern electric cars try to mimic the interface that drivers are used to. This allows drivers to be able to drive an electric car the same way they drive an internal combustion or hybrid car. This means a driver can switch from one to the other, and have to understand as little new information as possible.

## Violation of Abstraction

Consider the following partial classes:

```java
public class Student {
    private List<Course> courseList;

    public List<Course> getCourseList() {
        return courseList;
    }
}

public class Course {
    List<Student> roster;
    
    public void addStudent(Student student) {
        roster.add(student);
    }
}

public class Registrar {
    public void registerStudentForCourse(Student student, Course course) {
        course.addStudent(student);
        List<Course> courseList = student.getCourseList();
        courseList.add(student);
    }
}
```

Does this look fine? Well, one problem is that specifically the last two lines of code in `registerStudentForCourse` - namely, this assumes the following:

1) That a student's list of courses are, in fact, a List
2) That the list returned by `getCourseList` is **mutable**, and changes to that List affect student.

This is a violation of abstraction because *we must assume details not included in the interface*! More specifically, this is a violation of the **Law of Demeter**, also know the *principle of least knowledge*. To see how these last two lines are wrong, let's consider the following alteration:

```java
public class Student {
    private List<Course> courseList;

    public void addCourse(Course course) {
        courseList.add(course);
    }
}

public class Course {
    List<Student> roster;
    
    public void addStudent(Student student) {
        roster.add(student);
    }
}

public class Registrar {
    public void registerStudentForCourse(Student student, Course course) {
        course.addStudent(student);
        student.addCourse(course);
    }
}
```

Now, you'll notice that we are no longer making **any** assumption about the nature of courseList, and in fact the method `registerStudentForCourse` is no longer even **aware** of courseList. This is good, because courseList is an implementation detail.

In general, all interactions with another module should be made through an interface. A general guideline here is:

"If module A has an instance of or uses Module B, and module B  an instance of or uses Module C, module A should never directly interact with Module C."

## Using Information Hiding

As we saw in the above example, abstraction goes beyond just `public`, `private`, and `protected`. In general, we should