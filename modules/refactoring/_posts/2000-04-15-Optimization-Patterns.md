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

Ultimately, remember that your goal as an author of software is to meet the customer/users need.  If the user is happy with performance, optimization should be a very low priority, even *if* there is low-hanging fruit.

## DRY - Don't Recalculate Yourself


## Conditional Optimizations


## P-Streams for big data


## Lazy Initialization


## Lazy Evaluation


## Memoization
