---
Title: Optimization Example
---

* TOC
  {:toc}


# Source Code Example

For this section, we will use [this source code example](https://github.com/sde-coursepack/EfficiencyExample) at two ways to store a list of coordinates and calculate the total distance between them.

For this code, we define the idea of a `Point` as two coordinates, and `x` and a `y`. I include the class Constructor and distance function below.

## [Point.java](https://github.com/sde-coursepack/EfficiencyExample/blob/master/src/main/java/edu/virginia/cs/sde/efficiency/Point.java)

```java
public class Point {
    private double x, y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }
    ...
    //Euclidean distance
    public double distanceTo(Point point) {
        double xDiff = this.getX() - point.getX();
        double yDiff = this.getY() - point.getY();
        return Math.sqrt(xDiff * xDiff + yDiff * yDiff);
    }
}
```

Right out of the gate, this seems like an obvious way to represent points, as instances of a class `Point`. This would be easy to communicate, almost anyone looking at this class would understand what it does, the purpose of each method, and how it works.

We might also imagine a `Path` where we add `Point`s and then can calculate a distance, draw it on a map, whatever. We can define the abstract interface of a `Path` below:

## [Path.java](https://github.com/sde-coursepack/EfficiencyExample/blob/master/src/main/java/edu/virginia/cs/sde/efficiency/Path.java)

```java
public interface Path {
    void add(double xCoordinate, double yCoordinate);
    void add(Point point);
    Point get(int index);
    int size();
    double totalDistance();
}
```



You'll notice I have two different `add` methods, one which adds two coordinates as `double` values, and one that uses a `Point` instance. However, from the perspective of someone looking at this code, I would view these methods as doing the same thing. This will, however, become relevant later, so put a mental pin there.

Let's take a look as an "obvious" way to implement Path, using an `ArrayList<Point>`:

## [PointListPath.java](https://github.com/sde-coursepack/EfficiencyExample/blob/master/src/main/java/edu/virginia/cs/sde/efficiency/PointListPath.java)

```java
public class PointListPath implements Path {
    private final List<Point> points;

    public PointListPath(ArrayList<Point> points) {
        this.points = points;
    }
    public PointListPath() {
        points = new ArrayList<>();
    }

    public PointListPath(int initialCapacity) {
        points = new ArrayList<>(initialCapacity);
    }

    @Override
    public void add(double xCoordinate, double yCoordinate) {
        points.add(new Point(xCoordinate, yCoordinate));
    }

    @Override
    public void add(Point point) {
        points.add(point);
    }

    @Override
    public Point get(int index) {
        return points.get(index);
    }

    @Override
    public int size() {
        return points.size();
    }

    @Override
    public double totalDistance() {
        double totalDistance = 0.0;
        //Iterate through all but the last point, getting distance to next point
        for (int i = 0; i < size()-1; i++) {
            Point firstPoint = points.get(i);
            Point secondPoint = points.get(i + 1);
            totalDistance += firstPoint.distanceTo(secondPoint);
        }
        return totalDistance;
    }
}
```

This seems a pretty straightforward implementation, letting the `ArrayList points` handle most of the work, and using a simple accumulator `for` loop to get the distance. And this approach is fine! I would be happy to see a student write this code, especially with [useful test-cases](https://github.com/sde-coursepack/EfficiencyExample/blob/master/src/test/java/edu/virginia/cs/sde/efficiency/PointListPathTest.java)!

A quick note about the two Constructors: The `ArrayList<E>()` constructor, when creating an empty `ArrayList`, creates an array of size 10. Remember that if the `ArrayList` needs to handle more than 10 elements, it doubles the size of the underlying array. While this by itself is an **O(n)** operation, this still means that `ArrayList.add` still has an amortized constant runtime. That said, if we know how large we need our `ArrayList` to be ahead of time, we can get a small optimization by specifying the `ArrayList<E>(int capacity)` constructor, which defines the size of the initial array. If we pick a good number for the capacity based on our needs, we may never need to run this doubling array size operation. So I have already made some minor optimizations here.

## Explaining the solution

The great part about Object-Oriented Programming is that it allows us to think of systems as objects interacting with one another, which lends itself to useful visual diagrams. For example, if I were explaining my code in `PointListPath` to a colleague, I might draw a diagram like this:

![pathFalseDiagram.png](..%2Fimages%2Foptimization%2FpathFalseDiagram.png)

I visualize my `Path` containing an array that acts like a List, where each Point is stored at a particular index. I could use a whiteboard and explain how my `add`, `get`, and `distance` function works.

The problem is, _this isn't actually how Java works_, or how most implementations of OOP work. The below diagram is a bit closer to how Java works

![pathListBetterDiagram.png](..%2Fimages%2Foptimization%2FpathListBetterDiagram.png)

This diagram looks a bit more chaotic, with a lot more arrows. The key here is to illustrate that the `Point` objects **are not actually inside the `PointListPath`** object at all! For instance, if I had a `PointListPath` variable called `path`, it's important to note that `path` is a **reference** to a `PointListPath` object in memory. That is `path` tells me where to **look** for its contents, not the contents themselves. So, something that seems simple, like getting the `x` coordinate of the point at index `2` involves the following:

1) Go to the memory address `path` references and access the reference value of `points`
2) Go to the memory address `points` references and access the reference value of `contents` plus 8 bytes (at least, typically 8 bytes, since each reference is 4 bytes "wide")
3) Go to *that* memory address and access the `double` value of `x`

For step 2, it's important to note that Arrays are stored sequentially, and for step 3, primitive data types are stored **by value**, not by reference. But the key takeaway is that even if we have a reference to `path`, we still have to travel to three different memory addresses to get a single value.

## Before you panic and burn down your code

Let me repeat what I said before: "And this approach is fine! I would be happy to see a student write this code...". There is nothing *wrong* about this code. But what you should be aware of is that jumping around memory locations *takes time*! Remember, ***premature optimization is the root of all evil!***

## Where this hurts performance

This is why arrays are so good, because they store all contents sequentially in order! The fact that all the data is sequential makes iterating through it faster. Modern computers are optimized, using *caching*, to store sequential data directly in the CPU's cache, instead of having to go to memory, which takes longer. This is the main reason why iterating through an `ArrayList` is **substantially** faster than iterating through a `LinkedList`, even though **both** are O(n).

Every time we call a constructor using `new`, the JRE allocates a location in memory for our data. However, the JRE is never the only process on your computer allocating memory! This means even if we create several points in a row in our code...

```java
    Point a = new Point(3, 5);
    Point b = new Point(2, 7);
    Point c = new Point(-3, 17);
```

...there's no reason to believe that these objects are adjacent, or even near each other, in memory. This is a problem, because even if we create an array of `Point`, we are only allocating a sequential piece of memory to store **reference**, not the `Point` data! This means we always have to take at least one extra step in memory to access a value.

## CoordinateArray
