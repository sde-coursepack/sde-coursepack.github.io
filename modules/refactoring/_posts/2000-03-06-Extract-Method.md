---
Title: Refactoring - Extracting a Method
---

# Extracting Methods

In this module, we will look at the refactoring practice of extracting methods. We may extract methods for the following reasons:

* Address methods that are too large, breaking them into smaller methods
* Improve the readability of our code by encapsulating low-level logic into a method with an intent-communicating name.
* Removing duplicate code and duplicate logic by extracting the repeated behavior into a single method

---

* TOC
{:toc}

---

## Breaking up big methods

**Function are good at doing one thing and one thing only!**

But what is one thing? Lets look at a practical example.

## Point Distance Example

Consider the following code we mentioned in the Code Smells

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

To break this up, we can use the "extract method" refactor tool in IntelliJ. To use this, you can highlight a section of code in a method you wish to extract, right-click, refactor, and select "extract method":

![](../images/refactoring/refactorMethod.png)

### totalDistance

For starters, one obvious candidate is the last loop in our `main` function, which, given a `List<Point>`, calculates the totalDistance traveled going from the first to the last point.

Doing this, we get:

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

        double totalDistance = getTotalDistance(pointList);

        System.out.printf("Total distance = %.3f", totalDistance);
    }

    public static double getTotalDistance(List<Point> pointList) {
        double totalDistance = 0.0;
        for (int index = 0; index < pointList.size() - 1; index++) {
            Point first = pointList.get(index);
            Point second = pointList.get(index + 1);
            double differenceX = Math.abs(first.getX() - second.getX());
            double differenceY = Math.abs(second.getX() - second.getY());
            double distance = Math.sqrt(differenceX * differenceX + differenceY * differenceY);
            totalDistance += distance;
        }
        return totalDistance;
    }
}
```

Now, we can understand what the purpose of the function `getTotalDistance` is, *and* we could test this function completely indepedent of the rest of `main` (meaning indepedent of opening a file and creating a List<Point>).

In the same way, we can break apart:
1) Opening a `BufferedReader` for a particular filename
2) Building the `List<Point>` from the contents of the file


```java
public class DistanceCalculator {
    public static void main(String[] args) throws IOException {
        String filename = args[0];
        BufferedReader bufferedReader = getBufferedReader(filename);
        List<Point> pointList = getPointListFromReader(bufferedReader);
        double totalDistance = getTotalDistance(pointList);
        System.out.printf("Total distance = %.3f", totalDistance);
    }



    public static BufferedReader getBufferedReader(String filename) throws FileNotFoundException {
        FileReader fileReader = new FileReader(filename);
        return new BufferedReader(fileReader);
    }

    public static List<Point> getPointListFromReader(BufferedReader bufferedReader) throws IOException {
        List<Point> pointList = new ArrayList<>();
        for(String line = bufferedReader.readLine(); line != null; line = bufferedReader.readLine()) {
            String[] splitLine = line.split(",");
            double x = Double.parseDouble(splitLine[0].strip());
            double y = Double.parseDouble(splitLine[1].strip());
            Point point = new Point(x, y);
            pointList.add(point);
        }
        return pointList;
    }

    public static double getTotalDistance(List<Point> pointList) {
        double totalDistance = 0.0;
        for (int index = 0; index < pointList.size() - 1; index++) {
            Point first = pointList.get(index);
            Point second = pointList.get(index + 1);
            double differenceX = Math.abs(first.getX() - second.getX());
            double differenceY = Math.abs(second.getX() - second.getY());
            double distance = Math.sqrt(differenceX * differenceX + differenceY * differenceY);
            totalDistance += distance;
        }
        return totalDistance;
    }
}
```

### Well-Written Prose

Let's now look at the `main` method. For the sake of alignment, I replaced all the data type declarations with `var`

```java
    var filename = args[0];
    var bufferedReader = getBufferedReader(filename);
    var pointList = getPointListFromReader(bufferedReader);
    var totalDistance = getTotalDistance(pointList);
    System.out.printf("Total distance = %.3f", totalDistance);
```

How would you read this code? This is how I would read it, line by line:

1) Get the filename from the first command line argument
2) Get a `BufferedReader` from that filename so that I can read the file
3) Get the list of points from the file `bufferedReader` opened.
4) Calculate the total distance of that list of points
5) Print the total distance

You can see now how `main` defines an understandable high-level procedure, and then each function handles the low-level details of **how** that behavior is implemented. If I need specifics, I can just go to the function in question.

### Not done yet

Consider our `getPointListFromReader` function:

```java
    private static List<Point> getPointListFromReader(BufferedReader bufferedReader) throws IOException {
        List<Point> pointList = new ArrayList<>();
        for(String line = bufferedReader.readLine(); line != null; line = bufferedReader.readLine()) {
            String[] splitLine = line.split(",");
            double x = Double.parseDouble(splitLine[0].strip());
            double y = Double.parseDouble(splitLine[1].strip());
            Point point = new Point(x, y);
            pointList.add(point);
        }
        return pointList;
    }
```

Whenever I see a large loop body, I always will ask myself if it makes sense to move part or all of this body to a function. Here, I'll notice I'm really doing two things for each line from my `bufferedReader`:

1) Getting the x and y values to make a `Point` instance
2) Adding that `Point` to the pointList

Consider that when getting a `Point`, I'm taking a line like `"1.0, 3.5"` and splitting it on the comma to get each value. This idea of "you give me a String and I'll give you a Point" *sounds* like a function, so let's extract it. This gives us:

```java
    public static List<Point> getPointListFromReader(BufferedReader bufferedReader) throws IOException {
        List<Point> pointList = new ArrayList<>();
        for(String line = bufferedReader.readLine(); line != null; line = bufferedReader.readLine()) {
            Point point = getPointFromLine(line);
            pointList.add(point);
        }
        return pointList;
    }

    public static Point getPointFromLine(String line) {
        String[] splitLine = line.split(",");
        double x = Double.parseDouble(splitLine[0].strip());
        double y = Double.parseDouble(splitLine[1].strip());
        Point point = new Point(x, y);
        return point;
    }
