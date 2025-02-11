---
title: Bash Variables and Loops 
layout: page
permalink: /talapas-scripting/loops-variables.html
parent: Bash Scripting on Talapas
nav_enabled: true
nav_order: 5
---

# Bash Variables and Loops 

## Variables in Bash

The shell is just a program, and like other programs, it has variables.
Those variables control its execution,
so by changing their values
you can change how the shell and other programs behave.


Every variable has a name.
By convention, variables that are always present are given upper-case names.

Let's show the value of the variable `HOME`, or the path of *your* home directory:

```bash
echo HOME
```

```output
HOME
```

That just prints "HOME", which isn't what we wanted.
Let's try this instead:

```bash
echo $HOME
```

```output
/home/emwin
```

This behavior is different from languages like R and Python. The dollar sign tells the shell that we want the *value* of the variable rather than its name.

```bash
echo $HOSTNAME
```

```output
login1.talapas.uoregon.edu
```

To set our own variables for a given terminal session, we can use the following syntax:

```bash
MY_DOG='Cleo'
echo $MY_DOG
```

```output
CLEO
```

## Loops: Easy as `for`, `do`, `done`
**Loops** are a programming construct which allow us to repeat a command or set of commands
for each item in a list.
As such they are key to productivity improvements through automation.
Similar to wildcards and tab completion, using loops also reduces the
amount of typing required (and hence reduces the number of typing mistakes).

Let's navigate to our scripts directory to get an example of for-loop syntax in Bash. It may come in handy for your work if you're doing batch processing.

`cd ~/talapas-bash/scripts`

If you look inside, you'll see a script called `a-slow-script.sh`.

Use `cat` to print it to the terminal.

```bash 
cat a-slow-script.sh
```

```output
#!/bin/bash
for i in 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15; 
do
	echo $i # Print the current number
	sleep 1 # Wait one second before the next
done
```

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

```bash
ls -lha
```

```output
-rw-r--r--. 1 emwin uoregon 152 Feb  3 22:48 a-slow-script.sh
```

In this case I, the owner, do *not* have permission to run this file. I need to add **user** permissions.

```bash
chmod u+x a-slow-script
./a-slow-script
```

```output
1
2
3
```

This script will run for at least fifteen seconds before terminating on its own, which could be bothersome.

This brings me to **Ctrl+C**. Use it terminate running commands or programs that are running too slowly, caught in infinite loops, or are printing more to the terminal than you desire.

Cancel the script using **Ctrl+C** before it prints all fifteen numbers.
