---
Title: Code Quality
---

# "Good Code"

What is "good code"? It can be a hard term to define. For example, code that is "good" in Java, with it's intended style and usage, may not be considered "good" in Python or C++ because of style and syntax differences between the languages.

In our last module, we talked about the Analyzability of our software. Software that is **understandable** is inherently easier to read and comprehend than software that is written poorly, even if in both cases the software works.  The idea of "code quality" extends this idea. We need understandable code, but it goes deeper. We need code that we can build from, change, maintain, update, and improve over time.

---

* TOC
{:toc}



---


<img src="https://imgs.xkcd.com/comics/code_quality.png">  

Source: [https://xkcd.com/1513/](https://xkcd.com/1513/)



## Software Entropy

The idea of software entropy is that as a project ages, it becomes harder and harder to maintain. This means doing things like:

* Bug fixing
* Adding new features
* Updating for new infrastructure

Becomes harder and harder the older the software is.

For example, have you ever found a bug, did what you thought was a simple fix, only to see that 3 other features (potentially features that seem completely unrelated) in your program broke because of that bug fix? If so, you've fallen prey to software entropy. 

![img.png](../images/clean_code/productivity_decline.png)

It has been observed empirically that as software ages, the time it takes to implement changes increases dramatically. However, the rate at which this speed increases isn't a sentence, but a decision. **We can slow down entropy!**

## Technical Debt

Last year, I installed a ceiling fan in the living room of my house. It took a lot of time and effort, and when I was done, I stopped working, sat down, and watched some Netflix. The next day, I wanted to install a fan in the Master bedroom. Except...I couldn't find my screwdriver nor my electric wand (for detecting if a wire has current in it and therefore not save to touch). They weren't in my toolbox!

The problem was that *I never cleaned up*! I didn't pull my tools away in my toolbox. I just left them on the couch in the living room near where I was working. Sometime during the night, my cat jumped up on the coach, pushing the screwdriver onto the floor under the couch, and the electric wand into the couch cushions. I spent nearly an hour just looking for my tools before I could start the next day. **All of that time could have been saved if I just spent 1 minute cleaning-up when I finished the previous task**.

This mindset of "cleaning up" applies to code! 

You aren't done when your code works! You are done when you've cleaned up!

By not putting my tools away, I accrued a "debt". That is, I didn't spend time putting my tools away now, so I paid for it later (with substantial interest) when I had to look for my tools. Our source code is just as vulnerable to debt. When you leave a method in an overly complex, unreadable state, or you use static global variables to store information relevant only to one function, you are accruing a debt that will cost you time later to fix (or to work around if you can't fix it).

### Technical debt accrues interest

Credit card interest rates can be outrageously high! Waiting just one month to pay off a $5000 with a 20% annual interest rate (close to the national average for credit cards) costs $83.33 in *interest alone*. If you continue to not pay it off, the next month's interest costs even more, $84.72. If you continue to not pay, in one year's time you will owe an additional $1000 dollars. In two years time? Nearly an additional $2500 dollars!

The best way to pay down debt is "as fast as you can". For example, say I completely stop using the $5000 credit card, and pay $250 dollars a month to pay it down. If I do this, I will pay $1,133 dollars in interest, or $6,133 dollars total on $5000 of debt! And it will take me over two years to pay it off!

However, if instead, in the very first month, I sell my retro video game collection and make a 1-time $1000 payment, and then continue paying $250 each month, I nearly **half** the total interest paid, to just $691. And I'll pay it off faster, in just over a year and a half.

That one time lump payment drastically reduced the total cost of fixing my debt! **Technical debt is no different!** As our software builds up technical debt, it gets harder and harder to edit! And eventually, it gets so hard to make changes to your code base, that you think the best idea is...

### Starting over from scratch

In his book "Clean Code" Bob Martin refers to this as "The Big Redesign in the Sky". We'll just start from scratch. After all, now that we know that our existing design is bad, we can avoid making the same mistakes, right?

No! If we don't commit to improving and maintaining the quality, we will simply fall into the same trap again and again and again! If you are sitting on a project that is hard to maintain or change, then you should consider when the best time to start refactoring and cleaning your code is:

* The best time to start improving your code is immediately after you wrote it
* The second-best time is now!
* The absolutely worst possible time is "later".

Never say "I'll do it later," because you almost certainly won't!

> "Later equals never"   
>  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-LeBlanc's Law

And even if you *do* do it later, the code won't be fresh in your mind! You'll have to read your bad, dirty, ugly code to understand it, and that will take time! Do it while it's fresh in your mind!.

No design we can make will perfectly meet all possible future changes to our software. Refactoring is a continuous battle against entropy. But everytime we pay off our technical debt immediately, that's effort we are spending on paying it back later with interest!

