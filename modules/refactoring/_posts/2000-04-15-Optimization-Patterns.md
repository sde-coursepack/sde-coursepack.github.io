---
Title: Optimization Patterns
---

# Optimization Patterns
In this module, I will present some simple optimizations you can do with your Java code that could help improve performance. Keep in mind, however, that "premature optimization is the root of all evil." Additionally, many optimizations that work in *theory* only work in practice with very large inputs. For example, `StringBuffer` comes with a large enough overhead that it only gives improvements when `append` is used a large number of times.


* TOC
{:toc}

## My approach to optimization

In general, my approach to optimization is this:
1) Double-check my damn data structures: Am I using the right ones?  
2) Run a profiler and look for outliers, and focus on those outliers. If you get a 90% speed increase on a function that only uses 1% of the total processor time, you have only made a 0.9% improvement to your overall speed. If you get a 10% speed increase on a function that uses 50% of the processor time, you get a 5% increase to overall speed. Don't waste your time on code that only executes at start-up unless start-up is itself the problem.  
3) Check your algorithms: is there an algorithm or established technique that could improve your speed? Has someone else implemented a more optimized solution in a library you can use?  
4) Before attempting optimization, always start working in a new git branch. That way, when/if you FUBAR your code, you can just throw it away. This also ensures you have a "previous" version to benchmark against.  

## Disclaimer

A reminder: I'm *not* a professional software engineer. I'm a computer science professor who only has limited intern experience, and even that was a long time ago. Yes, I write code a lot. But not as much as a professional developer who doesn't have to answer emails from 500 students a week or give 6 hours of training meetings (aka lectures) a week. This means that there are things that I suggest that could be wrong for *any number of reasons*. If a domain expert tells you "hey, when solving these kinds of problems, use this algorithm", then they are almost certainly right, and I am almost certainly wrong.

Optimization is one of those things where "general rules" can break down. Things that should work in theory won't work in practice, or even make things worse. An optimization could work, until changes in how the module is interacted with make the optimization worse than the original solution. And optimizations may often be hardware dependent. For example, in video games, optimizations that can be used for the Playstation 5 may not be possible, or not be optimal, on X-Box Series X. Optimizations that work for NVidia graphics cards, like using DLSS, are not possible on AMD graphics cards. Etc. 

Ultimately, remember that your goal as an author of software is to meet the customer/users need.  If the user is happy with performance, optimization should be a lower priority, even *if* there is "low-hanging fruit".

## DRY - Don't Recalculate Yourself

Consider our interface `Path` from the Optimization Example module:

```java
public interface Path {
    void add(double xCoordinate, double yCoordinate);
    void add(Point point);
    Point get(int index);
    int size();
    double totalDistance();
}
```

Let's say we had a list `Path` objects, and we wanted to find the shortest path. We might do something like:

```java
public Path getShortestPath(List<Path> paths){
    Path shortestPath = paths.get(0);
    for (Path path:paths) {
        if (path.totalDistance() < shortestPath.totalDistance()) {
            shortestPath = path;
        }
    }
    return path;
}
```

The above code makes sense for our purpose, but pay special attention to the if-statement: `if (path.totalDistance() < shortestPath.totalDistance())`

In this statement, we calculate both the distance of our iteration variable `path`, but we also calculate the `shortestPath`'s distance as well. The problem with this is this means we are always **re-calculating** the distance of `shortestpath` over every iteration of the loop, when in every single case but the first loop, we would have already calculated it!

Keep in mind that the `totalDistance` function is using a square root operation, which is computationally expensive. As such, we want to limit how much we call `totalDistance`, since each call is `O(n)` where `n` is the number of points in the Path, and for each `n` we have to call the square root function (meaning the `C` value is quite large).

Instead, we can **trade** `calculation-time` for `memory`, by storing the `shortestDistance` as a variable. That way, we ensure we never recalculate the distance of the path multiple times over the course of this function.

```java
public Path getShortestPath(List<Path> paths){
    Path shortestPath=paths.get(0);
    double shortestDistance = shortestPath.totalDistance();
    for (Path path:paths) {
        var pathDistance = path.totalDistance();
        if (pathDistance < shortestDistance) {
            shortestPath = path;
            shortestDistance = pathDistance;
        }
    }
    return path;
}
```

