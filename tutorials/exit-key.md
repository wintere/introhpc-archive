---
title: Exit Quiz Key
layout: page
permalink: /tutorials/exit-key.html
nav_enabled: false
---

## Introduction to Bash - Oct. 14th

### General Knowledge
1. What is the difference between a relative path and an absolute path?
1. Why is knowledge of Bash required for using Talapas?

### Bash
1. Confirm that you are in your home directory on Talapas by printing your current working directory.
- `pwd`
1. To avoid modifying the source materials for future exercises, make a new folder inside your home directory called `IntroHPC2025`.
- `mkdir IntroHPC2025`
1. Copy the `/projects/racs_training/intro-hpc-f25/talapas-check/` folder (with its contents) to `IntroHPC2025`.
- `cp -r /projects/racs_training/intro-hpc-f25/talapas-check IntroHPC2025/`
- If you cannot access the `/projects/racs_training` directory, you are not a member of the racs_training PIRG. Please
    contact *@emwin* to make sure you are added before Thursday.
1. Change your working directory to the `IntroHPC2025` directory.  
- `cd IntroHPC2025`
1. Change your working directory to `talapas-check` subfolder.
- `cd talapas-check`    
1. List the contents of the `talapas-check` subfolder. How many files are there?
- `ls`, 2
1. Make a new file in the command line text editor of your choice (nano, vim, emacs, etc.) named `README.txt` with the words “My new file” inside.   
- `nano README.txt` or `vim README.txt` etc.
1. Output (concatenate) the contents of your `README.txt` to the terminal.
- `cat README.txt`
1. The contents of Shakespeare's *The Tempest* are contained in `tempest.txt`. Use a Bash command to print the last
2 lines of the play.     
- `tail -n 2 tempest.txt`

## Bash Scripting on Talapas - Oct. 16th

### General Knowledge:
1. Talapas has four login nodes. What Bash command could you use to determine which of the four login nodes you've connected to?
`hostname`
1. What is the difference between a login node and a compute node?

### Bash
1. Inspect the permissions of the contents of the `talapas-check` directory.
`ls -lha`  
1. Do you have permission to execute `a-little-script.sh`? How can you tell?   
- No, you do not have execute permissions.
1. Modify the `a-little-script.sh` to be executable by you.
- `chmod u+x a-little-script.sh`
1. Before running `a-little-script.sh`, print its contents to the terminal   
- `cat a-little-script.sh`
1. What does this first line of this script mean?   
- It indicates this is a Bash script to be run by `/bin/bash`.
1. Run `a-little-script.sh` (it’s not a real Slurm job or doing real work, so it’s fine to run on the login node) 
- `./a-little-script.sh`
1. Using redirection, find all the lines in `tempest.txt` that contain the word "queen", then count them. 
- One approach is `cat tempest.txt | grep -i queen | wc -l`.
- Whichever approach you choose, you should find 11 lines that contain the word queen.