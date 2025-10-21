---
Title: How Java Works
---

# How Java Works
{: .no_toc }

This module will discuss the nature of how Java code compiles and executes.

---

## Contents
{: .no_toc }

* TOC
{:toc}

---



## What is code?

Code is instructions that a computer can follow.

Programmers have to read code. The very act of
debugging, for example, is reading your code and 
looking for a defect to fix. However, the way a
programmer reads code is very different from how
a computer reads code.

When humans read code, we:

* misunderstand things
* skim over uninteresting or simple things
* make incorrect assumptions

Obviously, computers can't do this. Computers need
code that it can translate into instructions. This
code must be:

* unambiguous
* syntactically correct
* logically possible (such as no null pointers)

The earliest programming involved carefully defining every
machine operation, one by one, often with mechanical or
physical inputs used to communicate instructions to the
computer. Often, these instructions were written in a
form of binary. The thing is, computers today fundamentally work
the same way; we've just replaced the mechanical parts
with small, faster electronic parts. We've traded in
punch cards for keyboards, but we still have to write precise code.

From the software side, we found that writing code in
binary was cumbersome, time-consuming, and error--prone.
So human-readable languages started being developed.

Early languages, like Fortran, added human-readable
identifiers, like "IF," "PRINT," etc. You can see
such structure in this [Greatest Common Divisor code](https://en.wikibooks.org/wiki/Fortran/Fortran_examples#Greatest_common_divisor)
for Fortran 77.

---

## Compiling

It's important to understand that, when you write source code in modern high-level language,
your computer actually can't run your source code directly. Typically, we write code, and compile that code into something that can be executed by the computer. The below diagram is a simplification of this relationship:

<img alt="Image shows source code with an arrow pointing to a box labeled code compiler, which has an arrow pointing to a box labeled Executable File" src="{{site.baseurl}}/modules/java/images/3/compiler1.png"/>

---

## Compiling in C

Before talking about Java, let's talk about how C works,
and how C code is turned into an executable program.

```c 
    #include <stdio.h>
    int main() {
        printf("Hello, World!");
        return 0;
    }
```

I can write the above code in any plain-text editor,
like Notepad (though not "document editors" like Word),
and save the file as ```helloWorld.c```.

<img alt="A screenshot of notepad with the helloWorld.c source code" src="{{site.baseurl}}/modules/java/images/3/helloWorldC.png"/>

Now, if I save this file to a folder called "code",
I can open a terminal (Mac/Linux) or Powershell[^1] and navigate to that folder. If I use the "ls" (list) command, I can see the file in my directory.

<img alt="A screenshot of a terminal window, ls has been executed showing a file list including helloWorld.c" src="{{site.baseurl}}/modules/java/images/3/shell1.png"/>

However, that's all I can do with it. If I try to run
the file as though it were a program, my Windows operating system thinks I'm trying to "open" the file.

<img alt="When I typed helloWorld.c into my terminal, a pop-up window asks me what I want to open the file in, suggesting several text editors" src="{{site.baseurl}}/modules/java/images/3/shell2.png"/>

That's because **source code isn't an executable program.** Source
code is meant for humans. We write the source code using a
programming language because it's easier to read than those
low-level machine instructions. If we want to run our program, we must **compile** it.

---

## Why not just write low-level machine code?

**Compiling** is turning human-readable source code
into computer readable instructions. We can
compile c files with a program like GCC. So, I compile
the code into a program called ```helloWorld.exe```, and
then open in Notepad and:

<img alt="I opened helloWorld.exe in notedpad, and the contents of the file appear to be random unintelligible characters" src="{{site.baseurl}}/modules/java/images/3/executable.png"/>

What is going on here?!?! Well, what is happening is that the .exe
program contains the low-level machine instructions that tells my
operating system how to run the program. This machine code
is stored in bytes which do not properly translate to human-readable text in a format that Notepad knows how to display. However, it's quite all right that this code isn't
human-readable! It doesn't need to be human-readable! If
we want to change the program, we can change the C code and recompile.

We never have to work directly with the contents of the .exe file, 
which, as you can see above, *is a definite bonus*, because even
in the best of circumstances, working with that machine code
is cumbersome, difficult, and very limiting.

---

## Advantages and Disadvantages of .exe

The advantage is that I can run this program as is! I don't
need any special software! In fact, anyone with a modern Windows
OS could run this same .exe file, meaning I can share the
runnable program without sharing the source code! Anyone 
running this code doesn't even need to install a C IDE or compiler!
This is why  when you download a program, it often starts by downloading
an executable first. However, notice that whenever you download a
program, you typically have to select your operating system...

The disadvantage, *which we will come back to when talking about Java*,
is that this executable only works on my operating system, Windows.
Linux and Mac executable files are fundamentally different, and
so even a simple executable like helloWorld.c (which only prints "Hello, World!")
is not compatible with any other operating system. Now,
there are ways to do "cross-compiling," that is, compile for
a different environment than the one you are programming in,
like compiling a Linux executable in Windows, but now you have
to consider if every part of your code will work on a given operating system.
In fact, you may at times see C code like this:

```c
#ifdef linux
   //do something with relation to linux
#elif _WIN32
   //do something for Windows 32
...

```

This limitation, that we can't make portable executables
is something Java was built to address.

---

## Interpreters

Some languages, like Python[^2], are typically *interpreted* rather than compiled. This is fundamentally the same idea as compiling, only rather than turning the entire program into machine code first, and then running second, when code is interpreted, we do both at the same time with the help of another program.

In Python, as you execute the program, the Python interpreter translates each line into machine instructions as you come to it. This means you typically don't have a static compiled file.

---

## Translation Metaphor

> "Hooray for metaphors"
> - Sterling Archer, __Archer__, "Skytanic" Season 1 Episode 7 

The difference between compiling and interpreting can be explained like the United Nations: 
imagine  you are a United Nations ambassador who is only fluent in English, and a speaker is giving a speech in Spanish. There are two solutions:
* Wait, twiddling your thumbs, for someone to produce a translated transcript after the speech is over [*compiling*] 
* Use headphones to listen to a live translator who is translating the speech as it is being made [*interpreting*]

The advantage of the first approach is that you have an easily
redistributable transcript of the speech that anyone who can read English can use, regardless of whether or not they have access to a live translator of their own. 
The disadvantage is you have a transcription ("compilation") process to produce the translation, and you can't do anything until it is done.

The second approach has the advantage of translation on the fly, but it's only possible with the help of a live translator actively working in the background (python's interpreter). And a live interpreter has to do a lot of work on the fly. Additionally, anyone without a translator cannot use this approach.

