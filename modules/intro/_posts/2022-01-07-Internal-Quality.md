---
Title: Internal Software Quality
---

![](https://imgs.xkcd.com/comics/software_development.png)
Source: [XKCD #2021](https://xkcd.com/2021/) by Randall Munroe

# Internal Quality

With Internal software quality, we are concerned with the software
quality *from the perspective of the developers.* The primary
measure of Internal Quality is **Maintainability**. That is, the
internal quality of software is how **maintainable** it is from
the developers.

---

## Contents
{: .no_toc }

* TOC
{:toc}

---


## Software Maintenance

Maintenance in this context relates to anything from the evolution
of software over initial development, bug fixes, new features,
changing features, etc. We can break maintainability down into:

* __Analyzability__ - to what extent can the software construction be understood?
* __Changeability__ - the effort it requires to makes changes to the software
* __Stability__ - the extent to which changes in one part of a software construction affect other parts
* __Reusability__ - the extent to which parts of the construction can be reused in other parts of the construction or in other software projects.
* __Testability__ - the extent to which the software can be tested to find faults or defects in the software construction

## The key to internal quality

In this class we will spend a lot of time focusing on internal
software quality:
* We will improve the analyzability of our
software by writing clean code and using decomposition to
make each portion of our code more modular. 
* We will use version control software to ensure our software is changeable,
and follow good design principles to ensure our code is
receptive to change. 
* We will use the principles of modularity and design patterns to decouple unrelated portions of our system, increasing stability.
* We will use object-orientation to improve and refactoring
to improve the reusability of our code. 
* We will use JUnit and Test Driven Development to ensure our software is not only
testable, but thoroughly tested.

And the most important skill to achieving high internal
software quality is **design**. We need to design our
software in a way to make it maintainable. We cannot
add maintainability after the fact! We have to use good
practice to ensure our software *stays* maintainable.

## Software Entropy

*Entropy* is the idea that in our Universe, energy within
a closed-system cannot become more orderly overtime, and
not all processes are reversible. That is, things trend towards disorder. A consequence of this
is that the chemical energy in gasoline cannot be perfectly
turned into kinetic energy. We will lose
some energy to heat and sound energy, and it is impossible
to turn all that *back* into chemical energy without needing
even more energy. Ultimately, given enough time, even our
[observable universe itself will no longer produce stars](https://www.youtube.com/watch?v=F1CddzgVW14), as
all energy becomes so spread out as to never meaningfully
interact again.

In the same way, overtime, software naturally becomes harder
and harder to modify. The *maintainability* of the software
decreases. This is party because the software becomes larger and
more complex; the more code there is, the longer it takes
to understand. This is also partly because of bad design or
haphazard changes, leading to things like Spaghetti code, where
changing code in one system produces unexpected results in
seemingly unrelated systems.

Even relatively small applications can fall prey to software entropy.
Think back to homework you've done in the past. How many times
have you tried to fix a bug, thought it was a simple "one line fix",
only to realize changing that one line broke other parts of the program?

A common parody of the children's song "99 Bottles of Beer on the Wall"
may go something like:

> "99 unfixed bugs in the code  
> 99 unfixed bugs  
> You take one down, patch it around  
> 105 unfixed bugs in the code"

For some software, this song never ends: we never fix all the known defects (let alone unknown defects), because doing so would require more effort than it is worth. If a bug only occursin exceedingly rare cases, is hard to recreate, and fixing would
require a massive redesign of several highly connected sub-systems,
we often may not fix it at all.

But that's bad! If we're at this point, we've already had
a design and development failure! Ultimately, we can never create a perfect
design that will always ensure are software stays maintainable
forever. But a good design can dramatically slow down entropy,
whereas a bad design guarantees entropy is going to be nearly
instantaneous (for example: write a large application entirely
in a single main function or class).
