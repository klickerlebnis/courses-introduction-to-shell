---
title       : Creating new tools
description : >-
  History lets you repeat things with just a few keystrokes,
  and pipes let you combine existing commands to create new ones.
  In this chapter, you will see how to go one step further
  and create new commands of your own.
  
--- type:NormalExercise lang:shell xp:100 skills:1 key:864f978f0d
## Shell scripts

You have been using the shell interactively so far:
each time you want to do something,
you enter a command
and the shell runs it right away.
But since the commands you type in are just text,
the shell lets you store them in files to use over and over again.

To start exploring this powerful capability,
put the following command in a file called `dates.sh`:

```{shell}
cut -d , -f 1 seasonal/*.csv | grep -v Date | sort | uniq
```

Reading from left to right,
this selects the first column from all of the CSV files in the `seasonal` directory,
removes lines with the word "Date",
sorts the result,
and removes duplicates.
In short,
this pipeline produces a list of the unique dates in the seasonal date.

Once you have created this file,
run the following command:

```{shell}
bash dates.sh
```

This tells Bash (which is the shell we are using)
to run the commands contained in the file `dates.sh`.
Sure enough,
it produces the same list of dates that would have been produced
if you had typed the commands directly into the shell.
By putting those commands in a file,
though,
you have saved yourself having to remember the steps a second, third, or hundredth time,
and you can be sure that you are doing exactly the same thing
each time you get the files' unique dates.

A file full of shell commands is called a *shell script*,
or sometimes just a "script" for short.
Shell scripts don't have to have names ending in `.sh`,
but this lesson will use that convention
to help you keep track of which files are scripts and which are not.

*** =instructions

Create another shell script called `teeth.sh`
that prints a count of the number of times each tooth name appears in the seasonal data.

*** =hint

*** =pre_exercise_code
```{shell}

```

*** =sample_code
```{shell}

```

*** =solution
```{shell}
cut -d , -f 2 seasonal/*.csv | grep -v Tooth | sort | uniq -c
```

*** =sct
```{shell}

```

--- type:NormalExercise lang:shell xp:100 skills:1 key:1b0da86491
## Passing filenames to scripts

A script that re-runs exactly the same commands on exactly the same files is useful
as a record of what you did,
but one that allows you specify what files you want to process is usually more useful.
To help you do this,
the shell has a special expression `$@` (dollar sign immediately followed by ampersand)
that means "all of the command-line parameters given by the user when the script was run".
To see how this works,
edit `dates.sh` and replace `seasonal/*.csv` with `$@`
so that the file contains the line:

```{shell}
cut -d , -f 1 $@ | grep -v Date | sort | uniq
```

If you run the command with:

