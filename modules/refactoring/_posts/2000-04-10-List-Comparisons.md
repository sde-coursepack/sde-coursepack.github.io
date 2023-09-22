---
Title: List Comparisons
---

* TOC
{:toc}


# ArrayList vs LinkedList

This module shows some short experiments to show various changes and how List selection effects performance. Nothing here will be really *new*, so you can proceed to the next module and consider this reading optional. However, I feel it's worth pointing out a few examples of additional changes and how they could affect performance.

Building from the example in the last module, lets consider the performance impact if we consider a `LinkedList` for `PointListPath`. I will note that, in general, `LinkedLists` are bad. To show off just *how* bad `LinkedList` can be, I wanted to illustrate by making one change to the constructor in `PointListPath`

```java
    public PointListPath(int initialCapacity) {
        points = new LinkedList<>();
    }
```

Let's compare the runtime of the old `PointListPath` with an `ArrayList` vs. the new one with a `LinkedList`. Note that by its nature, LinkedList cannot "pre-allocate" space. So yes, it's *terrible* style to not use the argument, but my goal was just to test this idea as quickly as possible. I never committed this change.

Looking **only** at the `distance` with `PointListPath`

| Input Size | ArrayList | LinkedList |
|:-----------|----------:|-----------:|
| 10         |       0ms |        0ms |
| 100        |       0ms |        0ms |
| 1000       |       0ms |        0ms |
| 10000      |       1ms |       59ms |
| 100000     |       2ms |     6061ms |
| 1000000    |       3ms |        N/A |
| 10000000   |      19ms |        N/A |
| 100000000  |     199ms |        N/A |

All the N/As are because, after running for 30 minutes on 1 million inputs, I stopped the process and never bothered trying larger sizes. Why is `LinkedList` so bad? Well, the main reason is that my code simply not implemented to iterate through a LinkedList correctly.

```java
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
```

The problem is the two calls to `get`. This is fine for an `ArrayList`, as `get` is a constant-time operation. But in a `LinkedList`, get is `linear`. That is, the number of steps you need to take is proportional to the `index`. The problem is that I'm doing this in a loop that executes once for each element of the list (well, technially size() - 1, but you get the idea). This means that for a `LinkedList` my code isn't `O(n)`, it's `O(n^2)`! This means using a `LinkedList` in this way is *actually wrong*, and will have a serious enough impact on performance that it simply cannot be ignored.

In general, if you have a LinkedList that you need to iterate through, you should always use an Iterator, so that getting each next node in the list is a constant time operation. However, adjusting my code to account for this makes it more complicated

```java
    public double totalDistance() {
        double totalDistance = 0.0;
        Iterator<Point> iterator = points.listIterator();
        Point currentPoint = iterator.next();
        while (iterator.hasNext()) {
            Point nextPoint = iterator.next();
            double distance = currentPoint.distanceTo(nextPoint);
            totalDistance += distance;
            currentPoint = nextPoint;
        }
        return totalDistance;
    }
```

I'm not saying this code is *too* complex, but it's one of those instances where an iterator can feel awkward to use, since I need both the `Point` I'm currently visiting and the next `Point`. I need to use `next()` both to get the nextPoint for a distance calculation *and* to update my `currentPoint` variable, which forces me to do some manual reference manipulation. It's not *bad*, but it makes the code more difficult to understand. *But*, if I insisted on using a LinkedList (I wouldn't as we'll see it a bit), this is more or less how I have to use it in order to not end up with O(n^2) running time.

For the note, the iterator approach *also* would work with ArrayList, but since using `get` is constant time with `ArrayList`, I would simply stick with my previous code. In theory, this  *could* give a very very very slightly worse performance, since we are introducing another object. In practice, `ArrayList` if there was a difference in performance, I couldn't demonstrate it. I got effectively the same runtime using either `get` or `iterator` with ArrayList, and didn't see a consistent difference even at 100 million input size.

So, let's see how this change helps, again only looking at `distance`

| Input Size | ArrayList | LinkedList w/ get(index) | LinkedList w/ iterator |
|:-----------|----------:|-------------------------:|-----------------------:|
| 10         |       0ms |                      0ms |                   1ms* |
| 100        |       0ms |                      0ms |                    0ms |
| 1000       |       0ms |                      0ms |                    0ms |
| 10000      |       1ms |                     59ms |                    0ms |
| 100000     |       2ms |                   6061ms |                    4ms |
| 1000000    |       3ms |                      N/A |                  202ms |
| 10000000   |      19ms |                      N/A |                 1717ms |
| 100000000  |     199ms |                      N/A |                    N/A |

&ast; - the 1ms at 10 input size is an aberration, and I am assuming either my computer hitched on some other process or I just happened to reach the "start time" point just before a millisecond ticked over. I only include to let you know that benchmarking can produce misleading results with small input sizes.

It's *still* way slower than an `ArrayList`, and I *still* was unable to run my 100 Million input benchmark. This is because, as I mentioned earlier, **references are slower than data**. Each node of the LinkedList *is not* a point. It's a `Node` object which has separate references to the `Point` and the next `Node`. This means instead of our old approach, where we had to travel 2 references (from `PointListPath` -> `ArrayList` -> `Point`), we now have to travel 3 references. `PointListPath` -> `Iterator` -> `Node` -> `Point`).

Even worse, while Java could leverage the cache with the ArrayList as an automatic optimization, it cannot do so with the LinkedList because the data is not sequential. In fact, a `Node` may reference a `Point` that is stored nowhere near it in memory, and the reference to the next `Node` may also be nowhere nearby. In short, you can't expect Java to do any cache optimization and it shows.