Note that the above is an imperfect metaphor. **Do not make any assumptions about which process is more efficient** from this metaphor; the point is only to explain the difference between compiling and interpreting. It's worth noting that, in practice, 
interpreted languages tend to be a fair amount less efficient than compiled languages.
Specifically, C is routinely consider the fastest/most efficient language, which is why it's so useful for low-level applications and embedded systems, while Python is actually *dramatically* less efficient than C. Despite what you may have heard,
Java is actually capable of being quite efficient (although often there is a trade-off between efficiency and design, especially as you leverage object oriented programming).


This is why you *don't* need to install C to run a program written in C, since it can be compiled into an executable. However, you *do* need to install Python to run Python programs. 

---

## How Java works (finally)

Okay, now we're ready to dive into specifically discussing Java.

### JDK
Much like C, Java is a compiled language. When you compile a Java file, the Java Development Kit (JDK) compiles your code, producing a .class file. The .class file is the bytecode that specifies the machine instructions. Much  like the C executable, the compiled file is not, nor intended to be, human-readable.

<img alt="An image showing a .java file being compiled into a .class file which is run on the computer" src="{{site.baseurl}}/modules/java/images/3/compiler2.png"/>

However, unlike C, the Java .class file is **portable**. **Portable**, in the context
of software, means that you can move software easily from one environment to another
with minimal effort. In C, to move a program from Windows to Linux, you have to recompile
the program. In Java, you just send the .class file, no recompiling needed!

How does it do this? Well...first I need to acknowledge that the last figure is a lie.
A more accurate figure is this:

<img alt="An image showing a .java file being compiled into a .class file which is being used by a box labeled Java Runtime Environment, which is running on the computer" src="{{site.baseurl}}/modules/java/images/3/compiler3.png"/>

Specifically, the .class files don't run on any particular hardware. Rather, they run in a virtual Java Runtime Environment (JRE).

### JRE

A **JRE** is an environment used to run Java Programs. The **JDK**, by contrast, is
use for *compiling*, not *running* Java programs. When you download a JDK, it will include
a compatible JRE. However, users can download a JRE to be able to run Java programs
without installing a JDK (non-developers wanting to run Java programs will often do this).

JREs also contain various class libraries (like the core Java class libraries). In a specialized runtime environment, the included libraries could be changed, creating a custom JRE. That, however, is beyond the scope of this course.


### JVM
Each JRE contains a **Java Virtual Machine (JVM)**. A *virtual machine* is a software resource
that acts like a separate computer; virtual machines, functionally, have their own operating system, 
memory, processor, instruction set (machine language), etc.
However, each of these resources is effectively borrowed from the host (physical) machine.

