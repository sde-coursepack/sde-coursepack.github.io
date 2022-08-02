---
Title: Scaling up Software
---


If you're taking this class, you've likely been learning how to write code for a year or more. This means you are capable of writing simple scripts, and even smaller multi-class programs, especially in Java. This valuable skill is the bedrock on which to grow your skills.

#Large scale systems

However, this has not prepared you for writing *enterprise* systems. The word enterprise means "a project or undertaking, typically one that is difficult or requires effort." When we talk about *enterprise software*, we are talking about larger software systems. These systems are built and maintained by teams of programmers over a long period of time, and must respond to changing customer needs or environmental changes. 

**This problem isn't unique to software!** Engineers of all stripes have to deal with these problems!

## Ad hoc solutions

Imagine a scenario: You are hiking with several small children. You come across a small creek in your path. You need to get children over the creek without their feet getting wet. What do you do?

<img src="https://www.bassettcreekwmo.org/application/files/1514/5676/5130/PlymouthCreek-BassettCreekWatershed.jpg" width="40%" alt="A picture of Bassett Creek in Plymouth, Minnesota, via Bassett Creek Watershed">

Well, thinking quickly, you may see a log nearby, drag it over, and lay it across the creek, having kids walk over. After the kids are over, you may leave the log and walk away, never thinking of crossing that creek again.

This is an *ad hoc* solution: that is, a one-time solution to the problem. You had a short-term need, so you made a quick and easy solution to solve your immediate problem. You don't care about maintaining the solution, you don't care about other people being able to use the solution, you don't have a long term plan in place if the log rots, etc.

The thing is, most engineering problems aren't solved this way!

## Where ad hoc fails

Now imagine that instead of a small creek, you are crossing a large river. Instead of small children, you are transporting cars. And instead of crossing the river once, you need these cars to be able to go back and forth across this bridge for several decades. This means not just building the bridge, but establishing the process of *how* you are going to build the bridge, what is the budget of this bridge, how is this bridge going to be maintained, how are you going to monitor this bridge for faults, how will you ensure the bridge doesn't fail when a heavy truck drives over, etc. 

"Dragging over a big log" isn't a solution.

The key takeaway is that small scale ad hoc problem solving is fundamentally different from building larger, sustainable systems.

Large software isn't built by one programmer working alone for one afternoon, testing their code against a teacher provider auto-grader. In software, long-term projects are the norm. And even seemingly simple problems, can expose massive difficulties.

<iframe width="560" height="315" src="https://www.youtube.com/embed/-5wpm-gesOY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In this class, we will start preparing you for writing larger software systems, practice incremental development alongside other programmers.
