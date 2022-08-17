---
Title: Build Tools
---

# Build Tools

In this module, we will discuss using build tools. We will start in this unit by talking about what Build tools are and why we need
them. We will then specifically look at `gradle` in the next until.

## The problems

### Using external libraries

Imagine you are building a webapp that uses the weather. You use a public weather
API like the one provided by the [National Weather service](https://www.weather.gov/documentation/services-web-api)
to get any weather warnings in a state. The National Weather Service provides
an API where you can specify a state by postal code, and that will give you
all of the watches, warnings, and advisories in the area. Here is what
that looks like:

[https://api.weather.gov/alerts/active?area=VA](https://api.weather.gov/alerts/active?area=VA)

![img.png](../images/1/weather_json.png)

Now, you have two choices:
1. Write your own code to parse this data
2. Use an external library, like [org.json](https://www.json.org/json-en.html) and one of many tutorials to parse it for you

If you wisely decide to do it the second way (after all, why reinvent the wheel when
there is already such a perfectly good wheel with great tutorials). But, how
do you actually add the library to our code?

We cannot just add `import org.json` at the top of our code, because org.json
is not a built-in Java Library.

Well, we could manually download the [json library here](https://mvnrepository.com/artifact/org.json/json/20220320).
Then, in IntelliJ go to File -> Project Structure -> Libraries -> Click the plus
sign in the top left -> Java -> Browse to our download location -> Add the library...

Man, this sounds like a lot of work. Further, imagine we are working with other people.
Let's imagine we have a team of 8 programmers. All 8 have to add the library the same way.
All 8 also need to make sure they are getting the same version.

What happens if the library changes because of a bug fix, security patch, or new
features? Well, everyone had to repeat that process again.

**Wouldn't it be nice if there were a *one-step* process to add an external
library that can be shared with everyone to ensure everyone has the same version
of the same external libraries with no effort required?**

**Wouldn't it be nice if we could update to new versions of external libraries
by changing only *one line* in our entire project?**

### Generating a jar file

Also, what if we want to generate a Jar file? Well, we haven't talked about how to
set that up in IntelliJ yet. We have shown how to build a jar file in the Java->Terminal and Java
module, but that process has some problems.

1) It's manual
2) You have to rerun the command every time you want to rebuild the jar
3) Even if we want to recompile only one or two classes, the way we showed requires recompiling the entire project
4) Everyone has to follow a complex script to generate a jar file.

**Wouldn't it be nice if we could simply and easily setup a *build process* that, with
only one command handles generating our output .jar file in an efficient way**


## What build tools can do

Build tools can do both of the above:
1) Greatly simplify the process of downloading and using external libraries
2) Greatly simplify the process of updating to new libraries
3) Greatly simplify the process of building a Java project into a distributable Jar file.

## What we will use in this class

In this class, we will be using Gradle, a modern Java build tool that has a script,
`build.gradle`, which describes the process for getting external libraries, building
the .jar distributable file, testing, and more.

## Java Build Tools to know about

Here is a brief list of Java build tools to be *familiar* with. **You do not need
to know how to use all 4 of these,** just be familiar enough to know that they are build
tools, and how common they are today.

### make

[Make](https://www.gnu.org/software/make/), developed by the Free Software Foundation
as part of the GNU project, is one of the oldest build automation tools. A ``makefile``
typically listed a set of commands (often terminal commands) that would be executed
to generate a build file. It was popular early, but is not uniquely configured
for any one programming language.

Make is still widely used in many projects because of it's low barrier to entry and
ease of use. However, the high degree of developer effort make it not ideal for
very large projects like we often have in Java.

### ant

Built by the [Apache Software Foundation](https://www.apache.org/), a very
large open-source foundation, [ant](https://ant.apache.org/) ("Another Neat Tool") 
was the most popular early Java build tool.

It used XML configuration files to define the build process. You can find
an existing tutorial in [the official documentation here.](https://ant.apache.org/manual/tutorial-HelloWorldWithAnt.html)

Ant is flexible, as it allows the user to specify their project layout (where source
and build files go, what exact command to execute in what order), but this flexibility
came at the cost of a high manual effort to write and maintain the build script, and
each script wasn't inherently re-usable because of the high level of customization.

Ant's biggest weakness is that it doesn't have any means of managing dependencies (like)
external libraries.

### maven

By the late 2000s and into the 2010s, [maven](https://maven.apache.org/), also
built by the Apache Software Foundation, grew to be more popular than ant. Like ant
Maven uses XML files for configuration. However, maven projects also have conventions
and standards, which means Maven scripts are inherently re-usable and much easier
to construct.

While ant was a build-tool (effectively read a script to build output files), Maven
goes one step further as a project-management tool.

One of the biggest advantages in Maven is that it added dependency management. You
can add a dependency, like an external library, to Maven simply by modifying
the build file, which is `pom.xml` by adding to the `<dependencies>` list:

```xml
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20220320</version>
</dependency>
```

After adding this, the next time you build, Maven will automatically download
the .jar file for this library *and* add it to your project's set of libraries.
You as a human don't need to do anything else. This means 1 person on the team
can add a new library simply by adding the appropriate dependency information
to the build file `pom.xml`, then simply share their `pom.xml` file and everyone
on the team will automatically download the same version of all dependencies.

To assist with this, the central Maven Repository contains tons of Java .jar libraries
that you can download. For example, the dependency above (org.json) [links to
here.](https://search.maven.org/artifact/org.json/json/20220320/bundle)

### gradle

This will be the build tool we focus on in this class. It is designed
to be very easy to use and modify. We will discuss it in detail
in the next module.

It has the same features as Maven, but a shorter and easier to read syntax
than XML. It also has a feature that allows you to run the build script
without having to download the gradle software itself.

## Conclusion

Build tools can make it easier to distribute source code, onboard new
developers, handle external dependencies, and build output files. In this
class, we will be using gradle, which we cover in detail in the next module.