What this means is that the JVM handles the direct interactions with the actual underlying hardware
(computer CPU, physical memory, disk, monitor, etc.), but from the perspective of the JRE, the
underlying physical hardware is irrelevant. It's just a Java Virtual Machine.

It's worth noting that the JVM is actually an *interpreter* that interprets the code it is
given at runtime. However, that code *is not Java source code*, but rather .class *bytecode*
compiled by the JDK, and passed to the JVM by the JRE.

### Restaurant metaphor

> "Hooray for metaphors"
> - Sterling Archer, __Archer__, "Skytanic" Season 1 Episode 7

I like to think of the JRE as a waiter, and the JVM as the chef. A .class file looking
to be run walks into the restaurant, sits down, and starts asking the JRE to cook up a 
nice machine instruction for them. The JRE takes the order back to the chef (aka JVM), the JVM
uses the physical resources (oven, stove, fryer, etc.) to actually complete the order.

As far as the .class file is concerned, they don't care whether the restaurant is using electric or gas appliances (*think Mac or PC*), because the *chef* (aka JVM) handles
that. The JRE has a standardized interface (it doesn't matter what "operating system" the chef
is using, because the customer never talks to them), so the .class file only needs to worry about interacting
with the JRE. Whether it's a Mac JVM, PC JVM, Linux JVM, Android JVM, etc. doesn't matter
to the .class file or the JRE.


### JIT

Obviously, one of the most important interactions the JVM has is with the processor, the
"brain" of the computer that actually handles executing instructions. As we mentioned before,
the JVM interprets Java bytecode. However, as we mentioned before, interpreted code is
inherently slower than directly executed machine instructions.

Making the process of interacting with the processor/memory/etc. as efficient as possible can lead to significant performance gains. 
*Enter the Just In Time compiler (JIT)* The JIT is part of the JVM, specifically the part that can compile JVM bytecode instructions
into machine code instructions for the underlying hardware. This natively compiled machine code,
machine code specifically compatible with the underlying physical hardware, is much more efficient than
interpreted Java bytecode. The JIT can have numerous optimizations to increase performance, reduce
memory consumption, etc.

---

## Why Java is built this way

A key advantage of Java is portability. That is, an application built and compiled with a JDK on
a Windows computer will predictably behave the same way on a Linux and Mac computer
provided that both have a compatible JRE installed.

This allows us to share executables, without sharing underlying source code, of
apps. The JRE provides enough features to build meaningful and complex applications
that can be run on hardware that has a compatible JVM.

The JIT provides several optimizations between a JVM and a specific 
underlying hardware architecture, ensuring that Java is capable of being efficient.

All of this together gives us convenience, distributability, portability, and performance.

---

## JVM: It's not just for Java anymore

The JRE can run any compatible bytecode, and does not require a specific JDK. In fact, the JRE
doesn't interact with a JDK at all. This fact allows for other programming languages to interact with the JRE, provided
they can compile correct .class bytecode. Two languages that do this are **Kotlin** and **Groovy**. 

**Kotlin** has been gaining rapid popularity in the Android community (as Android is a JVM
base system). The growth accelerated dramatically when, in 2019, [Google announced
that Kotlin was now its preferred language for Android app development](https://techcrunch.com/2019/05/07/kotlin-is-now-googles-preferred-language-for-android-app-development/).
I personally enjoy the language, and it's worth looking at (we will *very* briefly later on
in this class), as it cuts down on Java's length, removes some annoying features like checked exceptions,
and adds some useful features like null-safety.

**Groovy** similarly works on the JVM, and we will use a small amount of Groovy when working with
*Gradle* later on. However, beyond its use in Gradle as a domain-specific language (DSL), it never
gained the general-use popularity that Kotlin is now seeing.

---

## Key takeaways

Key takeaways.

* Compiling is taking a source code resource, and producing a bytecode resource
designed to run on a particular machine
* Interpreting is similar to compiling, but is done dynamically at runtime
* Java uses a JDK for compiling in bytecode that runs on a virtual machine
* The JRE runs Java programs via a JVM
* The JVM interprets Java bytecode into machine instructions of the underlying hardware
* The JIT is used to compile the java bytecode into machine instructions for the underlying hardware
at runtime and boosts optimization and efficiency.


[^1]: Note for Windows users, I strongly recommend using Powershell rather than command-prompt, as Powershell uses pretty much the same commands as Mac and Linux, and helps standardize everything. Powershell comes installed with Windows 8 and later.


[^2]: A note that while Python is often referred to as an interpreted language, the truth is a little murkier. Python does, actually, "compile" code, though there is actually a compilation process in Python to translate instructions into bytecode
