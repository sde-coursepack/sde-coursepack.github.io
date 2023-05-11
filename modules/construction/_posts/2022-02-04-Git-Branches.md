---
Title: Git Branches
---

* TOC
{:toc}

# Branching

**We are covering branching before add/commit/push because
understanding and working with branching is vitally important.**
You should always be in a non-main branch when writing new code!

One of the most important concepts in any version control is the
concept of branching. By default, when we create a git repository,
we will have one branch called `main` or `master`, depending on
what we use to create the repository (Github will create
repositories with 'main', while using `git init` on your computer
locally will typically create the branch 'master'). For the
rest of this unit and future modules, when we say "main" or "main branch",
this is typically the branch you mean.

Now, if you are working alone on a person project you do not plan
to release publicly, it may make sense to only work in main. However,
there are several reasons we want to avoid primarily working in the main
branch:

* If we are releasing this software publicly, we want to avoid
  adding new work directly to main, as any partially completed
  or untested work could break our released software.
* If we are working with multiple people, and everyone is
  committing to main for everything, we'll end up wasting a lot
  of time dealing with conflicts (see Conflict Resolution below).

As such, we want to instead work in a separate branch, and then
merge our work into main when it's ready to be released. Generally,
we create a branch for each feature we are developing, and merge
that branch into other work once the feature is finalized.

### Using IntelliJ's Git GUI for Branching

This is **highly recommended** when you are first getting used to
working with git, as the GUI means you won't have to worry
about remembering specific commands.

In IntelliJ, you can manage branches by going to Git-> Branches,
and using the GUI, either add a branch ("+ New Branch"), which
boths creates a **checks out** (switches your working directory to)
the new branch. The starting state of this new branch will be whatever
the working branch and working copy of the files you have. You can
also do all of this with the "Git" tab in the bottom right by
right-clicking on the branches to switch between them or add a new branch.

You can also handle branch merging. Just got to Git-> Merge and
select the branch you want to merge into the branch you are currently
working in. If you want to merge your current branch into another branch,
checkout the other branch first, and then you can use merge. As a general
rule, if you are working in a feature branch, you should merge the development
branch (which may be "main" or some other branch depending on your group's
workflow) into the feature branch *first*, handle any conflicts, commit, and
then checkout your development branch and merge your feature.

### ```branch```

In terminal, you can see all existing branches with

`git branch`

After you hit enter, you'll see something like:

```shell
* development
  main
```

The asterisk next to development tells me I'm currently *in*
the development branch, but that there is also a main
branch.

#### creating a new branch

You can add a new branch with:

`git branch my_new_branch`

Example using my terminal in a project called HibernateDemo:

```
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git branch
    * development
      main
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git branch my_new_branch
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git branch
    * development
      main
      my_new_branch
 ```

Be aware that creating a new branch this way **does not check the
new branch out**. Your working copy will still be whatever branch you
started with. You will need to use `git checkout my_new_branch` to switch
to the new branch.

### ```checkout```

"Checkout" means "I want to switch to this branch". When you switch
branches, your working copy will be updated to the most recent
commit in your local repository in that branch. **You should always
commit to your current branch before switching branches.**

`git checkout branch_to_switch_to`

```shell
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git branch
  * development
    main
    my_new_branch
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git checkout my_new_branch
    Switched to branch 'my_new_branch'
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git branch
    development
    main
  * my_new_branch
```

### ```merge```

Once you are ready to merge your work, you need to use the merge command.
Merge does what you think it would do, it merges the repositories of
two seperate branches. However, a merge is a one directional operation:

If I say merge from branch_a to branch_b, what I mean is copy any
repository changes from branch_a to branch_b, but *not the other way around.*
That is, merging from branch_a to branch_b only affects the repository in
branch_b. So if I wanted branch_a and branch_b to be sync, I would first merge
from branch_a to branch_b, resolve any conflicts, and then from branch_b
to branch_a.

Let's imagine we wanted to merge our new branch into `development`. As a
general rule, when we are merging from a single feature branch into a
development branch (or main branch), we want to **first merge from development
to our feature branch**, resolve any conflicts *within our feature branch*, and
**only then** do we merge from our feature branch to development.

**Merging from development to my_new_branch**

```shell
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git branch
    development
    main
  * my_new_branch
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git checkout my_new_branch
  On branch my_new_branch
  * my_new_branch
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git merge development
  Auto-merging src/main/java/edu/virginia/cs/Main.java
  Merge made by the 'recursive' strategy.
  src/main/java/edu/virginia/cs/Main.java | 2 +-
  1 file changed, 1 insertion(+), 1 deletion(-)
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git branch
    development
    main
  * my_new_branch
```

If I had any conflicts, I would handle them, and then commit.

**Merging from feature branch to development**

```shell
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git checkout development
    Switched to branch 'development'
PS C:\Users\pm8fc\sde-Homeworks\HibernateDemo> git merge my_new_branch
    Updating c54b5fd..5f305f1
    Fast-forward
    src/main/java/edu/virginia/cs/Main.java | 2 +-
    1 file changed, 1 insertion(+), 1 deletion(-)
```