Here, in the above, we can see that by using variables to store the path distances, we only calculate the `totalDistance` of each path **once**. This means, compared to our earlier implementation, we are calculating the `totalDistance` of path only **half** as often as before.

Be aware, both implementations are `O(m * n)`, where `m` is the number of Paths, and `n` is the average size of each path. However, our second solution avoids recalculation, and as such the `C` becomes much smaller.

### Nested Arrays

One common problem with nested-arrays is that they get used in a way that requires recalculation. For example, consider the following code:

```java
public class Matrix {
    private int[][] matrix;
    private int width, height;
    
    public Matrix(int numberOfRows, int numberOfColumns) {
        this.width = numberOfColumns;
        this.height = numberOfRows;
        this.matrix = new int[height][width];
    }

    public int get(int row, int column) {
        return matrix[row][column];
    }
    
    public void set(int row, int column, int value) {
        matrix[row][column] = value;
    }
    
    public void add(Matrix other) {
        for (int row = 0; row < height; row++) {
            for (int column = 0; column < width; column++) {
                this.matrix[row][column] += other[row][column];
            }
        }
    }
}
```

The above seems reasonable, but there is an inefficiency in the `add` method, and it has to do with how "2-d arrays" work (and why they aren't actually 2-d arrays).

Understand that an int[][] is **not** a 2-dimensional array, because such things don't exist in Java. Instead, this is an **array of int[]**, or an array of integer arrays. And integer arrays are referenced objects.

This means that when you say `other[row][column]`, you are following 3 different references:

1) From the function scope to the `other` object's memory location  
2) From the `other` location to it's `matrix` array of `int[]`
3) From the `other.matrix[row]` reference to the underlying array

Each of these memory references are passed through every single time you use `other[row][matrix]`. However, by using variables, we can limit our travel each time:

```java
    public void add(Matrix other) {
        for (int row = 0; row < height; row++) {
            thisRow = this.matrix[row];
            otherRow = other.matrix[row];
            for (int column = 0; column < width; column++) {
                thisRow[column] += otherRow[column];
            }
        }
    }
```

This mini optimization helps reduce the number of references we need to pass through in the critical section substantially, and could provide noticeable improvement on large arrays, especially if the `add` operation itself is used frequently. We aren't "re calculating" the memory reference over and over again.

## Lazy Evaluation

To explain what Lazy Evaluation means, lets look at an example of "eager" evaluation:

```java
public class CoordinateArrayPath implements Path {
    private final double[] pointArray;
    private final int length;
    private int currentSize;
    private double distance;

    @Override
    public void add(double xCoordinate, double yCoordinate) {
        if (currentSize == length) {
            throw new IllegalStateException("CoordinateArray is full and cannot add more points");
        }
        pointArray[currentSize * 2] = xCoordinate;
        pointArray[currentSize * 2 + 1] = yCoordinate;
        currentSize++;
        distance = calculateDistance();
    }

    private double calculateDistance() {
        double totalDistance = 0.0;
        for (int i = 0; i < currentSize - 1; i++) {
            double diffX = pointArray[2 * i] - pointArray[2 * i + 2];
            double diffY = pointArray[2 * i + 1] - pointArray[2 * i + 3];
            totalDistance += Math.sqrt(diffX * diffX + diffY * diffY);
        }
        return totalDistance;
    }
    
    @Override
    public double totalDistance() {
        return distance;
    }    
}
```

Here, every time we add a need point to our path, we re-calculate the total distance of the path (see the last line of `add`). This may seem like a good idea, since now whenever we call `totalDistance`, well we've already calculated the value, so we don't have to calculate distance again!

Except, consider the following sequence of calls:

```java
public class Demo {
    public static void main(String[] args) {
        Path path = new CoordinateArrayPath(20);
        
        path.add(0,0);
        path.add(1,1);
        path.add(2,2);
        path.add(0,3);
        path.add(3,0);
        
        System.out.println(path.totalDistance());
    }
}
```

In this case, what we are doing with eager evaluation is bad. That is, we end up calculating the full-distance of the path 5 times, once after each coordinate is added, but we only ever *use* the last distance (the distance after the coordinate [3,0] is added.) This means all of our calculations beforehand are simply wasting time, since we never use those partial distance calculations.

The idea of lazy evaluation is that we only evaluate something *when we need to*. In general, it is a good idea to avoid eager evaluation, as it is at best as good as lazy evaluation, and at worst can be far worst.

