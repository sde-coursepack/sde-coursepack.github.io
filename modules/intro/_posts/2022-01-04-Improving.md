---
Title: Learning to Learn
---

# Don't panic

> “In many of the more relaxed civilizations on the Outer Eastern Rim of the Galaxy, the Hitch-Hiker's Guide has already supplanted the great Encyclopaedia Galactica as the standard repository of all knowledge and wisdom, for though it has many omissions and contains much that is apocryphal, or at least wildly inaccurate, it scores over the older, more pedestrian work in two important respects. First, it is slightly cheaper; and secondly it has the words DON'T PANIC inscribed in large friendly letters on its cover.” - __Hitchhiker's Guide to that Galaxy__, Douglas Adams

At this point, three pages in, you may be worried this is a class about software theory, reading articles from the 1980s. Nothing can be further from the truth! **This class is all about practice!** Nearly everything we do in this class will involve planning, writing, analyzing, and fixing code. The vast majorite of modules will involve writing code!

So far, you've been learning how to write code. Good! This is a vital skill set that will serve you the rest of your life! But now understand two unfortunate realities: 

1) You don't know everything  
2) You don't know most of what you don't know

The same is true of everyone! Accepting this is the first step in the act of learning. More importantly, you'll never be done learning during your career: there will always be room for improvement, new tools, etc.

---

## In this course 

In this course, you will be exposed to several tools and techniques for software development:  
1) Version control systems to help ensure everyone is able to work together, share code, and track changes/improvements/fixes.  
2) Testing practices to efficiently detect and correct bugs, and ensure they stay corrected 
3) Design principles and patterns that allows the system to evolve over time with less effort and more stability.  
4) Communicating design decisions to your team and others  
5) Writing and refactoring code in such a way that it's easier to understand, re-use, and test  
6) Building GUIs to broaden the usability of our software  
7) Connecting to a database to ensure our software system data can persist even when it's not running.  

Learning these tools and techniques is a valuable undertaking for its own purposes - having these skill sets already under your belt will make you a more attractive candidate for an employer. But in learning, we seek not just to use these tools, but use them creatively and wisely, knowing their limits, and planning how to look for and prevent failures.

---

## "Do I really need to..."

There will be times during this semester you say:

* "Do I really need to do test driven development? I can just test at the end..."  
* "Do I really have to use all these classes? I can write it all in Main.java..."  
* "Do I really need to commit after every function and every test? It's faster if I just commit at the end..."  
* "Do I really need to learn this design pattern? I can just figure out a solution on my own..."  

Etc. Etc. Etc. This betrays a form of fundamental conceit we humans have. To quote Fred Brooks:

"All programmers are optimists. Perhaps this modern sorcery especially attracts those who believe in happy endings and fairy godmothers. Perhaps the hundreds of nitty frustrations drive away all but those who habitually focus on the end goal. Perhaps it is merely that computers are young, programmers are younger, and the young are always optimists. But however the selection process works, the result is indisputable: 'This time it will surely run,' or 'I just found the last bug.'" - Fred Brooks, The Mythical Man-Month

Even new programmers can see the wisdom in these words. How many times during your intro programming class did you think "there, I fixed that bug, now my code will work" only for your code to immediately crash when you hit run for a completely different reason? Or, hell, even the same reason because you didn't actually fix the problem? In such situations, it's easily to fall into a pattern of:

* Pick some code to change
* Change it in some way
* Pray
* Hit run
* Be disappointed
* Repeat

But this is also **the least efficient possible way to develop software**! Instead, we need to take time to plan our actions! But this isn't natural, what's natural is to just keep trying. Learning how to plan, design, test, and write code effectively **will make you more efficient**! As the tongue-in-cheek engineering expressions goes:

> "Hours of work can save you minutes of planning"

The fact is that learning these skills will make you better and more efficient as a programmer...eventually. But not right away.

---

## Learning is hard

All the "do I really need to..." questions above remind me of...well...me. I didn't learn to ride a bike until I was 13 years old. For those who need to get it out of your system, now's a good time to point and laugh. It wasn't that I didn't have a bike; I did! It's not that I didn't have anywhere to ride it; I did! It's not that I had no teacher; I did! I had parents who tried to teach me *and* two older siblings who could ride a bike by age 6 who *wanted* me to ride a bike with them! 

I had support! I had the tools I needed! I had all the time I wanted to practice! I had a bike with training wheels to practice. I had everything in my favor pushing me towards riding a bike, and yet I never learned.

Why? **Because I built bad habits and I refused to change them**. I naturally leaned to the right while riding a bike, on the right side training wheel. And I was okay with that! The extra wheel gave me stability, I didn't have to worry about balance. *Why would I change when my what I was doing worked with the bike I was riding?*

<img src="https://images.unsplash.com/photo-1595182747080-3b43712dd27d?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=1170&q=80" width="40%" alt="Two children riding bikes with training wheels">