## You always read more than you write

The basic reason that clean code is important is that when you are writing code, very little time is actually spent *writing code*. Most of the time you are reading. Consider, for example, writing a program that reads an Excel file.

* How much time is spent looking up examples
* How often do you need to look at other classes
* What order are the Constructor arguments in?
* How do I use that data structure?
* Where does this variable come from again?

On and on and on. Most estimates say that while writing code, we spend **ten times** as much time reading code as we do *writing code*. This means that it is vitally important that we make our code as readable as possible, and easy to use as possible!

The thing that will slow down your projects is entropy and complexity. As your projects age, they naturally get larger and more complicated. If we spend time cleaning up our code and making it easy to read, this entropy slowdown will...well...slow down. Our project will be easier to develop longer, provided that each step along the way we take time to clean-up, refactoring, improve our variable and function names, etc.

## "But this will slow me down..."

> "There is no trade-off of quality vs. speed in software. There never has been. Low quality means low speed. Always.  
>   
> **The only way to go fast is to go well.**"   
> 
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Bob Martin, author of __Clean Code__

In automobile racing, how you go around corners can be arguably the most important skill on the track. However, counterintuitively, you don't want to greedily go as fast as you can at all times. If you approach a corner too fast, you have to slow down **longer** while going through the corner, which means you start accelerating later. As a result, you will end up going slower on the next straightaway because you tried to go too fast through the corner. On the other hand, if you approach the corner a little slower, than you can start accelerating out of the corner sooner, meaning you will go faster on the next straightaway.

In short, slowing down a little more now can save you significant time in the very near future! And the same applies to code!

Because your code must evolve and change overtime, taking time to clean it up is paying off technical debt before the interest has time to accrue! Do a little clean-up, ensure your code is readable and understandable, remove duplicate code, etc. Doing all this will mean that the next time you add to your code, you will save time! And don't think you have to write thousands of lines before this pays off. Writing clean code pays off **the very next time you have to re-read your code**, and you *will* have to re-read your code! Many, many times!

Ultimately, practicing the intentional effort to write readable, understandable code will help you go faster overall. "Fast and dirty" code may make the first couple of classes and functions faster to write, but before too long, you will start to reap what you sow, and your productivity will grind to a halt.

## Taking pride in your work

Ultimately, the greatest reason to ensure you are writing understandable, modifiable code is because **you should want to**! Programming is an incredibly powerful tool! Consider just how much of our lives are controlled by software, or rely heavily on software. And we have the power to touch and bend software with our programming ability! Think of all the amazing things we can, no, **we will** do in our lives with programming: the problems we can solve, the suffering we can abate!

Programming gives us this power, so we should aspire to be the best developers we can be! Because by being the best developers we can be, we solve the most problems we can! This very idea should give you a sense of pride!

While it may be filled with typos and behind schedule, I am *proud* of this textbook. I am *proud* of the assignments I write for this class, and the lectures I have given. Because I worked really well at them. Not just worked hard, but worked well. I spent months planning this course, writing example code, writing lesson plans, writing assignments.

In the exact same way I feel proud of what I've done, I want you to feel proud for your work! You're a programmer! Be the best programmer you can be! Do it right! Write code so clean that the next person who sees your code says "Wow, the person who wrote this is a professional who really cares about their craft!" Don't think writing "great code" is above you, it isn't! It just takes practice and persistence!

## Conclusion

In the rest of this unit, we will look at examples of "dirty code" and "clean code", as well as talk about techniques for turning the former into the latter. We will also look at techniques around using exceptions, as well as functional programming and streams, to help make our code simpler and shorter without sacrificing understandability.

## About "Clean Code"

Some of the material in this unit is derived from:

* [Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/B08X8ZXT15/) by Robert C. Martin. I will note that this book is criticized by many. My general thoughts on "Clean Code" is that it's a good entry point for someone looking for ideas to improve their code quality. However, *do not* take everything in Clean Code as gospel, or as a rules to always apply in every situation. In fact, I would specifically address some of the specific code examples in the book as, frankly, bad. But many of them are also good. In short, never take one book/blog/youtuber as the ultimate authority, but try to expose yourself to as many views as you can and adopt what gives you the best results.

Here is a [particularly well known criticism](https://qntm.org/clean) of Clean Code. Personally, I think it raises good points, but I also think it's pretty hyperbolic at times, and some of the complaints to me are things I just disagree with. I personally like [this article that highlights good lessons from the book](https://dev.to/thawkin3/in-defense-of-clean-code-100-pieces-of-timeless-advice-from-uncle-bob-5flk), even if many of the examples break good style, understandability, etc. 


