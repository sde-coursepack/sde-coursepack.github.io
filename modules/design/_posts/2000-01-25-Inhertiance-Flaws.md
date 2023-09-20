---
Title: Inheritance Flaws
---

# "Prefer Aggregation over Inheritance"

The above phrase, "Prefer Aggregation over Inheritance", has become a common expression. In this module, I'll look at a hypothetical code base which can show the difficulties inheritance has specifically with responding to change.


* TOC
  {:toc}


## Confession

I have a confession to make.

I use to try to use inheritance all the time. I would force abstract classes onto concrete classes I didn't even have a plan to extend more than once. Because I thought "that's what code should do!" By making everything an abstraction up front, and by re-using shared code through inheritance, I was writing ***"Good Code!"*** And now, I almost never use it at all.

Inheritance, which was a core idea of Java, advertises itself as a time saver: when two or more classes have shared code, extract the shared code to a parent class, and then have the child class extend, only implementing and override their own unique behavior. This is an Object-Oriented way of implementing DRY: don't repeat yourself. However, after several years of coding, and changing my style, and learning, and seeing more code...

**I and many others think inheritance was probably a bad idea.** I don't want to say *everyone* thinks this. There are champions of inheritance. But I think when many people reach for inheritance early in a process to "be DRY" and end up frequently regretting it as needs change.

In fact, [Go](https://go.dev/), often called the much more searchable "golang" after it's original domain name, is a programming language designed by Google that, while not meteoric, has been growing in popularity. It has many similarities to Java: 

* static typing (variables, once declared, cannot change types)
* a `type` system which is very similar to classes, with fields, methods, and constructors acting on `type`s, as well as the `subject.verb(object)` syntax found in Java and similar languages.
* polymorphism via interfaces, where interfaces define, but do not implement, method signatures. `types` can implement these method signatures, though unlike Java, such implementations are inferred rather than explicitly declared (that is, no `implements` keyword)

However, go *doesn't support inheritance*, and this was an intentional decision by the developers. Rather, Go encourages *aggregation over inheritance*.

## Code reuse

Consider different ways we could re-use code. Take into account, specifically, a `BufferedReader`. Here is me reading a file with a *subclass* of BufferReader:

```java
public class FullFileReader extends BufferedReader {
  public FullFileReader(Reader in) {
    super(in);
  }

  public List<String> getFileAsListOfLines() throws IOException {
    return lines().toList();
  }
}

public class Demo {
  public static void main(String[] args) throws IOException {
    String filename = args[0];
    FileReader fileReader = new FileReader(filename);
    FullFileReader fullFileReader = new FullFileReader(fileReader);
    List<String> contents = fullFileReader.getFileAsListOfLines();
  }
}
```

Note that `fullFileReader` can also do everything a BufferedReader can do, so the below code does the same thing.

```java
public class Demo {
  ...
    
  public static List<String> getFileContents(String filename) throws IOException {
      FileReader fileReader = new FileReader(filename);
      BufferedReader fullFileReader = new FullFileReader(fileReader);
      return fullFileReader.lines()
            .toList();
  }
}
```

So...all we added to `BufferedReader` was: `getFileAsListOfLines`, which does the same thing as `bufferedReader`'s `lines().toList()`. Does this minor addition warrant the complexity of having to understand the rest of `FullFileReader` is in another class, making it harder to learn how to use the class?

But if I **really** want a single class that extends the behavior of `BufferedReader` to give me a simple function `getFileAsListOfLines`, what if, instead, I just did this:

```java
public class FullFileReader {
    BufferedReader bufferedReader;
    
    public FullFileReader(BufferedReader bufferedReader) {
        this.bufferedReader = bufferedReader;
    }
    
    public List<String> getFileAsListOfLines() throws IOException {
        return bufferedReader.lines()
                .toList();
    }
}
```

That is, replace the *inheritance* with an Aggregation. Note that if I do this, our Demo `main` method:

```java
public class Demo {
    public static void main(String[] args) throws IOException {
        String filename = args[0];
        FileReader fileReader = new FileReader(filename);
        FullFileReader fullFileReader = new FullFileReader(fileReader);
        List<String> contents = fullFileReader.getFileAsListOfLines();
    }
}
```
...still works exactly the same. Only now, `FullFileReader` is a *composition* of BufferedReader rather than a child class. This means if, for whatever reason, `BufferedReader` changed the interface of `lines()`, we still get the benefit of encapsulating this change inside of `FullFileReader`, but we don't have to worry about if `BufferedReader` adds an abstract method that the child must implement, for example. Because `FullFileReader` is no longer *extending* `BufferedReader`, and inheriting a ton of hard to find code/fields/etc., it's just *using* it.

In short, before considering adding functions to a class, consider just adding the class to your functions.

## The dangers of inheritance

Polymorphism is a powerful tool, as we discussed in [Benefits of Polymorphism](https://sde-coursepack.github.io/modules/objects/Benefits-of-Polymorphism/). It allows us to create "plug-in" type systems where we can hide which of several implementations of a particular feature were are using, and gives us flexibility. A **caller** can invoke a behavior, without having to be aware of specific implementation details. However, polymorphism requires introducing some form of abstraction, either an `interface` or a parent class (often, but not necessarily, abstract). And this creates a tight coupling:

* The **caller** is coupled to the abstraction as either an aggregation or a dependency
* Any **implementators** are coupled to the abstraction in a realization way

This means that changes to the abstraction's *interface* (using the generic definition of interface, not java 'interface') will have significant ramifications for the **caller** *and* the implementors. As such, we want to make these interfaces as small and stable as possible.

## Example

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

## It's time to stop

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

## Dependency Injection

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

For this reason, I have basically stopped using inheritance entirely. I still use interfaces when I want to leverage polymorphism, but I keep my interfaces as small and simple as possible. Rather than prioritizing avoiding code repetition bad, I prioritize reducing coupling, and only seek to avoid *knowledge* repetition. And if I want to re-use code, I use aggregation or dependency injection to re-use it.

While this may lead to some repetition of boilerplate code (such as several classes opening a BufferedReader), this code is loosely coupled with how it is used, lending itself to be quick to build, and easy to replace.
