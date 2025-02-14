---
title: Lesson
layout: page
permalink: /talapas-essentials/lesson.html
parent: Talapas Essentials
nav_enabled: true
nav_order: 2
---
# Talapas Essentials: The Structure of Talapas

## RACS
The Talapas cluster is managed by [Research Advanced Computing Services](https://racs.uoregon.edu/) or RACS. RACS administers the hardware, software, PIRGS, and other key services for Talapas. 

## RACS Resources:
* [Talapas Knowledge Base/Documentation](https://hpcrcf.atlassian.net/wiki/spaces/TW/overview)
* [How to Login to Talapas](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756485/How+to+Login+to+Talapas)
* [Talapas Quick Start Guide](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755757568/Quick+Start+Guide)
* [Talapas Customer Portal](https://hpcrcf.atlassian.net/servicedesk/customer/portal/1)

## A University Supercomputer

Talapas enables computation of programming tasks that could not be computed without more CPU cores, GPU cores, or memory than is available on consumer hardware.
It also accelerates research at UO by offloading
repetitive, highly parallel computation jobs from researchers' devices, allowing scientists to focus on more important tasks.

Talapas is a heterogenous cluster consisting of login nodes, compute nodes, and private "condo" nodes. These nodes are much more powerful than a personal computer! Each compute node can have up to 128 CPU cores. Some specialized nodes for large memory jobs have up to 4TB of RAM.

![talapas-hierarchy](../images/talapas-structure.png)

Talapas is a living, growing computational ecosystem. New software is added upon request, 
CPU and GPU hardware is periodically upgraded, and new computing nodes are added as research groups buy special nodes for their needs.

### Getting to Talapas: Login Nodes
Login nodes are shared by hundreds of Talapas users simultaneously.
They are intended for loading data, transferring large datasets from the internet or the cloud to the Talapas filesystem, preparing software environments, and connecting to IDEs.Unlike other nodes in the cluster, login nodes are open to connections from the broader internet.

The CPU cores and memory on login nodes are **not** for doing computational work.

[There's a detailed tutorial for connecting to login nodes here](../talapas-scripting/setup.html).

### What is `login.talapas.uoregon.edu`?

Talapas has a load balancer at `login.talapas.uoregon.edu`
that  distributes users as evenly as possible among
the four entry or "login" nodes. 

If you connect to `login.talapas.uoregon.edu` through your terminal, you will be routed to one of the four login nodes -- *login1*, *login2*, *login3*, or *login4* -- based on how many people are currently connected to each node.

## The Talapas Filesystem
Talapas uses a networked filesystem called GPFS to make code, input files, and other crucial pieces of data available across all nodes in the cluster.

All users have (at most) 256GB of storage available to them in their home directory at `/home/yourDuckID`.

You can check your home directory path by using the following Bash command.

```bash
echo $HOME
```

If you are on Talapas, you are also part of a PIRG. Your lab's data should live in `/projects/PIRG_NAME`.
PIRG project directories have a maximum of 2TB of storage.

Because the [file structure for PIRGs recently changed](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755758286/Directory+Structure), some PIRGS may have a slightly different stucture in their `/projects/PIRG_NAME` folder.

### New PIRGs
Joined Talapas recently? This implementation assumes all files and folders within a PIRG are shared among all members.
That means any files stored in `/projects/PIRG` are, by default, readable by members of that PIRG. 

### Sharing Files in Legacy PIRGs
Older PIRGS were created with the same storage limit but a *different permissions structure*.
This implementation had problems for collaboration among lab members, as labs have members that leave to work at other institutions. 

Each user had their own folder within their PIRG with files their labmates couldn't see at `/projects/PIRG/DuckID`
While all members could access data shared in `/projects/PIRG/shared` 

Shared data was reserved for the folder `/projects/PIRG/shared`, but it wasn't uncommon for lab members to want to share files and folders from their `/projects/PIRG/DuckID` folders with fellow lab members.

### I have a legacy PIRG. Can you fix our my group's permissions now that members have left?
Yes, to fix the file structure and permissions within your /projects/PIRG directory, **open a ticket with RACS**. They have special scripts that can help retrofit your PIRG into a flatter, easier file and permissions structure. 

Need to clean out a graduated members' data? Open a ticket with RACS.

### What happens when new members are added to legacy PIRGs?
You can choose to keep the existing file and permissions structure or have the new file permissions scheme implemented.

## Symlinks
If you do an `ll` or long-listing command on Talapas, you will see special folders in your home directory that begin with `@`.

```bash
ll
```

```output
...
lrwxrwxrwx. 1 root  root         26 May 28  2024 library_it -> /projects/library_it/emwin
lrwxrwxrwx. 1 root  root         23 Sep  5 16:22 racs_training -> /projects/racs_training
```

These folders aren't *actually* in your home directory. Symlinks or *symbolic links* are references between different locations in the filesystem. If you `cd` into the `/home/racs_training` symlink, you will be redirected or *linked* to `/projects/racs_training`.

These pointers are added for your convenience so that you can move files and code from your home directory to your project directory.

Some new PIRGs do not have this symlink in place. Do not worry, you can still access the same folder through `/projects/PIRG_NAME`. PIs can request that the symlink be added to their lab members' project directories.

## Which Folder Should I Use for What?
`/home/yourDuckID`
- code, testing instances, personal work

`/projects/yourPIRG` 
- datasets, project data, code you want to share with other members of your PIRG
 

## Transferring Files to Talapas
- To transfer files to Talapas from your web browser you can use the Talapas file browser. There's 10GB limit on what you can upload at a time/.
- To transfer files from the command line, use the `scp` command.
- To transfer datasets to Talapas from the internet, use the `wget http:/<filepath>` command on a **login node**. Remember, compute nodes are firewalled.
- For complex or large-scale transfers, try [Globus](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756888/Globus) or FTP tools like [Filezilla](https://filezilla-project.org/).

## Software: Modules on Talapas

Software on Talapas is controlled through [lmod](https://lmod.readthedocs.io/en/latest/) modules.

You don't need to worry too much lmod's implementation details to use modules, especially if the software you need is already available in the module catalogue.

To run software from within a Slurm job, you'll need to load the appopriate modules.

For example, let's load Python.
```bash
module load python3
```
<kbd>Tab</kdb> to autocomplete is available to you when searching through the list of modules.

To see the module you currently have loaded, run `module list`.
```bash
module list
```

```output
Currently Loaded Modules:
  1) miniconda-t2/20230523
  2) python3/3.10.13
```

Now that Python has been loaded, you can access it from your path at `python3`.
```bash
python3 --version
```

```output
Python 3.10.13
```

To remove a module, use the `module unload` command followed by the module name.

```bash
module unload python3/3.10.13
```

Or get rid of ALL modules with `module purge`.

```bash
module purge
```

Now, you can see that there are no modules loaded. 

```bash
module list
```

### Modules and PATH
The lmod system works by modifying your PATH variable.

The PATH variable defines the shell’s search path for executables: the list of directories that the shell looks in for runnable programs when you type in a program name without specifying what directory it is in. The rule it uses is simple: the shell checks each directory in the PATH variable in turn, looking for a program with the requested name in that directory. As soon as it finds a match, it stops searching and runs the program.

For example, loading Python3 added `/packages/miniconda-t2/20230523/envs/python-3.10.13` to my path.

```bash
module load python3/3.10.13
echo $PATH
```
```output
/packages/miniconda-t2/20230523/envs/python-3.10.13/bin:/packages/miniconda-t2/20230523/condabi...
```

### Browsing Modules
Want a more user friendly list of modules available?

Try `module spider [keyword]`. Below, I'll search for the neuroscience related software, FSL.

```bash
module spider fsl
```

```output
-----------------------------------------------------------------------------------------------------------------
  fsl:
-----------------------------------------------------------------------------------------------------------------
     Versions:
        fsl/5.0.9
        fsl/5.0.10
        fsl/6.0.1
        fsl/6.0.7
        fsl/6.0.7.9
     Other possible modules matches:
        FSL  FSLeyes  fsleyes  fslpy

-----------------------------------------------------------------------------------------------------------------
  To find other possible module matches execute:

      $ module -r spider '.*fsl.*'

```

### Reproducibility with Modules
Always use complete module names in your batch jobs and scripts when possible.

`module load fsl/6.0.7.9` is preferred to `module load fsl` because
the default version will change over time. The default version could be
out of date.

You should always know which packages and which versions your code relies upon, as it will make it easier for other scientists to **reproduce** your code.

## Talapas Partitions: Where Do I Run My Jobs

Here is a summary of the primary partitions on Talapas so you can decide where to schedule your jobs at a glance.

| Partition | Max Job Time | GPUs | Description |
| ---------- | ----- | --- | --------------------------------- |
| compute | 24 hrs | no | default partition, appropriate for most users |
| compute_intel | 24 hrs | no | for software that requires *Intel* processors, older computers |
| computelong | 2 wks | no | default partition for jobs that take longer than 24 hours |
| gpu | 24 hrs | yes | for shorter jobs that requires GPUs |
| gpulong | 2 wks | yes | partition for GPU jobs that take longer than 24 hours |
| interactive | 12 hrs | no | partition for interactive `srun` jobs, OnDemand apps Talapas Desktop, JupyterLab |
| interactivegpu | 8 hrs | yes | GPU partition for interactive `srun` jobs, OnDemand apps Talapas Desktop, JupyterLab |
| memory | 24 hrs | no | for memory-intensive jobs that require up to 4TB of RAM |
| memorylong | 2 wks | no | for memory-intensive jobs that require up to 4TB of RAM of a long duration |
| *preempt* | 1 wk | yes | [special "partition" that appropriates nodes in other partitions](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756724/How-to+Use+the+preempt+Partition) |

### Partition Status: `sinfo`

Want to know the current status and time limits of *all* the partitions on Talapas? The command `sinfo` displays all available partitions that **you** can schedule jobs on. (That means condo owners will see a slightly different list.)

```bash
sinfo
```

Each partition has one line for each state in order to list the number of nodes in each of the following states: mix, alloc, and idle.

```output
PARTITION         AVAIL  TIMELIMIT  NODES  STATE NODELIST
compute              up 1-00:00:00      2   plnd n[0116-0117]
compute              up 1-00:00:00     18    mix n[0115,0121,0135,0180-0183,0186-0196]
compute              up 1-00:00:00     21  alloc n[0111-0114,0119-0120,0122-0134,0184-0185]
compute              up 1-00:00:00      1   idle n0118
...
```
You can interpret the results from `sinfo` as follows.

* The **AVAIL** column represents the status of partition.
* The **TIMELIMIT** column represents max job time in days. `1-00:00:00` is 24 hours.
* The **NODES** column indicates how many nodes are in the partition have a given state.
* The **STATE** column lists the node states, ie. are the nodes allocated or idle.
* The **NODELIST** column lists the node in each of the possible states within a partition. 

To learn more, see the [Slurm documentation](https://slurm.schedmd.com/sinfo.html) for `sinfo`.

## Scheduling Batch Jobs with Slurm

To practice with Slurm tasks, logon to a Talapas login node. For this exercise, feel free to use the [Talapas OnDemand shell](https://ondemand.talapas.uoregon.edu/pun/sys/shell/ssh/login1.talapas.uoregon.edu).

Once you're on Talapas, copy this folder of example jobs to your home directory as follows.
```bash
 cp -r /projects/racs_training/resources/slurm_examples/ .
 ```

Open the job in a text editor of your choice. I have reproduced the contents below.
```bash
 nano hello.sbatch
```

```bash
#!/bin/bash

#SBATCH --partition=compute              ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=hello_world_python    ### Job Name
#SBATCH --output=%x-%j.out               ### File in which to store job output
#SBATCH --error=%x-%j.err                ### File in which to store job error messages

#SBATCH --time=0-00:05:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                        ### Number of nodes needed for the job
#SBATCH --mem=500M                       ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task

### Load needed modules
module purge
module load python3/3.11.4
module list

### Run your actual program
python3 hello.py
```
Let's parse this file using the comments as a guide! To learn more, see the [Slurm documentation](https://slurm.schedmd.com/sbatch.html) for `sbatch`.

* This job runs for a maximum of five minutes on one node and one CPU core of that node.
* It request 500MB of RAM.
* The job output will be written `hello_world_python-[jobid].out` and `hello_world_python-[jobid].err` respectively.
* It loads the `python3/3.11.4` module.
* It lists the current modules loaded and prints it to standard output.
* It runs a file in the `slurm_examples` directory called `hello.py` that prints to standard output. 

To submit your job file, run the `sbatch` command.

```bash
sbatch hello.sbatch
```

Look at your job history with `sacct`. Uh oh, there's an error.

Check the error log using `cat`.

```bash
cat hello_world_python-[YOURJOBID].err
```

```output
Traceback (most recent call last):
  File "/gpfs/home/emwin/slurm_examples/hello.py", line 4, in <module>
    print(dictionary['b']) # This should raise a KeyError
          ~~~~~~~~~~^^^^^
KeyError: 'b'
```

This is an invalid lookup error, a common mistake when using Python dictionaries. 
Check the original Python file in a command line text editor like `nano`.

```bash
nano hello.py
```

```python
print("Hello Wold !!")

dictionary = {'a': 1}
print(dictionary['b']) # This should raise a KeyError
```

For those of you who don't Python, this can be fixed by checking the dictionary for a valid entry: `a`.
Change the last line to the following in `nano`, then use <kbd>Ctrl</kbd>+<kbd>O</kbd> and <kbd>Ctrl</kbd>+<kbd>X</kbd> 
to modify the file.

```python
print(dictionary['a']) # This will not raise a KeyError
```

Now that your file is saved, rerun your fixed job.

```bash
sbatch hello.sbatch
```

Running `sacct` will show your job completed successfully.

```bash
sacct
```

```output
31538218     hello_wor+    compute racs_trai+          1  COMPLETED      0:0
31538218.ba+      batch            racs_trai+          1  COMPLETED      0:0
31538218.ex+     extern            racs_trai+          1  COMPLETED      0:0
```

Check the output log to see the results of your job. You'll want to pick the *second* and later run's log. Note that because each log is named for the jobid, and jobids are unique, your logs will not overwrite each other.

```bash
cat hello_world_python-[yourJOBID].out
```

To check the status of queued jobs, use the `squeue` command. Do not use squeue without an argument unless you want to see all the queued jobs.

```bash
squeue -u [yourDuckID]
```

To cancel a job, use the `scancel` command followed by the jobid of the job you want to cancel.

### Scheduling Tips
* Read the resource descriptions carefully: jobs that exceed their request time, memory, or processor usage will be automatically killed.
* Need to run for more than 24 hours? You must use `computelong`, `gpulong`, or `memorylong`.
* Most jobs should have `--ntasks-per-node=1` and `--cpus-per-task=1` if they do not reference tools or code that is explicitly multiprocessor code.

### Optimization with `seff`

Want to know how much of the resources you requested a finished job used? Try the `seff` command.

```bash
seff 31471361
```

```output
Job ID: 31471361
Cluster: talapas
User/Group: emwin/uoregon
State: CANCELLED (exit code 0)
Cores: 1
CPU Utilized: 00:00:00
CPU Efficiency: 0.00% of 00:02:48 core-walltime
Job Wall-clock time: 00:02:48
Memory Utilized: 456.00 KB
Memory Efficiency: 0.09% of 500.00 MB
```
Adjust your resource requests accordingly after your job runs. Be courteous and jobs won't be deprioritized. 
Remember that your job's runtime is its **queue time + execution time**, so asking for too much can make your job run *slower*.

### Helper Scripts from RACS

RACS has a few helper scripts for looking at which hardware features are available for specific nodes on Talapas.

This script shows which nodes have which models of GPU.
```bash
/packages/racs/bin/slurm-show-gpus`
```

```output
n0110 gpu:nvidia_l40:2(S:0)
n0149 gpu:nvidia_a100_80gb_pcie_1g.10gb:28(S:1)
n0150 gpu:nvidia_a100_80gb_pcie_3g.40gb:4(S:1)
n0151 gpu:nvidia_a100_80gb_pcie_1g.10gb:21(S:1)
...
```

This script lists the *processor* for each node.
```bash
/packages/racs/bin/slurm-show-features`
```

```output
n0029 intel,broadwell,e5-2690
n0030 intel,broadwell,e5-2690
n0031 intel,broadwell,e5-2690
n0032 intel,broadwell,e5-2690
...
```

### GPU Jobs
Let's examine an example GPU job in detail: `gpu.sbatch`.

Only three partitions are compatible with GPUs:
* gpu
* gpulong
* interactivegpu

Each node in a GPU partition has **at most** 4 GPUS.

```bash
#!/bin/bash

#SBATCH --partition=gpu                  ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=gpu_hello_world       ### Job Name
#SBATCH --output=%x-%j.out               ### File in which to store job output
#SBATCH --error=%x-%j.err                ### File in which to store job error messages

#SBATCH --time=0-00:05:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                        ### Number of nodes needed for the job
#SBATCH --mem=500M                       ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --gpus=1                         ### Number of GPUs to request 
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task
#SBATCH --mail-type=BEGIN,END
#SBATCH --mail-user=YOURDUCKID@uoregon.edu     

### Load needed modules
module purge
module load cuda/12.4.1
module list

### Run your actual program
nvidia-smi
```

* You request a gpu with the --gpus argument. It's not enough to go run on a GPU partition.
* If you need to select certain nodes within a partition, the `--constraint=` flag, can allow you to specify certain nodes based on job constraints.
* Every GPU requires a CPU too!

### Bonus: Email Me My Job Status
Running a really long job? Tired of checking `squeue`?

SLURM can even email you when jobs reach certain states:

```bash
### #SBATCH --mail-type=BEGIN,END,FAIL      
### #SBATCH --mail-user=<duckID>@uoregon.edu
```

Add the lines above to your Sbatch script with *your* DuckID. You should get an email from Slurm when the job begins and finishes (whether in success or failure.)

## Shared Resource Etiquette
- Close out your jobs when you're done!
- Book your interactive jobs for as long as you need, but not longer.
- You will not be warned time is about to run out. Track your own time conscientiously. 