```

From here, I'll "tighten up" the function `getPointFromLine` by turning:

```java
  Point point = new Point(x, y);
  return point;
```

Into

```java
  return new Point(x, y);
```

And similarly in for `getPointListFromReader`:

```java
    Point point = getPointFromLine(line);
    pointList.add(point);
```

becomes

```java
  pointList.add(getPointFromLine(line));
```

And so my end result for these functions is:

```java
    public static List<Point> getPointListFromReader(BufferedReader bufferedReader) throws IOException {
        List<Point> pointList = new ArrayList<>();
        for(String line = bufferedReader.readLine(); line != null; line = bufferedReader.readLine()) {
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
```

Now notice that each function handles separate things. `getPointListFromReader` steps through our file opened by `bufferedReader` one line at a time to build our `pointList`. `getPointFromLine` handles turning a single line of our CSV file into a Point instance.

### Distance function

The last "long" function we are left with is `getTotalDistance`:

```java
    public static double getTotalDistance(List<Point> pointList) {
        double totalDistance = 0.0;
        for (int index = 0; index < pointList.size() - 1; index++) {
            Point first = pointList.get(index);
            Point second = pointList.get(index + 1);
            double differenceX = Math.abs(first.getX() - second.getX());
            double differenceY = Math.abs(second.getX() - second.getY());
            double distance = Math.sqrt(differenceX * differenceX + differenceY * differenceY);
            totalDistance += distance;
        }
        return totalDistance;
    }
```

Here, similar to what we did when reading the file, we can separate the individual operation (getting the distance between two `Point`) from the loop that gets the sum.

```java
    public static double getTotalDistance(List<Point> pointList) {
        double totalDistance = 0.0;
        for (int index = 0; index < pointList.size() - 1; index++) {
            Point first = pointList.get(index);
            Point second = pointList.get(index + 1);
            double distance = getDistance(first, second);
            totalDistance += distance;
        }
        return totalDistance;
    }

    private static double getDistance(Point first, Point second) {
        double differenceX = Math.abs(first.getX() - second.getX());
        double differenceY = Math.abs(second.getX() - second.getY());
        return Math.sqrt(differenceX * differenceX + differenceY * differenceY);
    }
```

But notice that this leaves us with a function (`getDistance`) that takes in two points, uses their fields, and produces a result. At this point, I think, that *this really is a `Point` operation*. So, I can move it into the Point class:

```java
class Point {
    private final double x, y;

    Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }
    public double distanceTo(Point point) {
        double differenceX = Math.abs(this.x - point.x);
        double differenceY = Math.abs(this.y - point.y);
        return Math.sqrt(differenceX * differenceX + differenceY * differenceY);
    }
}
```

Now, I can re-write the loop body of `totalDistance` as:

```java
    public static double getTotalDistance(List<Point> pointList) {
        double totalDistance = 0.0;
        for (int index = 0; index < pointList.size() - 1; index++) {
            Point first = pointList.get(index);
            Point second = pointList.get(index + 1);
            double distance = first.distanceTo(second);
            totalDistance += distance;
        }
        return totalDistance;
    }
```

So by breaking up my functions, I noticed that I had a behavior in my class `DistanceCalculator` that really described a behavior of the `Point` class. By moving the method, now the `Point` class handles determining the distance between two points, which makes more sense.

### Level of abstraction

As a last note, let's take a final look at our `main` method:

```java
public class DistanceCalculator {
    public static void main(String[] args) throws IOException {
        var filename = args[0];
        var bufferedReader = getBufferedReader(filename);
        var pointList = getPointListFromReader(bufferedReader);
        var totalDistance = getTotalDistance(pointList);
        System.out.printf("Total distance = %.3f", totalDistance);
    }
    ...
}
```

It's a bid odd here that we have the first and last line being "low-level" implementation details (accessing a specific value in an array, printing a number in a specific format). In general, we want our functions to stay at the same level of abstraction, rather than bouncing up and down to low-level and high-level. As a result, I extract methods for the first and last line so that I can use the function name to communicate *high-level intent*, as well as isolate the details of "Where the filename comes from" and "how to display the total distance" to their own functions:

```java
    public static void main(String[] args) throws IOException {
        var filename = getFilenameFromArguments(args);
        var bufferedReader = getBufferedReader(filename);
        var pointList = getPointListFromReader(bufferedReader);
        var totalDistance = getTotalDistance(pointList);
        displayTotalDistance(totalDistance);
    }

    public static String getFilenameFromArguments(String[] args) {
        return args[0];
    }

    private static void displayTotalDistance(double totalDistance) {
        System.out.printf("Total distance = %.3f", totalDistance);
    }
```

The advantage of this isn't just improving the readability. Now, for example, if my command line arguments have to change such that my filename isn't necessarily at argument 0, I have already isolated the `main` function from that change. I would only need to make changes to `getFilenameFromArguments`. Similarly, if the display format needs to change, that change is isolated to the `displayTotalDistance` method.


## We don't write this way the first time.

To be absolutely clear, I wrote the first code you saw (the giant main method) on my first pass. I do not try to "clean up" the code as I'm writing. In the same way as writing prose, we focus on writing the first pass, and then edit on subsequent passes.

The cleaner version we just wrote together was my first attempt at cleaning up code in this project. And while this article was long, the actual process of extracting methods can be rather quick with practice. A large part of this being so fast is because of the tools in the IntelliJ IDE being so robust.