The reason `LinkedList` failed at 100 Million inputs is that a `LinkedList` also comes up with a significant memory overhead, because in addition to storing `Point`s, we now have to *also* store `Node`s, which contain two references. So, we are creating twice as many objects, they are non-sequential, and we can't leverage the cache.

## What are LinkedLists even for?

The key takeaway is that just because two operations are linear time, or `O(n)`, doesn't mean they are equally fast. It just means they grow at the same rate. Because of their construction, any `O(n)` operation on a `LinkedList` will be slower than an `ArrayList`.

In fact, the only time `LinkedLists` are even theoretically better than `ArrayLists` is adding/removing from the front of the structure (which we need to do when implementing a Queue). That's because adding at the front of a `LinkedList` is **constant time** , or O(1), while for an `ArrayList` it's **linear time**, or O(n), because you have to shift all the values right to make room.

So, I ran a benchmark to compare calling `add(0, value)` (add to the front) and `remove(0)` remove from the front on both a `LinkedList` and an `ArrayList` with different sized inputs. Here are those results:

| Size      | LL add |  AL add | LL rem |  AL rem | LL tot |   AL tot |
|:----------|-------:|--------:|-------:|--------:|-------:|---------:|
| 10        |    0ms |     0ms |    0ms |     0ms |    0ms |      0ms |
| 100       |    0ms |     0ms |    0ms |     0ms |    0ms |      0ms |
| 1000      |    0ms |     1ms |    0ms |     0ms |    0ms |      1ms |
| 10000     |    0ms |     4ms |    1ms |     4ms |    1ms |      8ms |
| 100000    |    2ms |   316ms |    1ms |   281ms |    3ms |    597ms |
| 1000000   |  132ms | 97609ms |   10ms | 55569ms |  142ms | 153178ms |

In the table above, "LL" means `LinkedList` and "AL" means `ArrayList`, "add" refers to the `.add(0, value)` operation, "rem" to the `.remove(0)` operation, and tot to the total time for both operations combined. So, for example at size = 100, for the both lists, I call use `add` 100 times, then `remove(100)` times. So I'm looking at both operations separately and combined. **To be clear**, this is *only* for adding/removing **at the front**.

Ignoring the 153-second elephant in the room for a second, it's worth noticing that `add`ing on a `LinkedList` is slower than `removing`, despite bother operations seemingly working with the same number of references. The reason for the difference is memory allocation. Allocating memory takes significant more time than dereferencing memory. Be aware, however, that this time does *not* account for Java's garbage collection to actually free the memory.

And, yeah, this is the one case where `ArrayList` is waaaaay worse, and it's because adding/removing `n` elements at the front of an `ArrayList` becomes an `O(n^2)` problem, whereas it's simply `O(n)` for the Linked List. However, you'll notice that I don't get a spike until 100 thousand, and while slower, it's still really quick at 10 thousand. I think the reason for the spike is two-fold:
1) It's quadratic, and quadratic grows more rapidly
2) At 10000 and below, while we still have to shift values, the ArrayList is small enough that we can do everything in the cache, and Java has a lot of optimizations built for this. Once we exceed the cache limit, those optimizations can no longer be used.

That said, this is the only time when `LinkedList` is better. Anytime you expect to need random access (i.e., using `get`) or adding anywhere but the front, `ArrayList` are going to be as good or better.

For example, when `add`-ing or `remove`-ing from the **end** of the List, `ArrayList`s are proportionally much better, though the concrete difference is small. The following graph shows adding then removing only from the **end** of the list.

| Size      | LL add | AL add | LL rem | AL rem | LL tot | AL tot |
|:----------|-------:|-------:|-------:|-------:|-------:|-------:|
| 10        |    0ms |    0ms |    0ms |    0ms |    0ms |    0ms |
| 100       |    0ms |    0ms |    0ms |    0ms |    0ms |    0ms |
| 1000      |    0ms |    0ms |    0ms |    0ms |    0ms |    0ms |
| 10000     |    0ms |    0ms |    0ms |    0ms |    0ms |    0ms |
| 100000    |    3ms |    1ms |    1ms |    1ms |    4ms |    2ms |
| 1000000   |  128ms |   14ms |    8ms |    5ms |  136ms |   19ms |

Here, we can see that both operations are very fast, but `ArrayList` is definitely faster. So in general, unless you **absolutely know** you want a `LinkedList`, default to an `ArrayList` for `List` implementations in Java. *But*, you should know when a `LinkedList` is better.

## Built-in optimizations

A common need in lists is sorting, which can be done with the `List.sort()` method to use natural order, or the `Collections.sort()` method to define your own sorting `Comparator`. In general, Java uses a variant of a merge sort for `List` objects. But with a merge sort, we definitely need random-access, which sounds like bad news for a `LinkedList`. That means `LinkedList` sort times must be horrible, right? Well, let's test:


| Size      | LL sort | AL sort |
|:----------|--------:|--------:|
| 10        |     0ms |     0ms |
| 100       |     0ms |     0ms |
| 1000      |     0ms |     0ms |
| 10000     |     4ms |     2ms |
| 100000    |    31ms |    19ms |
| 1000000   |   372ms |   261ms |

Wait, how is `LinkedList` not slow if sorting using random access?

Well, because you never actually sort a `LinkedList`. The way Java Collections sort is by first dumping the contents of the `List` into an array, and then sorting the array. After the array is sorted, it simply sets the value of each element into the List to the value that matches its index in the sorted array (using an iterator to ensure a linear-time copy). You can think of this as the `LinkedList` being turned into an array, sorted, then turned back into a `LinkedList`. This means the actual sorting step for both `LinkedList` and `ArrayList` is identical, so the only difference is the time to iterate through the `List` twice (once to copy to the array, once to copy the array to the List). Because iterating through a `LinkedList` is slower, that accounts for the time difference. But both are still O(n * log(n)).


