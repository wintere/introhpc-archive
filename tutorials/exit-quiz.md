---
title: Exit Quiz
layout: page
permalink: /tutorials/exit-quiz.html
nav_enabled: false
---

# Bash Self-Test for Talapas

The first day of this workshop is spent reviewing essential filesystem skills and Bash commands.
If you're already a command-line expert but new to Talapas, you can skip ahead to Thursday's lesson, which will
be conducted on Talapas. We'd be happy to have you either way!

If you've been using Talapas and Slurm for a while now, 
confirm you have access to the racs_training pirg using the `groups` command. 
You can choose to start next week or join us for a refresher on Bash.

This self-test will give you a chance to decide if and how much to skip ahead.

## Prerequisite: Confirm Talapas Access

Open a connection to a Talapas login node at \[duckID\]@login.talapas.uoregon.edu through either an SSH client (Terminal, Putty (Windows), Git Bash, Command Prompt) or [through the Shell](https://ondemand.talapas.uoregon.edu/pun/sys/shell/ssh/login1.talapas.uoregon.edu) application on OnDemand. 

**SSH Client Route:** ssh \[duckID\]@login.talapas.uoregon.edu 

* Enter your DuckID password when prompted

**Web Browser Route:**

* Navigate to [ondemand.talapas.uoregon.edu](https://ondemand.talapas.uoregon.edu/)    
* Log in via DuckID   
* Go to Clusters \-\> Talapas Shell Access   
* A new browser tab will open with a terminal 

**If you are not able to access Talapas, stop here. Please contact *@emwin* to make sure you 
are added to Talapas in time for Thursday's session.**

## Introduction to Bash - Oct. 14th

### General Knowledge
1. What is the difference between a relative path and an absolute path?
1. Why is knowledge of Bash required for using Talapas?

### Bash
1. Confirm that you are in your home directory on Talapas by printing your current working directory.   
1. To avoid modifying the source materials for future exercises, make a new folder inside your home directory called `IntroHPC2025`.
1. Copy the `/projects/racs_training/intro-hpc-f25/talapas-check/` folder (with its contents) to `IntroHPC2025`.
    - If you cannot access the `/projects/racs_training` directory, you are not a member of the racs_training PIRG. Please
    contact *@emwin* to make sure you are added before Thursday.
1. Change your working directory to the `IntroHPC2025` directory.  
1. Change your working directory to `talapas-check` subfolder.    
1. Make a new file in the command line text editor of your choice (nano, vim, emacs, etc.) named `README.txt` with the words “My new file” inside.   
1. Output (concatenate) the contents of your `README.txt` to the terminal.
1. The contents of Shakespeare's *The Tempest* are contained in `tempest.txt`. Use a Bash command to print the last
2 lines of the play.     

## Bash Scripting on Talapas - Oct. 16th

### General Knowledge:
1. Talapas has four login nodes. What Bash command could you use to determine which of the four login nodes you've connected to?
1. What is the difference between a login node and a compute node?

### Bash
1. Inspect the permissions of the contents of the `talapas-check` directory.  
1. Do you have permission to execute `a-little-script.sh`? How can you tell?   
1. Modify the `a-little-script.sh` to be executable    
1. Before running `a-little-script.sh`, print its contents to the terminal   
1. What does this first line of this script mean?   
1. Run `a-little-script.sh` (it’s not a real Slurm job or doing real work, so it’s fine to run on the login node) 
1. Using output redirection, find all the lines in `tempest.txt` that contain the word "queen". 

Want to check your work? Here's our [answer key](./exit-key.html).