While it may seem eager evaluation prevents us from calculating the `totalDistance()` multiple times, there are better ways to handle that (see Memoization).

### Lazy Initialization

Alongside lazy evaluation is lazy initialization. In lazy evaluation, we avoid invoking costly functions until we need to, and in lazy initialization, we avoid create new objects until we need to.

For example, instead of:

```java
public class LazyDemo {
    private Thing thing;
    
    public LazyDemo() {
        thing = new Thing();
    }
    
    public Thing getThing() {
        return this.thing;
    }
}
```

instead we do the following:

```java
public class LazyDemo {
    private Thing thing;
    
    public LazyDemo() {
        thing = null;
    }
    
    public Thing getThing() {
        if (thing == null) {
            thing = new Thing(); // initialize the object only when we need to
        }
        return this.thing;
    }
}
```

This is beneficial if we frequently will create or use a class, but not need or interact with its aggregations (that is, the objects it has fields for). A big reason for this in Java is that:

1) Allocating new memory for objects is an expensive operation, even if the object is itself simple or never used  
2) Once allocated, that memory cannot be freed until the outer object is freed from memory

Of course, a trade-off here is that the first call to `getThing` may be slower. Additionally, are class now becomes harder to understand, as we have to consider two states, one where getThing exists, and one where it doesn't.

As such, if you know you are likely to always use the class fields of an object, Lazy Initialization is likely to cause more harm than good. However, if you, say, load a large number of objects from a database, which are associated with other objects via a List, you may hold off on initializing and populating that List until you know you need to.

## Memoization

Let's consider the following partial implementation of Path from the Optimization Example. This implementation uses lazy-evaluation for the distance:

```java
public class CoordinateArrayPath implements Path {
    private final double[] pointArray;
    private final int length;
    private int currentSize;

    @Override
    public void add(double xCoordinate, double yCoordinate) {
        if (currentSize == length) {
            throw new IllegalStateException("CoordinateArray is full and cannot add more points");
        }
        pointArray[currentSize * 2] = xCoordinate;
        pointArray[currentSize * 2 + 1] = yCoordinate;
        currentSize++;
    }

    @Override
    public double totalDistance() {
        double totalDistance = 0.0;
        for (int i = 0; i < currentSize - 1; i++) {
            double diffX = pointArray[2 * i] - pointArray[2 * i + 2];
            double diffY = pointArray[2 * i + 1] - pointArray[2 * i + 3];
            totalDistance += Math.sqrt(diffX * diffX + diffY * diffY);
        }
        return totalDistance;
    }
}
```

Now, imagine that we build up a large list of `Path` objects at program start-up of our program (such as by reading coordinates in from several files), and we know that after this start-up phase, that the coordinates in a given `Path` will be unlikely to change.

In this case, we can have our object **remember** its distance after it's calculated the first time. For example, we can modify our class to add an `Optional<Double> distanceMemo` field that, when we calculate distance, stores that distance. In this way, we'd update our fields and our `totalDistance` method as follows:

```java
public class CoordinateArrayPath implements Path {
    private final double[] pointArray;
    private final int length;
    private int currentSize;
    private Optional<Double> distanceMemo = Optional.empty();

    @Override
    public double totalDistance() {
        if (distanceMemo.isPresent()) {
            return distanceMemo.get();
        }
        double totalDistance = 0.0;
        for (int i = 0; i < currentSize - 1; i++) {
            double diffX = pointArray[2 * i] - pointArray[2 * i + 2];
            double diffY = pointArray[2 * i + 1] - pointArray[2 * i + 3];
            totalDistance += Math.sqrt(diffX * diffX + diffY * diffY);
        }
        distanceMemo = Optional.of(totalDistance);
        return totalDistance;
    }
}
```

This means if, after our `Path` is built, we end up calling `totalDistance()` several dozen times, we only actually calculate the `totalDistance` once.

This idea of storing this response to make future requests faster is called **memoization**, or **caching** (although I prefer the term memoization, to distinguish this from CPU Cache optimization, which was discussed our Optimization Example unit). The idea is that we trade *memory space* now for *improved time performance* later.

### Memo invalidation

Be aware, the memo we just created only works **so long as the totalDistance doesn't need to change.** The problem is that we can `add` new points to our Path, which will of course change the distance.