And then, when it came time to ride a bike for real at the age of 6. My dad took off my training wheels. When I went to ride my bike, I was surprised the training wheels weren't there, but tried riding anyways. I kept falling over on my right side. So I tried to correct it, and I fell over on my left side. So I tired to correct it, and fell over on my right side.

After falling down enough times, bruises and abrasions were just part of the damage. My pride also took a pretty big beat down (yes, I had an ego already at 6-years-old). And I got frustrated with everyone telling me it was easy. "Just go faster and you won't fall!" "Just keep your balance!" "Just don't think about falling and keep pedalling" 

Inside my head, all I could think was "**Well if it was so easy, why couldn't I do it?"

And so, *I stopped trying.* After all, I reasoned, **if I'm going to fall down every three feet, wouldn't it be faster if I just ran** Obviously, to anyone who has either ridden a bike or seen a bike, you know a bike lets you go far faster than a human can run, and with far less effort. But for me, in that moment, **it was easier to not learn and exhaust myself running than it was to learn.** Because learning a new skill *is* hard!

"Okay", I thought, "So I'll just use my training wheels forever." Except the dirty secret about training wheels is this: **they slow you down**! When you lean on the training wheels, you have more friction! More friction means you're converting all that wonderful speed into useless heat and sound, and you have to pedal even harder to go the same speed. All I was doing was riding slow and [speeding up the entropy of the Universe!](https://en.wikipedia.org/wiki/Heat_death_of_the_universe)

This is a perfect parallel to conversations I have with students every semester about, for example, unit testing. "It takes me a really long time to write unit tests." "It's just busy work that slows me down." "If I know my code works, why di I have to test it?" "Writing these tests is slowing me down".

And you know what, they're right! **Unit testing does slow them down!** But that isn't because unit testing slows everyone down. It's because they are like me **trying to ride a bike for the first time without training wheels**: they keep falling over. They don't write effective tests because they had an auto-grader that did the testing for them. Or they take too long to think about what tests to write because the auto-grader told them what types of inputs they were failing on. Or they write their code in a way that makes it difficult to test because they don't know how to write testable code.

Students don't do this because they are "bad programmers". They do it because they are **inexperienced** and relying on the same skills they used in an introductory programming course.

But the only way to get better, faster, and more efficient, is **PRACTICE**. No one has ever learned to ride a bike without falling off it at least a couple times.

---

## You won't be good at first

Programming is like riding a bike. No, not that you'll never forget how to do it; you can definitely forget things. Programming is like riding a bike because **when you first start learning, you lean on your training wheels.** 

In the two previous programming courses at the University of Virginia, your programming assignments looked like the following:

&nbsp;&nbsp;1) Here is the program you are going to write  
&nbsp;&nbsp;2) Here is the programming language you will use  
&nbsp;&nbsp;3) Here is the IDE you will use  
&nbsp;&nbsp;4) Here is exactly how it will work  
&nbsp;&nbsp;5) Here are the functions you are going to write  
&nbsp;&nbsp;6) Here are the classes you are going to write  
&nbsp;&nbsp;7) Here are the fields in your classes  
&nbsp;&nbsp;8) Here is what to name everything  
&nbsp;&nbsp;9) Here is how each function affects or is affected by each field  
&nbsp;&nbsp;10) Here is a submission system, where you submit your code and get feedback telling you if it's right or wrong.
&nbsp;&nbsp;11) Here are the exact library import statements to use  
&nbsp;&nbsp;12) Here are plenty of lecture examples using that library  

**These are training wheels!** Of course when you are learning we're giving you this much direction! We're helping you learn the foundational basics. But just like riding a bike, if you want to go fast and go far, **The training wheels have to come off!**

When you develop real software, here is how many of those conversations start:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Customer**: I would like software that does [vague description of something the customer does already without software]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Engineer**: Okay, can you be more specific?

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Customer**: \[*staring blankly*\] no.

Admittedly, the above is in jest and over the top, but there is an entire field of software engineering, Requirements Engineering, that focuses specifically on techniques to get and check the basic details of "what the software should do", because it's a very hard, and very important problem. You'll see more of that in CS 3240 (or, if you aren't at UVA, in a Software Engineering class).

This means:  
&nbsp;&nbsp;1) **You will decide** what program to write  
&nbsp;&nbsp;2) **You will decide** what programming language(s) you will use  
&nbsp;&nbsp;3) **You will decide** on how to develop the software  
&nbsp;&nbsp;4) **You will decide** how it works  
&nbsp;&nbsp;5) **You will decide** what functions to write  
&nbsp;&nbsp;6) **You will decide** what classes to write  
&nbsp;&nbsp;7) **You will decide** what fields your classes have  
&nbsp;&nbsp;8) **You will decide** how to name and label everything  
&nbsp;&nbsp;9) **You will decide** how each class behaves and interacts with other classes  
&nbsp;&nbsp;10) **You will decide** how to figure out how to tell if your code is working or not  
&nbsp;&nbsp;11) **You will decide** how to investigate if there is code out there that you want to use, what the licenses allow for, etc.  
&nbsp;&nbsp;12) **You will decide** what documentation or tutorials to use to learn a new technology/language/library.  

