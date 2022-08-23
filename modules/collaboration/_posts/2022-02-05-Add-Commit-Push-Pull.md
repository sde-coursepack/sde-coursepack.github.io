---
Title: Git Repository Operations
---

# Repository operations

Now that we have branching down, let's look at basic repository operations.

### Using IntelliJ

Once again, I strongly encourage using IntelliJ or any other Git GUI
if you are first starting out.

### ```add```

The name `add` may imply that the command is for adding new
files to the repository, but you actually use add for all changes.

`git add my_filename`

Adds any changes to the file to the repository. If the file hasn't
been added before, it adds the whole file. In either case, we call
these changes "staged", as in ready to be stored in the repository.
To add all files that aren't filtered out by .gitignore, you can run `git add .` (the last
item being a period) in the project's root directory. **Make sure
you have a correct .gitignore in place before you do this!)

### ```commit```

Committing means saving your staged changes to your local
repository. Commits require a meaningful commit message. The
format is:

`git commit -m "your message goes here"`

You cannot commit individual files, because commit automatically
stores all **staged** changes (that is, changes we have run
git add on). So your message should describe all staged
changes.

Your commit message should clearly communicate what the commit
contains. You don't need to write a novel, but it should be something
like "refactored myBigFunction to make myClass more readable", 
"fixed negative number crash in getClassByIDNumber", or
"continued working on CSVReader class". You may need to understand
what each commit did later on if you are trying to backtrace a bug,
or retrieve code that has been lost.

### ```push```

This pushes all commits in your current branch
since the last push to the remote repository.

`git push`

This can and will push multiple commits at the same time. Git will
automatically determine which commits to push to keep things simple.
**However, remember to always pull before you push**.

### ```pull```

`git pull`

This gets the most recent changes from the remote repository for your
current branch. Be aware that pulling only pulls for your current
branch, not for every branch. Whenever you checkout a new branch,
you should always pull.

**If you get an error when you try to pull** that reads something like:
"Your local changes to the following files will be overwritten by merge",
this means that you have changes that have not been committed.

However, because this means your local repository is behind the remote
repository, you won't be able to commit your changes. The solution is to use the 
stash. The stash is basically a stack for saving changes that, for whatever reason,
you don't wish to commit, but don't want to lose. Simply use:

```
    git stash
    git pull
    git stash pop
```

...and you should be good to go.

## Commit hashes

If you look at the commit history [for the Gradle Tutorial](https://github.com/sde-coursepack/NBAExcelTeams/commits/main)
that we will cover in the next unit, you will see each commit comes with a hexadecimal number
on the right hand side. For example, the most recent commit is c568af0. These are actually the first
7 hexademical characters of a hash that uniquely identifies the commit. Rather than using incrementing
numbers, GitHub uses a hashing system to manage commits.

If you need to use the commit number, you can simply use those 7 digits. If needed, you can also find
the full hash by [clicking on the commit hash](https://github.com/sde-coursepack/NBAExcelTeams/commit/c568af0cfe3709c466ad2767fc31736540df6676)

Where you will see it on the right next to commit:

`c568af0cfe3709c466ad2767fc31736540df6676`


## Conflict Resolution


## If you get stuck


## GitHub Permissions Issues on IntelliJ and Terminal