This means we need to know when to **invalidate** our stored distance value. In our case, anytime that we call `add()` on our `Path`, the total distance is very likely to change. So, we update our `add` method to *reset* our `distanceMemo`:

```java
    @Override
    public void add(double xCoordinate, double yCoordinate) {
        if (currentSize == length) {
            throw new IllegalStateException("CoordinateArray is full and cannot add more points");
        }
        pointArray[currentSize * 2] = xCoordinate;
        pointArray[currentSize * 2 + 1] = yCoordinate;
        currentSize++;
        distanceMemo = Optional.empty; //invalidate previous distance memo
    }
```

Now consider how this could affect future development. What if we add methods like `remove(int index)` or `insert(int index, double x, double y)`? Those methods now, in addition to peforming their primary role, must now also include the `distanceMemo = Optional.empty;` line. In other words, in addition to storing an extra variable, we need to also consider when to **invalidate** that stored value. This makes developing this class more difficult.

### Memoization with arguments

In the first example, we used a `zero-argument` function as an example for `memoization`. But what if our function *does* have arguments to consider?

Consider the following hypothetical code snippet which defines a class who sums every number in a file:

```java
public class FileSummer {
    public long getFileSum(String filename) {
        long fileSum = 0L;
        //opens file, gets every integer, returns the sum, 
        //stores in the variable fileSum
        return fileSum;
    }
}
```

Here, we can't use an `Optional` like we did in our `Path` example, because different `filename` inputs will necessarily produce different outputs. Instead, we can use a `Map`, which I named `fileSumMemo` in the snippet below. 

```java
public class FileSummer {
    private Map<String, Long> fileSumMemo;
    
    public long getFileSum(String filename) {
        if (fileSumMemo.containsKey(filename)) {
            return fileSumMemo.get(filename);
        }
        long fileSum = 0L;
        //opens file, gets every integer, returns the sum, 
        //stores in the variable fileSum
        fileSumMemo.put(filename, fileSum);
        return fileSum;
    }
}
```

Whenever the function runs, it will check "have we already gotten the sum of this file?" by checking that Map for the key `filename`. If we already have the sum for that `filename`, then we simply return the previously calculated value. Otherwise, we calculate as normal, but store the `fileSum` value in our memo before returning.

### Added complication

Note however, that the above **only** works if we assume that the files we are opening **do not change** during the "lifetime" of this object (that is, from when it is created until when it is de-referenced). If it's possible for the file contents to change *during* the lifetime of our `FileSummer`, this is no longer an appropriate approach. This is because if the contents file changes, which could happen completely outside of our program being run, our memo could be wrong.

For example, say when we first calculate our `fileSum` for `sandwich_order.text`, the file looks like this:

```text
I would like 3 grilled cheese, 2 ham and swiss, and 1 peanut butter and jelly.
```

In this case, the result from our `FileSummer` would be 6 (adding the 3, 2, and 1). However, someone, in another program, might edit the file to say:

```text
I would like 3 grilled cheese, 2 ham and swiss, and 2 peanut butter and jellies. (Edit, sorry, I forgot to order my sandwich)
```

Now, because we have 2 Peanut Butter and Jellies to order instead of 1, our fileSum should be 7, not 6. But our memo will still have us return 6.

### Always think about invalidation

> There are only two hard things in Computer Science: cache invalidation and naming things.
>
>— Phil Karlton

While I don't fully agree with the above quote (there are many hard things in computer science), the point here is that "cache invalidation", or "memo invalidation" is a hard problem. In general, if your memo is tied to a something that is **unstable** (that is, very likely to change moment to moment, or changes are unpredictable and hard to monitor), it's often better to avoid using memoization altogether, as the added complexity of maintaining/updating/invalidating the memo can add extra code complexity that is likely to create bugs, incorrect behavior, or even add its own inefficiencies.


### Thread Safety

Be aware none of my above examples are Thread-Safe, so if this class is accessed from multiple threads, you could get incorrect or inefficient behavior. Thread-safety is something to consider here in a parallel system, though that is beyond the scope of this course.

## Conditional Optimizations

Consider the following function using our `Path` interface from before:

```java
public void addToPathMap(Path path, Map<Path, Double> pathMap) {
    if (path != null) {
        var totalDistance = path.totalDistance();
        if (pathMap != null) {
            pathMap.put(path, totalDistance);
        } else {
            throw new RuntimeException("Error: Null PathMap");
        }
    } else {
        throw new RuntimeException("Error: Null Path");
    }
}
```

