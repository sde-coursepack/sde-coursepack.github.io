---
Title: Defensive Programming
---

# Defensive Programming

I like to introduce the idea of defensive programming by talking about defensive driving.

When you were learning to drive, you were likely told that you should drive *defensively*. But what does that actually mean? Are we saying you should drive ready to protect yourself, with violence if needed, like some Mad Max action sequence? 

As fun as that sounds, no. **Defensive driving** is the idea that, when driving, you should be prepared for people around you to *do incorrect things*. For example, the car you are passing in the right lane may try to switch lanes for no clear reason while you are passing, so you should pay attention to that car as you pass. When the traffic light turns green, you should still look before you go to ensure the intersection is clear or no one is running the red light. As you go around a blind corner, you should be mentally prepared in case someone coming the other direction strays out of their lane and into yours.

Driving defensively is *driving so that if other drivers make a mistake, you won't get caught up in it.* You cannot control how other people drive. You can only drive in a way to maximize your own safety.

## Understanding Defensiveness

Considered writing a command line program that takes in the name of, say, an Excel file, and performs some data manipulation or summarization based on the contents of the file. Consider all the ways the program could fail that are *not* the fault of the developer of that program:

1) The user may not include any command line arguments
2) The user may link to an invalid file
3) The user may have errors in their input file, such as missing columns, or bad format.
4) The user may have the file open in Excel, which can result in the file not being openable within your program

In general, all of these exceptions are not the fault of the program, but rather how the program is being used.

These exceptions are fundamentally different from programmer errors, like NullPointerExceptions caused by not correctly initiating your data structures. In essence, we are drawing a line between programmer error and user error.

In general:

1) A user should never see a programmer error: the program as released to the user should be stable and function correctly.
2) A user should be informed when they make a user error. The error message should include a clear description of what the user did incorrectly in their interaction with the software

## Client Class

When developing our own classes, we have to consider **clients**. A client class is any class that uses the code we are writing. This can be another class in the project, or this could be someone using our class like a library (such as the spreadsheet libraries in poi). In this case, we want to design our class so that:

1) The client uses our class correctly and as intended
2) If the client uses our class incorrectly, they are made aware of this
3) Our class cannot be used incorrectly by the client to produce an erroneous state.

## Example: `BankAccount`