And, I have some bad news. You will make *several* mistakes along the way. But...

---

## You will get better!

> "I wish I lived next to Carnegie Hall. Then, if someone asked me how to get to my house, I would just say 'Practice, practice, practice, and then take a left."
> __Dimitri Martin__

The good news is practice and time make you better at all of the above! I did eventually learn how to ride a bike. And even after learning, I continued to get better with balancing, navigating traffic and rough roadways, pacing myself so I don't arrive exhausted, etc. And the key tool was **PRACTICE**. The practice had to be frequent and spaced out, because that's how humans learn! We don't learn by cramming things at the last minute.

My first programming language I learned was TI-BASIC 83 (a dialect of BASIC written for the TI-83 graphing calculator). I only learned *just enough* to write a program on my TI-83 that would take in 3 parts of a triangle (Side-side-side, side-angle-side, etc.) and "solve" the triangle. I was taking trigonometry at the time, and so I showed my teacher the program I wrote, walked him through it. And he was impressed!

Just kidding, he made me delete it, line by line, right there in front of him. If you think I'm still bitter, understand that I was 15 at the time, and as of this writing I'm in my mid-30s, and I'm still bitter. I did ultimately rewrite it on my own later at home on my family's laptop. In my early 20s, I wanted to include it in a portfolio for grad school, saying "look, I've been programming for years!" It was still on a hard drive, so I asked my stepdad sent it to me.

The code is **awful**. It's hard to read because of tons of duplicate code and overly complex structure. It's littered with gotos because I didn't understand the idea of writing functions. The variable names are horrible, all one letter *a*/*b*/*c* names. There is a ton of duplicate code. There was actually a serious bug in solving side-side-side triangles that caused the program to crash if the triangle was obtuse that I never noticed.

In short, I realized that 15-year-old me was **awful** at writing code. Mid-thirties me is a lot better. I've gotten much better at writing code since through practice. But would I call myself an "expert programmer"? No! Because I still have a long way to go! And the way I get better is practice!

Even during summer or winter break when I'm not teaching, I try to write some meaningful program at least once a month to keep my skills sharp, practice a new design pattern, etc. Because I only know one way to get better at anything, **practice!**.

---

## Learning a new language

But back to BASIC: it took me about an hour to figure out how to *print the value of a variable* in basic. That command? ":DISP [variable_name]". That simple command cost me an hour of my life. And I felt dumb, and like I was learning slower than I should have been.

The next language I learned was Java. And I struggled with it. Semi-colons, when to declare a variables type, public static void main(String\[\] args), Objects and classes, interfaces, data structures, FileNotFoundExceptions, on and on and on. And it was hard. But, made easier because I understood if statements and variables ahead of time.

And then I learned Python. And I found it easier to do scripting, but I struggled with the Object-Orientation. It still took me a couple weeks, but certainly not the months of Java.

The next language I learned was C, and...well other than learning how to deal with pointers, I found it pretty easy to learn. It has a ton in common with Java, sans object orientation. But since I only needed it for some low level multi-threading, I didn't worry about this feature not being there. But, **man** I struggled with pointers. Why? Because they were new!

And then came Common-Lisp. Fundamentally very different, but everything was basically functions which I was more than familiar with, so it took me three weeks to get the basics down. I wouldn't call myself "proficient", and I use the `proc` keyword far too much in Lisp, but I will say learning it made me a lot better at writing functional code.

Flash forward to the weekend before I'm writing this, and I wrote a solution for the first homework in this class in Kotlin, a language I had never used before I started writing the code. Is this because I'm just *that* good? No, far from it. It's because I've had practice learning new languages, and I know what I need to look for and where to find help. It's a skill, just like any other. Kotlin is similar to Java in many ways, so I just connected my knowledge that way. And you get better at it each time you learn a new language.

---

## Summing it up

You will struggle when presented with new information, or a new challenge. And learning how to do things "the right way" sounds a lot like "eat your peas and carrots."

But understand that these techniques and practices were developed precisely because they make building larger, maintainable evolving systems faster, more reliable, and more efficient than the ad hoc approaches you may be currently used to. And these techniques have stuck around because, once programmers learn them, *they work*! For instance, while I support Test Driven Development (TDD), I'm a pragmatist. The second I find it isn't helping me, or another approach will be better, I will abandon TDD. I just haven't found that technique that consistently produces better results.

Just like me riding a bike without training wheels, you'll fall down, you'll make mistakes, you'll get frustrated, and you won't go particularly fast at first. But each time you do Test Driven Development, or decompose one class into two classes, or properly integrate a Design Pattern, you'll get a little better. It will feel more natural. And, before long, **you'll realize that you're going much farther faster on this bike than you ever could have gone on foot.**

> "I may not have gone where I intended to go, but I think I have ended up where I needed to be." - __The Long Dark Tea-Time of the Souls__ Douglas Adams

