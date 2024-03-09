---
Title: Testing with Reflections
---

# Testing with Reflections

This module will discuss using the reflection feature in the Java language in order to interact with private information in testing.

* TOC
{:toc}

## Reflection Overview

Reflection is an aptly named tool that a Java program can use to "reflect" upon itself and retrieve information at runtime about the objects and classes that it consists of. This includes a class's fields, methods, and constructors, as well as any annotations that have been applied to it (e.g. `@Override`).

If you have written any JUnit testing code, you have already used and benefited from reflection! Annotations like `@Test` and `@BeforeEach` are used by the JUnit framework to collect the methods you wish to be called during testing and identify what order to call them in, and others like `@DisplayName` and `@ParameterizedTest` are used to attach information to those test methods.

Reflection's most relevant feature for testing is its ability to bypass access modifiers like `private` or `protected`. This allows for getting/setting any state variables and invoking ordinarily inaccessible methods. This may sound great, but this particular use of reflection is not without cost and is thus generally advised against (more on this later). That said, it's valuable to familiarize yourself with reflection in case you come across it in the wild or find a valid justification for using it yourself.

Reflection features are provided in the [`java.lang.reflect`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/reflect/package-summary.html) package. However, almost none of the classes in this package have public constructors, so the use of these features first requires a reference to an object of type [`Class<T>`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Class.html). We will see how this works in the examples to follow.

## Working Example

Let's say that we want to test the following class:

```java
package edu.virginia.cs.reflectiondemo;

public class Person {
    private int socialSecurityNumber;

    public Person() {
        socialSecurityNumber = generateSocialSecurityNumber();
    }

    private int generateSocialSecurityNumber() {
        ...
    }

    private boolean validateSocialSecurityNumber() {
        ...
    }
}
```

As in real life, we want to provide very minimal access to an individual's Social Security number to avoid tampering/misuse, and thus have made relevant fields/methods `private`. However, given its importance, we also have some motivation to test that it is being generated and validated correctly, and that its value is not being incorrectly mutated by any of the methods in `Person`.

## Testing Private Members

### Getting a Class Reference

As mentioned previously, the entry point for using reflection is to get a `Class` object representing the class to be reflected upon. In testing, there are two primary ways of doing this:

#### ClassName.class

If the type is available to reference by name, we can use the `.class` syntax:

```java
Class<Person> personClass = Person.class;
```

#### Class.forName()

If we cannot reference the type by name (usually due to the class being `protected` or `private`), we can use the static `forName()` method in `Class` to retrieve it:

```java
try {
    Class<?> personClass = Class.forName("edu.virginia.cs.reflectiondemo.Person");
} catch (ClassNotFoundException e) {
    throw new RuntimeException(e);
}
```

Note that the expected syntax for the class identifier is `<full.package.name>.<ClassName>`.

The generic type parameter for `Class<T>` is the wildcard `?` – in this case we can't reference the type by name and thus cannot write `<Person>` without a compiler error, so we tell Java to *"accept any type here"* with `?`.

