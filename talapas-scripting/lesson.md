---
title: Lesson
layout: page
permalink: /talapas-scripting/lesson.html
parent: Bash Scripting on Talapas
nav_enabled: true
nav_order: 3
---
# Day Two: Bash Scripting on Talapas
{: .no_toc }
This lesson is adapted from *[The Unix Shell](https://swcarpentry.github.io/shell-novice/aio.html)* lesson of Software Carpentry and [].
- TOC
{:toc}

## The Talapas Shell
You should now be inside your home directory on Talapas. Just like on your local filesystem, the default working directory on login is your home directory.

```bash
pwd
```

```output
/home/emwin
```

Note that just this working directory path is not enough to tell you *where* are you really are. 
As a device keyed to the same DuckID, my work laptop has an `emwin` home directory too, but it's a completely separate filesystem.

Talapas's network filesystem is physically located in the Millrace building by the river. Though it might be backed up to a cloud service provider like OneDrive, your laptop's filesystem is stored on a disk drive inside your laptop. 

We're going to talk about the process of transferring files to and from the cluster in detail soon, but the aspect of filesystem separation is worth considering from the outset. 

**Before running computationally intensive code check if are you running locally, on a login node, or on an interactive node.**

## Shell Access: Login Nodes
Whether you choose to access Talapas through the web terminal or through the command line, you will start at a **login node**. 

The **login node** is a place to prepare input data, organizes, edit code, and configure environment object but it's **not** the appropriate node for executing it. 

Any script that takes longer than a second or two should be run on a **compute node** instead.

## Who and Where Am I: `hostname`, `whoami`, `who`
To find out the name of the computer (or node) you are connected to, use the `hostname` command.

```bash
hostname
```

```ouput
login1.talapas.uoregon.edu
```

As expected, we are on one of the login nodes. There are four login or "head" nodes for Talapas number `login1`, `login2`, `login3`, and `login4` respectively.

To have a special load balancer decide which login node 
you should connect to, you can `ssh` to `login.talapas.uoregon.edu`
instead of selecting a particular node.

Let's try another command. Who are you? `whoami`
```bash
whoami
```
This should give you your DuckID.
```output
emwin
```
Talapas access is given through your **DuckID**. This is why, unlike
other workshops we hold on campus, you must be an active member of the 
UO community to participate.

Let's do a more interesting command: `who`. This command will let you see the DuckIDs of everyone connected to the current login node. 
Run it and you should see a list of the DuckIDs of your classmates 
in the room *and* the DuckIDs of other researchers connected to
the login node right now.

```bash
who
```

```output
emwin
...
```

If you see a name twice, someone has multiple open terminals
connected to Talapas right now. This is a common use case for
users who might need to manage code in one filesystem location
and inspect outputs in another.

## Unix Permissions

Now that we're on Talapas, let's do an ls command with the long-listing argument and a human readable flag. 

I want to take this opportunity to talk about UNIX permissions now that we're all on machines with the same operating system. 
Because software cannot manipulate files it cannot read, permissions are a common source of code failures on shared filesystems.

Though many of you come from different labs, we all share a *racs_training* PIRG. That means, everyone should have access to the racs_training folder located in `/projects/racs_training`.

```bash
cd /projects/racs_training
ls -lh
```

It's time to unpack results from the `-l` or long listing flag. Let's see if we can understand **what each field of a given row represents**,
working left to right.

1. **Permissions:** On the very left side, there is a string of the characters
   `d`, `r`, `w`, `x`, and `-`. The `d` indicates if something is a directory
   (there is a `-` in that spot if it is not a directory). The other `r`, `w`,
   `x` bits indicate permission to **R**ead, **W**rite, and e**X**ecute a file.
   There are three fields of `rwx` permissions following the spot for `d`. If a
   user is missing a permission to do something, it's indicated by a `-`.
   - The first set of `rwx` are the permissions that the owner has (in this
     case the owners are `wwinter`, `emwin`, ie. the workshop developers).
   - The second set of `rwx`s are permissions that other members of the owner's
     group share (in this case, the group is named `racs_training`).
   - The third set of `rwx`s are permissions that anyone else with access to
     this computer can do with a file. Though files are typically created with
     read permissions for everyone, typically the permissions on your home
     directory prevent others from being able to access the file in the first
     place.
2. **References:** This counts the number of references ([hard
   links](https://en.wikipedia.org/wiki/Hard_link)) to the item (file, folder,
   symbolic link or "shortcut").
3. **Owner:** This is the username of the user who owns the file. Their
   permissions are indicated in the first permissions field.
4. **Group:** This is the PIRG of the user who owns the file. Members of
   this user group have permissions indicated in the second permissions field.
5. **Size of item:** This is the number of bytes in a file, in human-readable form thanks to the -h flag.
6. **Time last modified:** This is the last time the file was modified.
7. **Filename:** Filename.

Group membership is very important on Talapas, because it is configured such that research groups cannot see each other's data. It's also configured such that your home directory's data is private to you.

For example, you can open files I've created within the racs_training project directory because this file has **read** permissions enabled for members of the racs_training PIRG.

```bash
cat test.txt
```

However, we'll get a permissions error if I try to read Gabriele's home directory.

```bash
ls /home/ghayden/
```

```output
ls: cannot open directory '/home/ghayden': Permission denied
```

Note that, even if there were files in Gabriele's home directory that *were* readable by anyone, I could not read them because I cannot traverse into her home directory.

To share a file or directory that you own with someone in your PIRG, you can grant read and execute privileges for them within the shared `/project/PIRG` directory. However, you must also set the same privileges on any parent directories above the item you're sharing.

The best place to share files among your labmates are folders that are shared by your lab, like a project directory.

### Important: How Do You "Execute" A Folder??
The execute bit for directories controls whether or not the folder can be **traversed**. Without execute permissions, you cannot see the child folders and files of folders you can otherwise read.

#### Quiz
On websites like StackOverflow, you will often see bad advice that recommends you use the command `chmod 777` or `chmod -R 777` to alter the permissions of shared input files.

```bash
touch the-universal-file.txt
chmod 777
```
What does this do? Why is this such a bad idea on shared filesystems?

#### Answer

Use `ls -lh` to examine how the permissions on the file have changed.
```bash
ls -lh the-universal-file.txt
```

```output
-rwxrwxrwx. 1 emwin uoregon 0 Feb  5 11:21 the-universal-file.txt
```

This is a bad idea because it grants read, write, and execute permissions to **all** users for the file. 
In the majority of cases, your problems can be solved just as easily by giving **group** read access to files and execute access on folders for your labmates' traversal. 

That said, people without access to your home directory can't traverse to it.

## Transferring Files to Talapas with `scp`

Speaking of separate filesystems, today's zip file isn't on Talapas.
Let's move it there with the `scp` command.

The `scp` command needs to be run from our local machine, so let's 
exit the login node briefly by typing `exit`.

```bash
exit
```

You should see a message like this after exiting.
```ouput
logout
Connection to login1.talapas.uoregon.edu closed.
```

You should be back on your local machine. Use `pwd` to check that you are in your home directory and `ls talapas-bash.zip` to make sure you have today's zip folder ready.

```bash
pwd
ls talapas-bash.zip
```

```output
talapas-bash.zip
```

Remember the `cp` command? With SSH access, you can use the extremely convenient `scp` or "secure copy" command to copy files/folders to 
and from Talapas. 

Just like the `cp` command, the arguments for `scp` are indicated in [source] [destination] order. 

To indicate a remote filesystem location, use the `[yourDuckID]@login1.talapas.uoregon.edu:` prefix. Every character after the : refers to the filesystem as appears on a Talapas login node.

```bash
scp talapas-bash.zip [yourDuckID]@login1.talapas.uoregon.edu:~
```

```output
talapas-bash.zip                              100% 7187KB   5.4MB/s   00:01
```

#### Quiz
How would I copy the `/project/racs_training/test.txt` file to my home directory on my laptop using `scp`?

**Hint**: Remember what the `.` character means?
#### Answer

```bash
scp [yourDuckID]@login1.talapas.uoregon.edu:/projects/racs_training/test.txt .
```

```output
test.txt                     00%   52     0.8KB/s  00:00
```
Check the file transfer worked by inspecting its contents in `nano`. Remember `nano`?

```bash
nano test.txt
```

Use <kbd>Ctrl</kdb>+<kbd>X</kdb> to exit Nano.

### Big Data?
Moving **really** big files and folders around? Consider [Globus](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756888/Globus).

Let's get to learning Bash! Let's ssh to Talapas again. Use the arrow keys to traverse your command history and grab the right command.

```bash
ssh [yourDuckID]@login1.talapas.uoregon.edu
```

Confirm your zip file safely arrived at its destination in your Talapas home directory with an `ls`.
```bash
ls talapas-bash.zip
```

```output
talapas-bash.zip
```

Now, let's unzip it from the command line using the `unzip` command. (This may take a minute.)

```bash
unzip talapas-bash.zip
```

When the file is unzipped, change your current working directory
to talpas-bash. 
The contents of the folder should be very familiar to you at this point

```bash
cd talapas-bash
ls
```

```output
books/  exercise-data/  scripts/
```

## Writing Output to Files: `>` and `>>`

Which of these protein database files contains the fewest lines?
It's an easy question to answer when there are only six files,
but what if there were 6000?
Our first step toward a solution is to run the command:

```bash
wc -l *.pdb > lengths.txt
```

The greater than symbol, `>`, tells the shell to **redirect** the command's output to a
file instead of printing it to the screen.

 This command prints no screen output, because
everything that `wc` would have printed has gone into the file `lengths.txt` instead.
If the file doesn't exist prior to issuing the command, the shell will create the file. If the file exists already, it will be silently overwritten.
Thus, **redirect** commands require caution.

`ls lengths.txt` confirms that the file exists:

```bash
ls lengths.txt
```

```output
lengths.txt
```

We can now send the content of `lengths.txt` to the screen using `cat lengths.txt`.

```bash
cat lengths.txt
```

```output
  20  cubane.pdb
  12  ethane.pdb
   9  methane.pdb
  30  octane.pdb
  21  pentane.pdb
  15  propane.pdb
 107  total
```


We'll continue to use `cat` in this lesson, for convenience and consistency, but it has the disadvantage that it always dumps the whole file onto your screen.

## Filtering Output with `sort`, `head`, and `tail`

Next we'll use the `sort` command to sort the contents of the `lengths.txt` file.

The file `talapas-bash/exercise-data/numbers.txt` contains the following lines:

```source
10
2
19
22
6
```

If we run `sort` on this file, the output is:

```output
10
19
2
22
6
```

If we run `sort -n` on the same file, we get this instead:

```output
2
6
10
19
22
```

This is because the `-n` option specifies a numerical rather than an alphanumerical sort.

The `sort` command alone does *not* change input files; it prints their lines in sorted order to the screen.

```bash
sort -n lengths.txt
```

```output
  9  methane.pdb
 12  ethane.pdb
 15  propane.pdb
 20  cubane.pdb
 21  pentane.pdb
 30  octane.pdb
107  total
```

We can put the sorted list of lines in another temporary file called `sorted-lengths.txt`
by putting `> sorted-lengths.txt` after the command,
just as we used `> lengths.txt` to put the output of `wc` into `lengths.txt`.


Once we've done that,
we can run another command called `head` to get the line of `sorted-lengths.txt`:

```bash
sort -n lengths.txt > sorted-lengths.txt
head -n 1 sorted-lengths.txt
```

```output
  9  methane.pdb
```

This tells us that `methane.pdb` is the shortest of the files, with only 9 lines.

Using `-n 1` with `head` tells it that we only want the first line of the file; `-n 20` would get the first 20, and so on.

### Warning: Redirecting to the same file

It's a very bad idea to try redirecting
the output of a command that operates on a file
to the same file. For example:

```bash
sort -n lengths.txt > lengths.txt
```

Doing something like this may give you
incorrect results and/or delete the contents of `lengths.txt`. *Do not actually run this command.*


### Peeking at the Bottom with `tail`

We have already met the `head` command, which prints lines from the start of a file. `tail` is similar, but prints it lines from the end of a file instead.

If we look at the last two lines of sorted-lengths.txt using `tail -n 2 sorted-lengths.txt`, we get the longest and teh total lengths of all the `.pdb` files instead.

```bash
tail -n 2 sorted-lengths.txt
```

```output
  30 octane.pdb
 107 total
```

## Pipelines: The Magic of `|`

In our example of finding the file with the fewest lines,
we are using two intermediate files `lengths.txt` and `sorted-lengths.txt` to store output.
This is a confusing way to work because
even once you understand what `wc`, `sort`, and `head` do,
those intermediate files make it hard to follow what's going on.
We can make it easier to understand by running `sort` and `head` together:

```bash
sort -n lengths.txt | head -n 1
```

```output
  9  methane.pdb
```

The vertical bar, `|`, between the two commands is called a **pipe**.
It tells the shell that we want to use
the output of the command on the left
as the input to the command on the right.

This has removed the need for the `sorted-lengths.txt` file.


Nothing prevents us from chaining pipes consecutively.
We can for example send the output of `wc` directly to `sort`,
and then send the resulting output to `head`.
This removes the need for any intermediate files.

We'll start by using a pipe to send the output of `wc` to `sort`:

```bash
wc -l *.pdb | sort -n
```

```output
   9 methane.pdb
  12 ethane.pdb
  15 propane.pdb
  20 cubane.pdb
  21 pentane.pdb
  30 octane.pdb
 107 total
```

We can then send that output through another pipe, to `head`, so that the full pipeline becomes:

```bash
wc -l *.pdb | sort -n | head -n 1
```

```output
   9  methane.pdb
```

The redirection and pipes used in the last few commands are illustrated below:

![pipes filters](../images/pipe-filters.JPG)


#### Quiz
In our current directory, we want to find the 3 files which have the least number of
lines. Which command listed below would work?

1. `wc -l * > sort -n > head -n 3`
2. `wc -l * | sort -n | head -n 1-3`
3. `wc -l * | head -n 3 | sort -n`
4. `wc -l * | sort -n | head -n 3`

#### Answer
Option 4 is the solution.
The pipe character `|` is used to connect the output from one command to
the input of another.
`>` is used to redirect standard output to a file.
Try it in the `talapas-bash/exercise-data/alkanes` directory!


### Putting it Together: Pipes and Filters

This idea of linking programs together is why Unix has been so successful.
Instead of creating enormous programs that try to do many different things,
Unix programmers focus on creating lots of simple tools that each do one job well,
and that work well with each other.
This programming model is called 'pipes and filters'.
We've already seen pipes;
a **filter** is a program like `wc` or `sort`
that transforms a stream of input into a stream of output.

Almost all of the standard Unix tools can work this way.
Unless told to do otherwise,
they read from standard input,
do something with what they've read,
and write to standard output.

The key is that any program that reads lines of text from standard input
and writes lines of text to standard output
can be combined with every other program that behaves this way as well.

Let's put some of these skills into practice.

Move into the `mice` directory.
```bash
cd ..
cd mice
```

We can pipe the results of the `ls` command to `wc -l` to count how many objects in the current working directory. 

```bash
ls | wc -l
```

```output
5
```

#### Quiz
> How could I modify this command to get the number of `.txt` files in the current directory?

#### Answer
>
```bash
ls *.txt | wc -l
```
```output
4
```

This folder contains data from a behavioral experiment on mice described in `citation.txt`. 

The files `Animals.txt`, `Tasks.txt` and `Visit.txt` contain tab-separated observations with the first line of each observation indicating the variables in each row. Let's see what each file tracks by using `head -n 1` on each of the three files.

```bash
head -n 1 Animals.txt Tasks.txt Visit.txt
```

```output
==> Animals.txt <==
Animal  Tag     Sex     Group   CornerAssigned

==> Tasks.txt <==
Date    Night   Task

==> Visit.txt <==
VisitID VisitOrder      Animal  Tag     Sex     Group   
Module  Cage    Corner     CornerCondition PlaceError      
SideErrors      TimeErrors      ConditionErrors    NosepokeNumber  
NosepokeDuration        LickNumber      LickDuration       
LickContactTime StartDate       StartTime       StartTimecode      
EndDate EndTime EndTimecode     VisitDuration   Session
```

### Fun with Tab-Separated Values: `cut`

The `cut` command is used to remove or ‘cut out’ certain sections of each line in the file, and cut expects the lines to be separated into columns by a <kbd>Tab</kbd> character, just like our mouse data.

Cut must be used in combination with a `-f` argument indicating the fields we want to extract. In the event your data *isn't* tab-separated, you can use `-d` to specify delimiters like commas.

Let's try it with `Animals.txt`.

```bash
cut -f 1 Animals.txt
```

```output
Animal
GK-1894
GK-1895
GK-1896
GK-1897
GK-1898
GK-1899
GK-1900
GK-1901
GK-1902
GK-1903
GK-1904
GK-1905
GK-1890
GK-1891
GK-1892
GK-1893
```

This gives us the first column of the tab-separated value file: namely, the unique identifiers for each mouse in the study.

#### Quiz
> Datasets are often keyed or indexed by the value in their first column. What if, instead of the entire first column, I wanted to know the name of the first variable in each of the three text files? As a hint, you'll need to combine the last two operations we discussed with a pipe.

```bash
head -n 1 Animals.txt Tasks.txt Visit.txt | cut -f 1
```

```output
==> Animals.txt <==
Animal

==> Tasks.txt <==
Date

==> Visit.txt <==
VisitID
```



### Scripts

We've been running increasingly complicated commands, and it's time to think about putting them into a script to make them reusable.


When you give batch jobs to Slurm on Talapas, you will have typically give them in the form of shell scripts, signified by the `.sh` file extension.

Let's use nano to write our first script.

```bash
nano first-script.sh`
```

```vim
#!/bin/bash
echo "Hello World"
```
As always, use <kbd>Ctrl</kbd>+<kbd>O</kbd> then **Enter** to
write out your text to `first-script.sh`. Then use <kbd>Ctrl</kbd>+<kbd>X</kbd> to exit `nano`.

The `#!/bin/bash` comment is important, it communicates which executable (which shell) should run this script. Normally `#` indicates a commented line in Bash and causes the line to be ignored during execution.

Now, if we try to run it like any old command, we'll get an error.

`first-script.sh`

We need to specify that bash should run this script.

`bash first-script.sh`

This mode of executing the script is much like running `python3 myfile.py` or `Rscript myscript.R`

If the script doesn't execute, make sure it has execute permissions. 

`chmod +x first-script.sh`
`./first-script.sh`

You should now see a "Hello World" message. 
Congratulations, you've written your first script!

Let's write a more complicated script that uses a powerful new programming construct: loops.

## Loops: Easy as `for`, `do`, `done`
**Loops** are a programming construct which allow us to repeat a command or set of commands
for each item in a list.
As such they are key to productivity improvements through automation.
Similar to wildcards and tab completion, using loops also reduces the
amount of typing required (and hence reduces the number of typing mistakes).

Let's navigate to our scripts directory to get an example of for-loop syntax in Bash. It may come in handy for your work if you're doing batch processing.

`cd scripts`

If you look inside, you'll see a script called `a-slow-script.sh`.

Use `cat` to print it to the terminal.

`cat a-slow-script`

Let's unpack it line-by-line!

It's a bash script, so it has `#!/bin/bash` line.


When the shell sees the keyword `for`,
it knows to repeat a command (or group of commands) once for each item in a list.
Each time the loop runs (called an iteration), an item in the list is assigned in sequence to
the **variable**, and the commands inside the loop are executed, before moving on to
the next item in the list.
Inside the loop,
we call for the variable's value by putting `$` in front of it.

The `$` tells the shell interpreter to treat
the variable as a variable name and substitute its value in its place,
rather than treat it as text or an external command.

It uses our first for-loop, which is indicated by `for`. The loop runs once for each possible value of the loop variable `i`. `i` will take on each value of the numbers that follow `in` once and in order. That is, it will take on the value of 1-15 inclusive, and then the loop will terminate at `done`.

For-loops are a popular *control flow* structure in Python, too.

This code is equivalent to this loop in Python.

```python
# Python
import time # Sleep is separate module in Python
for i in range(1, 16): #16 is not included
    print(i) # Print i
    sleep(1) # Wait for one second
```

And it's equivalent to this loop in R.

```R
# R
for (i in 1:15){ # 1-15 inclusive
  print(i) # Print i
  Sys.sleep(1) # Wait for one second
}
```

Check that you have permission to run the script.

`ls -lha`

`chmod +x a-slow-script`

`./a-slow-script`

This script will run for at least fifteen seconds before terminating on its own, which could be bothersome.

This brings me to **Ctrl+C**. Use it terminate running commands or programs that are running too slowly, caught in infinite loops, or are printing more to the terminal than you desire.

Cancel the script using **Ctrl+C** before it prints all fifteen numbers.


### Managing Command History: `history`
We've done a lot of typing so far. Conveniently, the shell tracks the commands you've typed. Don't count on this history to be there after a reboot though.

To see your command history, type `history`.

```bash
history
```

## Getting New Software on Talapas: You're Not `sudo`
While we've covered the Bash you need to manage files, folders, and configure jobs on Talapas, there's more Bash to explore. 

That said, Talapas is not the place to experiment with unfamiliar commands, or commands that require **root access**.

On a machine you own, you can install new command line software through commands that begin with `sudo` or use a GUI to install new software.

Talapas is firewalled and you are not, for obvious security reasons, an administrator or super user. 

You cannot install software yourself. If you're researching Bash commands, and you see
on that requires sudo, then that command will not work on Talapas.
Software versioning is another common issue. A team might depend on 