```{shell}
bash dates.sh seasonal/summer.csv`
```

then the shell replaces `$@` with `seasonal.summer.csv`
and the pipeline processes only that file.
If you run it like this:

```{shell}
bash dates.sh seasonal/autumn.csv seasonal/winter.csv
```

it processes those two files,
while:

```{shell}
bash dates.sh seasonal/*.csv
```

processes all four files,
just like the original script.

*** =instructions

Along with `$@`,
the shell lets you use `$1`, `$2`, and so on to refer to specific command-line parameters.
Use this to write a script called `column.sh` that selects a single column from a CSV file
when the user provides the filename as the first parameter and the column as the second:

```{shell}
bash column.sh seasonal/autumn.dat 1
```

*** =hint

*** =pre_exercise_code
```{shell}

```

*** =sample_code
```{shell}

```

*** =solution
```{shell}
cut -d , -f $2 $1
```

*** =sct
```{shell}

```

--- type:PlainMultipleChoiceExercise lang:shell xp:50 skills:1 key:f7f6b78f25
## File details

FIXME: modify this to discuss only file lengths and modification dates - move the permissions and owners to the next exercise.

Unix keeps track of what it is and isn't allowed to do with particular files and directories
by storing a set of *permissions* for each one.
The three basic permissions are read, write, and execute,
which (as their names suggest) mean "read the contents",
"modify the contents",
and "run the contents".
Each permission is stored independently:
you can,
for example,
have read and execute for a file but not write,
or read and write but not execute,
and so on.
These permissions are often written `rwx`,
with dashes for permissions that are missing,
so the two examples just given would be `r-x` (read and execute)
and `rw-` (read and write).

Since Unix was designed as a multi-user operating system,
it actually stores three sets of permissions for each file or directory.
One set specifies what the owner of that file or directory can do;
the second specifies what people in the owner's group can do,
and the third specifies what everyone else is allowed to do.
To see something's permissions,
use `ls` with the `-l` flag (meaning "long form").
This option also displays how big things are (in bytes) and when they were last modified.
For example,
`ls -l` in the `seasonal` directory displays something like:

```
-rw-r--r--  1 repl  staff  399 18 Aug 09:27 autumn.csv
-rw-r--r--  1 repl  staff  458 18 Aug 09:27 spring.csv
-rw-r--r--  1 repl  staff  479 18 Aug 09:27 summer.csv
-rw-r--r--  1 repl  staff  497 18 Aug 09:27 winter.csv
```

which shows that the files are owned by a user named `repl` in a group named `staff`,
that they range in size from 399 to 497 bytes,
that they were last modified on August 18 at 9:27 in the morning,
and that each file can be read and written by their owner (the first `rw-`),
read by other people in the `staff` group (`r--`),
and also read by everyone else on the machine (`r--`).

*** =instructions

*** =hint

*** =pre_exercise_code
```{shell}

```

*** =sct
```{shell}

```


--- type:MultipleChoiceExercise lang:shell xp:50 skills:1 key:7c73b6e18e
## File permissions

FIXME: concentrate on the owner/group and the `rwx` bits.


*** =instructions

*** =hint

*** =pre_exercise_code
```{shell}

```

*** =sct
```{shell}

```
--- type:NormalExercise lang:shell xp:100 skills:1 key:b1d307aae6
## Making scripts runnable

You can always run a script by typing `bash script.sh`,
but it's easier on the eyes and fingers to run it directly by typing `script.sh`.
If you try doing this now,
though,
the shell will print an error message
because you haven't told it that it's allowed to run the contents of that file.

The solution is to change the permissions of the file using `chmod`
(which stands for "change mode").
The first parameter to `chmod` describes what permissions you want the file to have;
all the other parameters should be the names of the files whose permissions you are changing.

To describe permissions,
you write an expression like `u=rw` or `g=rwx`.
The first letter is one of "u", "g", or "o",
meaning "user" (you),
"group" (other people in your group),
or "other" (for everyone else).
The letters after the equals sign specify the permissions you want to give the file.
Thus,
to make your `dates.sh` script runnable,
and to prevent yourself from accidentally overwriting its contents,
you would write:

```{shell}
chmod u=rx dates.sh
```

You can also add or subtract permissions by using "+" or "-" instead of "=".

Unix's model of permissions dates from an earlier and more innocent time.
Today,
you will usually not care about group permissions unless you are working on a cluster
(since you're probably the only person working on your laptop).
You should probably also not change the permissions of directories without first doing a bit of reading:
Unix is consistent in how it treats these,
but not entirely intuitive…

*** =instructions

The directory `scripts` contains two shell scripts called `dates.sh` and `teeth.sh`.
Using a single command,
set their permissions so that everyone can read and execute them directly
(i.e., without typing `bash` in front of their names).
Run each script in order on all of the files in the `seasonal` directory
to test that your changes have worked.

*** =hint

*** =pre_exercise_code
```{shell}

```

*** =sample_code
```{shell}

```

*** =solution
```{shell}
chmod o=rx scripts/*.sh
scripts/dates.sh seasonal/*.csv
scripts/teeth.sh seasonal/*.csv
```

*** =sct
```{shell}

```