Finally, `Class.forName()` throws a checked `ClassNotFoundException` that must be handled. Many of the reflection operations throw checked exceptions; see the [checked exceptions](https://sde-coursepack.github.io/modules/refactoring/Exceptions-Best-Practices/#checked-exceptions) section of this course pack for details on dealing with them. In this module, we will re-throw them as unchecked exceptions for simplicity's sake.

### Calling Methods

Let's say we want to test the return value of the private `generateSocialSecurityNumber()` method in our `Person` class and verify that it is 8 digits long. To access and call it, we can write the following test:

```java
class PersonTest {
    @Test
    void generateSocialSecurityNumber_eightDigits() {
        Person testPerson = new Person();
        Class<Person> personClass = Person.class;
  
        int socialSecurityNumber;
        try {
            Method generateSocialSecurityNumberMethod = personClass.getDeclaredMethod("generateSocialSecurityNumber");
            generateSocialSecurityNumberMethod.setAccessible(true);
            socialSecurityNumber = (int) generateSocialSecurityNumberMethod.invoke(testPerson);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
  
        int socialSecurityNumberLength = String.valueOf(socialSecurityNumber).length();
        assertEquals(8, socialSecurityNumberLength);
    }
}
```

The `getDeclaredMethod(String name)` method on `personClass` will search for a method in that class that has a name matching the provided string, and will include `private` and `protected` methods in its search. There is another method, `getMethod(String name)`, but this will only retrieve `public` methods, so it is not suitable here. If a match is found, then a [`Method`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/reflect/Method.html) object is returned.

We then override the access permissions for this reference to the `generateSocialSecurityNumber()` method by calling `Method.setAccessible()` with the flag set to `true`. This will tell Java to skip any access checks when the method is called via this particular `Method` object.

We can now use `Method.invoke()` to call the method. Because `generateSocialSecurityNumber()` is an instance method (i.e. not static), it needs an instance of its declaring class in order to be called, just like how we regularly call instance methods. So, the `testPerson` object is passed as the first argument to `invoke()`. If the method being invoked is static, then this argument is still required but will be ignored, and so passing `null` will suffice.

The return value of `invoke()` is always of type `Object`, so an explicit cast is needed to convert it to the type we expect the method to return. Here, we cast the value to an `int`.

If the method declares parameters, then appropriate arguments also need to be supplied when invoking it. For example, if the method signature was `generateSocialSecurityNumber(String name, boolean unique)`, then we would provide those arguments similarly to regular method calling:

```java
generateSocialSecurityNumberMethod.invoke(testPerson, "Jane Doe", true);
```

### Setting and Getting Fields

Now, we want to test `validateSocialSecurityNumber()`. However, we don't have any way of manually setting the value for `socialSecurityNumber` to cover various partitions of valid and invalid SSNs, as the `Person` class does not provide any public access to that field. We will need to use reflection to perform this access instead:

```java
class PersonTest {
    @Test
    void validateSocialSecurityNumber_invalid_lessThanEightDigits() {
        Person testPerson = new Person();
        Class<Person> personClass = Person.class;
  
        int input = 12345;
        boolean isValid;
        try {
            Field socialSecurityNumberField = personClass.getDeclaredField("socialSecurityNumber");
            socialSecurityNumberField.setAccessible(true);
            socialSecurityNumberField.set(testPerson, input);
    
            Method validateSocialSecurityNumberMethod = personClass.getDeclaredMethod("validateSocialSecurityNumber");
            validateSocialSecurityNumberMethod.setAccessible(true);
            isValid = (boolean) validateSocialSecurityNumberMethod.invoke(testPerson);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
  
        assertFalse(isValid);
    }
}
```

The approach here is similar to that used for methods. `getDeclaredField(String name)` will search through the class's fields for a match, ignoring access modifiers, and return a [`Field`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/reflect/Field.html) object if one is found. We then call `Field.setAccessible(true)` to suppress access checks to this field. Finally, we can call `Field.set()` to set the Social Security number by passing in the test object (`socialSecurityNumber` is not static and thus needs an object reference in order to be set), and the desired value.

`Field` also provides specific implementations of `set()` for primitive values, which can be used like so:

```java
socialSecurityNumberField.setInt(testPerson, 12345);
```

We may also want to get the value of the field after `validateSocialSecurityNumber()` has been invoked in order to verify whether it has been mutated. To do this, we extend our test to the following, using `Field.get()`:

```java
class PersonTest {
    @Test
    void validateSocialSecurityNumber_invalid_lessThanEightDigits() {
        Person testPerson = new Person();
        Class<Person> personClass = Person.class;
  
        int input = 12345;
        boolean isValid;
        int currentSocialSecurityNumber;
        try {
            Field socialSecurityNumberField = personClass.getDeclaredField("socialSecurityNumber");
            socialSecurityNumberField.setAccessible(true);
            socialSecurityNumberField.set(testPerson, input);
    
            Method validateSocialSecurityNumberMethod = personClass.getDeclaredMethod("validateSocialSecurityNumber");
            validateSocialSecurityNumberMethod.setAccessible(true);
            isValid = (boolean) validateSocialSecurityNumberMethod.invoke(testPerson);
    
            currentSocialSecurityNumber = (int) socialSecurityNumberField.get(testPerson);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
  
        assertFalse(isValid);
        assertEquals(input, currentSocialSecurityNumber);
    }
}
```

Note that, similar to invoking methods, when using `Field.get()` an object reference to access the field on must be provided. Here, that is `testPerson`.

`Field.get()` always returns a value of type `Object`, and so that value also needs to be explicitly cast. However, like with `set()`, `Field` provides specific implementations of `get()` for primitives, and we can change the relevant line in our test to the following in order to remove the cast:

```java
int currentSocialSecurityNumber = socialSecurityNumberField.getInt(testPerson);
```

## Issues

### Tightly Coupled Tests

As you were reading the test code, the use of string literals for method and field names may have raised some questions. What if another member of our team decided to change the member's name, or remove it entirely? What if the parameters of a private method were to change, how would our use of `invoke()` be affected?

If these were regular method calls or field usages, our code would fail to compile, and the issue would be quickly identifiable and solvable. However, when reflection operations fail, they do so at runtime, and become much harder to trace when not isolated to the class in question. While we can design our tests defensively to handle these exceptions, we still can't expect that these private members won't ever change; the private methods and fields were never part of the `Person` class's outward-facing API, and so it would be acceptable to change these implementation details without considering whether any code outside the class relied on it.

The use of reflection in this way has thus *tightly coupled* our tests with our implementation: changes in the implementation require changes in the tests in order for them to function as designed. Introducing this sort of overhead imposes inefficiencies in maintainability that grow exponentially with the size and complexity of a software product, and is thus the primary reason why the use of reflection for bypassing access modifiers is generally discouraged.

### Performance Overhead

A lesser but still relevant issue with the use of reflection in testing is reflection's poor performance. In comparison to regular method and field usage, reflection involves significantly more operations such as validating access permissions and performing [autoboxing](https://docs.oracle.com/javase/tutorial/java/data/autoboxing.html) for primitive values, and loses many of the compiler and runtime optimizations that Java typically can provide ([this article](https://blogs.oracle.com/javamagazine/post/java-reflection-performance) from Oracle's *Java Magazine* provides an excellent breakdown if you're interested in learning more). While test performance is less important than the actual program's performance, slow tests do impede build times and thus impede the development process as a whole.

A quick improvement for reflection performance is to minimize the number of times any reflection operation is called. Let's refactor our tests so far to achieve this:

```java
class PersonTest {
    static Class<Person> personClass;
    static Method generateSocialSecurityNumberMethod;
    static Method validateSocialSecurityNumberMethod;
    static Field socialSecurityNumberField;
  
    Person testPerson;
  
    @BeforeAll
    static void initializePersonClassInformation() {
        personClass = Person.class;
        try {
            generateSocialSecurityNumberMethod = personClass.getDeclaredMethod("generateSocialSecurityNumber");
            validateSocialSecurityNumberMethod = personClass.getDeclaredMethod("validateSocialSecurityNumber");
            socialSecurityNumberField = personClass.getDeclaredField("socialSecurityNumber");
            AccessibleObject[] personClassMembers = { generateSocialSecurityNumberMethod, validateSocialSecurityNumberMethod, socialSecurityNumberField };
            AccessibleObject.setAccessible(personClassMembers, true);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
  
    @BeforeEach
    void setupTestPerson() {
        testPerson = new Person();
    }
  
    @Test
    void generateSocialSecurityNumber_eightDigits() {
        int socialSecurityNumber;
    
        try {
            socialSecurityNumber = (int) generateSocialSecurityNumberMethod.invoke(testPerson);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    
        int socialSecurityNumberLength = String.valueOf(socialSecurityNumber).length();
        assertEquals(8, socialSecurityNumberLength);
    }
  
    @Test
    void validateSocialSecurityNumber_invalid_lessThanEightDigits() {
        int input = 12345;
        boolean isValid;
        int currentSocialSecurityNumber;
    
        try {
            socialSecurityNumberField.setInt(testPerson, input);
            isValid = (boolean) validateSocialSecurityNumberMethod.invoke(testPerson);
            currentSocialSecurityNumber = socialSecurityNumberField.getInt(testPerson);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    
        assertFalse(isValid);
        assertEquals(input, currentSocialSecurityNumber);
  }
}
```

The JUnit tag `@BeforeAll` designates a method (which must be static) to run *once* before any tests are performed. Since we know that the `Person` class's information won't change while our tests are running, we can retrieve that information once and reuse it across all our tests to avoid recollecting redundant information. Additionally, we can turn off access checks on all of our reflected methods and fields in one place with `AccessibleObject.setAccessible()` by providing an `AccessibleObject` array (which `Method` and `Field` both extend). 

Note that once we retrieve the `Method` or `Field` reference for one member of a class, we can reuse that reference for any given instance of that class. For example:

```java
Person personA = new Person();
Person personB = new Person();

personASocialSecurity = socialSecurityNumberField.getInt(personA);
personBSocialSecurity = socialSecurityNumberField.getInt(personB);
```
