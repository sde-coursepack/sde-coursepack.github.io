---
Title: Best Practices for Git
---

* TOC
{:toc}

# ```git``` good - Best practices for git

Here are some best practices to keep in mind when working with git.

## Commit a .gitignore before committing anything else

Make sure you commit .gitignore before you start committing code or any other files, save for maybe
a README and License.md. This is important, because .gitignore **is not retroactive**. You can remove
files from the repository while keeping the local file with:

`git rm [file_name] --cached`

The --cached argument is very important. If you just say git rm without cached, it will delete
the file from **both the repo and your working copy.**

## Always pull before you start working

Always make sure you have the most up-to-date code before you start writing code. The worst case scenario
is that you don't pull, spend hours writing code, and then you have tons of conflicts to resolve that could
have been avoided by simply pulling first.

## Always pull before pushing and merging

You don't need to pull before every commit, but you *need* to pull before you push
or merge. This is because you want to handle any potential conflicts before you
do a more complex operation.

Before a push you should:

1) Commit your code
2) Pull
3) Resolve any conflicts
4) Commit
5) Push

## Merge into the more specialize branch first

If you are working in a feature branch you want to merge with main:

1) Commit your changes in your feature branch
2) Checkout main
3) Pull main
4) Checkout feature branch
5) `git merge main` to merge main into your feature branch
6) Resolve any conflicts, commit and push
7) Checkout main
8) `git merge featureBranch`

You want to handle conflicts in the most specialized branches first. That way, if
a conflict requires a more substantial change than just "pick the one you want", you can
handle that separately without breaking the program on the main branch.

## Always push when you stop working

If you have written a ton of code, and are done for the night, *push*! The longer you go without
pushing, the more chances there are to get a ton of annoying conflict resolutions to deal with.

## Commit early and often

Many students will tell me "well, I didn't commit because I didn't have anything finished," and while
this is understandable, in a distributed, branched system like Git, you can commit safely frequently.

Generally, you should think of commit like saving a file in a word processor. Write a paragraph, save.
Fix a typo, save.

Note that this doesn't mean "push early and often." You should push frequently if you can, but you
will generally have more commits than pushes.

## Never commit new code directly to main/master

You will be working with teammates this term. Committing new code to master (especially 
untested code) can be a recipe for disaster. As much as possible, you always want main working,
even if main doesn't have all the features.

Generally, in larger projects,0 I like to have two "special" branches where you 
don't write new code directly.

main - this is the current working version of the product
development - this is the main "development branch"

From there, everyone works in a branch off development. Each feature is a new branch. When that
feature is completed, it's merged into development. You then do integration testing
in development (that is, make sure that all the new features added are working together
correctly). Once you are confident development is working, *then* you merge it to main.
Each time I push to main I am effectively creating a new software release.

## Once a feature is completed, delete its branch

Always take a second to clean-up after yourself. Having tons of branches lying around
can cause confusion among your teammates. "Which branch should I be working in?" shouldn't
be a question anyone needs to ask. Make the branch names clearly communicate what feature
they are working on.

## Do not make dozens of commits at the last second!

In teaching this class and CS 3240, I have routinely seen students who procrastinated
making tons of small commits at the last second, often directly to main. Either trying
to fix or hide a bug, fixing some minor typo, or just trying to make one function work
right before the deadline. Every semester, I get students who, because of a last second
commit, have code that won't compile anymore, and as a result they end up losing far more
points than they would have had with that one minor bug.

Accept that you need to start early, and plan to finish 2-3 days early, so you have time
to catch and fix small bugs, safely knowing the major parts of the program are done.

## Use good commit messages early, and keep forcing yourself to use them.

<img src="https://imgs.xkcd.com/comics/git_commit.png">

This is especially true when you finish a feature or make changes to
an existing class! You want to track this progress! Avoid "fixed some bugs"
and "more work". Git messages are your chance to communicate what you did.
Messages can be short. For instance "added test for getDistance(Location l)"
is perfectly sufficient. It communicates what you were working on, and what
your goal was.

Eventually, you may need to go back through hundreds of commits and find where
a particular change was made. If all of your messages are "message", you're
going to be mad at your past self for their selfish and lazy decisions!