In this function, there is an obvious optimization we can make: we can check if `pathMap` is null **earlier**. In our current code, if `path` is not null, we always calculate `totalDistance` (which can be an expensive operation). However, if `pathMap` is null, we never actually **use** the `totalDistance`. We completely wasted our time calculating the `totalDistance` of our `path`!

Instead, always check function "easy" function pre-condition operations first, *before* performing more expensive operations:

```java
public void addToPathMap(Path path, Map<Path, Double> pathMap) {
    if (path == null) {
        throw new RuntimeException("Error: Null Path");
    }
    if (pathMap == null) {
        throw new RuntimeException("Error: Null Path");
    }
    var totalDistance = path.totalDistance();
    pathMap.put(path, totalDistance);
}
```

Now, in addition to having more readable code, we never "waste effort" by calculating the `totalDistance` when we're going to throw an exception anyways.

### Conditional Order

Consider the following function for checking if a reservation for a table at a restaurant is valid:

```java
public boolean isReservationValid(Reservation reservation, Time currentTime) {
    return reservationService.reservationExists(reservation) &&
        reservation.isTimeInWindow(currentTime);
}
```

Here, we necesarily need to check two conditions:

1) Does the reservation exist in our reservation system: we check this with `reservationService.reservationExists(reservation)`  
2) Is the current time in the reservation Window - that is, the time frame in which the reservation can still be used. For example, maybe if the reservation is for 6:00 p.m., you have to be there sometime between 5:50 p.m. and 6:15 p.m. to claim the reservation. we check this with `inReservationWindow(reservation.getWindow(), currentTime)`

But consider, what if we know ahead of time that `reservationService.reservationExists` is going to require communicating over the internet to a remote database, and that process can take up to 1 second to establish the connection, send the query to the database, and retrieve the results. Additionally, we know that `reservation.isTimeInWindow(currentTime)` is a function that requires no internet access, and ultimately just checks the current time against two numbers (the window start and window end).

In this case, we can take advantage of **short-circuiting** of boolean expressions by switching the order of the operations.

```java
public boolean isReservationValid(Reservation reservation, Time currentTime) {
    return reservation.isTimeInWindow(currentTime) &&
        reservationService.reservationExists(reservation);
}
```

Based on our prior knowledge, we can assume that `reservation.isTimeInWindow` will be **much** faster to calculate than `reservationService.reservationExists`, since the `reservationExists`. Because of this, we can do the *faster* operation **first**, that way if it is false, we never need to do the more time-consuming, resource-expensive remote database connection.

In this case, it's important to understand the difference in Java between `&&` and `&` (and similarly the difference between `||` and `|`).

* `&&` is a **short-circuited** and operation 
* `&` is *not* short-circuited

To understand this difference, consider `a() && b()` vs. `a() & b()`. In the short-circuited version, if `a()` is false, **we never run b()**. This is because in an `and` conjunction, if any one value is false, the entire thing is false. Therefore, there is no reason to even evaluate `b()` if we already know `a()` is false. In general, you should avoid using single `&` unless you have a very, very good reason you need to run both `a()` and `b()` (and in nearly all cases, you don't!). By extension, the same idea applies to using `||`, where in `a() || b()`, if `a()` is true, we never need to evaluate `b()`.

As such, when evaluating two or more boolean expressions where we know ahead of time some expressions will be significantly more expensive than others, it's a good idea to sort them from `fastest` to `slowest` in order to take advantage of short-circuiting.

In general `fastEvaluation && slowEvaluation` is better than `slowEvaluation && fastEvaluation`.

### Likelihood Conditional Order

Let's say now that we have two expensive operations in a conditional:

`if d() || e()`

If we know that `d()` is significantly more likely to be false than `e()` (over the lifecycle of this code being invoked), and both operations are roughly the same cost, then it would make sense to check `e()` first to take advantage of short-circuiting:

`if LikelyTrue() || LikelyFalse()` is better than `if LikelyFalse() || LikelyTrue()` when both are roughly the same "cost"

`if LikelyFalse() && LikelyTrue()` is better than `if LikelyTrue() && LikelyFalse()` when both are roughly the same "cost"










