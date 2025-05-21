---
title: Lesson
layout: page
permalink: /slurm/lesson.html
parent: Slurm on Talapas
nav_enabled: true
nav_order: 2
---

# Slurm on Talapas

[Slurm](https://slurm.schedmd.com/slurm.html) is the job scheduling software used on the Talapas. While Talapas has scheduling policies, partitions, and PIRGs that are specific to UO, 
Slurm is used for job scheduling on high-performance computing clusters around the world.

## Lesson Setup
For this lesson, you will need to connect to a Talapas login node through a shell application of your choice.

For convenience, we recommend the [Talapas OnDemand shell](https://ondemand.talapas.uoregon.edu/pun/sys/shell/ssh/login1.talapas.uoregon.edu).

To start, make sure you are in your home directory.

```bash
cd ~
```


Copy today's exercises from `/projects/racs_training/intro-hpc-s25/slurm/part2/` into your home directory. Don't forget `-r` to recursively copy the contents!

```bash
cp -r /projects/racs_training/intro-hpc-s25/slurm/part2 .
```

Navigate inside the `part2` folder using `cd`.

```bash
cd part2
```

Check that you see the following files inside with `ls`.

```bash
ls -F
```

```output
array.sbatch*  books/  books.sbatch*  hello.sbatch*  parallel_steps.sbatch*  random.py*  serial_steps.sbatch*  steps.sh*
```

## Batch Scheduling with `sbatch`

Batch scripts in Slurm are configured through special comments prefixed with `#SBATCH`. 

All batch jobs should have `#!/bin/bash` on the first line followed by `#SBATCH` options in any order. 
It doesn't matter what order you specify your `#SBATCH` options in as long you specify them one per line.

```bash
#!/bin/bash
#SBATCH --partition=compute
#SBATCH --account=racs_training
```
This set of comments represent the **minimum** required options for a Slurm job of Talapas:
* a valid Talapas partition
* an account (PIRG)

Before we look at the more complicated Slurm jobs, let's create a blank file representing a minimum viable Slurm job called `hello.sbatch` using nano.

```bash
nano hello.sbatch
```

Inside `nano`, enter the following lines. When you're finished, use <kbd>Ctrl</kbd>+<kbd>O</kbd>
and <kbd>Ctrl</kbd>+<kbd>X</kbd> to write out to the `hello.sbatch` file and then exit `nano`.

```bash
#!/bin/bash
#SBATCH --partition=compute
#SBATCH --account=racs_training
echo "Hello!"
```

## Required Slurm Job Elements
* `#!/bin/bash` on the first line
* `--partition=[a valid partition]` 
* `--account==[your PIRG]`

All other parameters like `mem`, `ntasks`, and `--cpus-per-task` have default values by partition.
The default value for job memory as configured through the `--mem-per-cpu` is 4GB per CPU.
For single core jobs, that means *single-core jobs start with 4GB RAM unless you specify otherwise*.

### Read More About Sbatch
The [Slurm documentation for `sbatch`](https://slurm.schedmd.com/sbatch.html) has an exhaustive list of all possible configuration options.

## A Minimum Viable Slurm Job
Let's run our minimum viable job by passing it to the `sbatch` command.

```bash
sbatch hello.sbatch
```

You will get a response with a (unique) job number when your job is submitted successfully.

```output
Submitted batch job 31700147
```

Check your job's status in the queue using the `squeue` command. The `--me` flag is a
helpful trick if you don't want to type `-u [yourDuckID]` each time.

```bash
squeue --me
```

With a job this simple, it's probably already finished.

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

If you see an empty queue like this, go ahead and check your most recent *finished* jobs with `sacct`.

```bash
sacct
```

This job doesn't have a specified output and error log file name, so it uses the slurm defaults: `slurm-[jobid].out`.
Doing an ls, we see a file that was created with the default parameters at `slurm-31700147.out`.
You can see where a `#SBATCH --job-name` might be more helpful in the debugging process.

```bash
ls
```

```output
array.sbatch  books.sbatch  parallel_steps.sbatch  serial_steps.sbatch  test.sh
books         hello.sbatch  random_test.py         slurm-31700147.out
```

Let's check the contents of the output log.
If it worked as intended, we should the results of the `echo` command from `hello.sbatch`.

```bash
cat slurm-31700147.out
```

```output
Hello!
```

Let's check the resource usage on this job with `seff`.

```bash
seff 31700147
```

```output
Job ID: 31700147
Cluster: talapas
User/Group: emwin/uoregon
State: COMPLETED (exit code 0)
Cores: 1
CPU Utilized: 00:00:00
CPU Efficiency: 0.00% of 00:00:00 core-walltime
Job Wall-clock time: 00:00:00
Memory Utilized: 0.00 MB (estimated maximum)
Memory Efficiency: 0.00% of 4.00 GB (4.00 GB/core)
```

Unsurprisingly, `hello.sbatch` didn't use many resources in performing the `echo` command.
You can also that 4GB of RAM was allocated to the job because Slurm used the default parameters.

## Adding Steps to Jobs with `srun`

Let's make `hello.sbatch` more interesting.

Open `hello.sbatch` in `nano` and modify it as follows.

```bash
nano hello.sbatch
```

```bash
#!/bin/bash
#SBATCH --mem-per-cpu=1G
#SBATCH --partition=compute
#SBATCH --account=racs_training

# Slurm treats srun as a job step
# Allows for more granular usage figures as the job is running
srun echo "Hello!"
```

Save your changes and submit the new version of this job using `sbatch`.

```bash
Submitted batch job 31638190
```

If we run `sacct`, you will see 4 entries rather than the usual 3. What happened here?

```bash
sacct
```

```output
31638190     hello.sba+    compute racs_trai+          1  COMPLETED      0:0 
31638190.ba+      batch            racs_trai+          1  COMPLETED      0:0 
31638190.ex+     extern            racs_trai+          1  COMPLETED      0:0 
31638190.0         echo            racs_trai+          1  COMPLETED      0:0 
```

Where did this mysterious 4th entry come from?
The line named `echo` indicates the task from the `srun` statement.
This is because the `srun` denotes a step within a job.
A job with multiple `srun` commands will create
a separate step (with the same jobid) in the `sacct`
for each step.

This feature is useful because it allows you to monitor the time, resources
spent on each step of a complicated batch job.

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
/packages/racs/bin/slurm-show-gpus
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

## Serial Processing: One Core and One Task at A Time
Do an `ls` to see the other job files available to you.

Look next at the `serial_steps.sbatch` job in `nano`.

`nano serial_steps.sbatch`

```bash
#SBATCH --partition=compute              ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=serial_steps          ### Job Name
#SBATCH --output=%x.out                  ### File in which to store job output
#SBATCH --error=%x.err                   ### File in which to store job error messages

#SBATCH --time=0-00:25:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --ntasks=1                       ### Number of tasks included in the job
#SBATCH --mem-per-cpu=50M                ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task


for i in {1..10}; do
    srun test.sh $i
done
```

This job runs a Bash for-loop. To learn more about Bash loops, [check out this quick tutorial](../talapas-scripting/loops-variables.html).

This script uses `srun` to create 10 sequential (serial) steps in one job. By default, each iteration (here `i = 1, 2, 3 ... 10`) must finish before the loop moves to the next.

```bash
sbatch serial_steps.sbatch
```

```output
Submitted batch job 31638207
```

Check the queue for the status of your jobs.
```bash
squeue --me
```

Check each step of serial steps in the sacct tab.

```bash
sacct
```

```output
31638207.0      test.sh            racs_trai+          1  COMPLETED      0:0 
31638207.1      test.sh            racs_trai+          1  COMPLETED      0:0 
31638207.2      test.sh            racs_trai+          1  COMPLETED      0:0 
31638207.3      test.sh            racs_trai+          1  COMPLETED      0:0 
31638207.4      test.sh            racs_trai+          1  COMPLETED      0:0 
31638207.5      test.sh            racs_trai+          1  COMPLETED      0:0 
31638207.6      test.sh            racs_trai+          1  COMPLETED      0:0 
31638207.7      test.sh            racs_trai+          1  COMPLETED      0:0 
31638207.8      test.sh            racs_trai+          1    RUNNING      0:0
...
```

This job is on loop 8 of the serial steps.
31638207.9 can't begin until 31638207.8 finishes.
**Serial jobs requires that steps be completed one at a time 
AND in a specific order.**

Let's look at the time spent on each stage in more detail using a specially configured `sacct` command. You can learn more about `sacct` [formatting on the Slurm documentation](https://slurm.schedmd.com/sacct.html).

```bash
sacct -u $(whoami) --units=G --format=JobID,jobname,MaxRSS,Start,Elapsed,State,ExitCode
```

```output
31638207.0      test.sh          0 2025-02-18T14:23:39   00:00:10  COMPLETED      0:0 
31638207.1      test.sh          0 2025-02-18T14:23:49   00:00:10  COMPLETED      0:0 
31638207.2      test.sh          0 2025-02-18T14:23:59   00:00:10  COMPLETED      0:0 
31638207.3      test.sh          0 2025-02-18T14:24:09   00:00:11  COMPLETED      0:0 
31638207.4      test.sh          0 2025-02-18T14:24:20   00:00:10  COMPLETED      0:0 
31638207.5      test.sh          0 2025-02-18T14:24:30   00:00:10  COMPLETED      0:0 
31638207.6      test.sh          0 2025-02-18T14:24:40   00:00:10  COMPLETED      0:0 
31638207.7      test.sh          0 2025-02-18T14:24:50   00:00:10  COMPLETED      0:0 
31638207.8      test.sh          0 2025-02-18T14:25:00   00:00:10  COMPLETED      0:0 
31638207.9      test.sh          0 2025-02-18T14:25:10   00:00:10  COMPLETED      0:0 
```

There are some jobs where certain steps or subcomponents must be completed in a serial fashion.
However, serial jobs do not take advantage of the computational power of Talapas or its support for parallelism.

How can make this job run in parallel? Will it be faster?

## Parallelism with `sbatch`

Let's examine `parallel-steps.sbatch`!

```bash
cat parallel-steps.sbatch
```

It looks very similar to `serial-steps.sbatch` but there are a few subtle and important differences.

```bash
#!/bin/bash

#SBATCH --partition=compute              ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=parallel_steps        ### Job Name
#SBATCH --output=%x.out                  ### File in which to store job output
#SBATCH --error=%x.err                   ### File in which to store job error messages

#SBATCH --time=0-00:25:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --ntasks=10                      ### Number of tasks included in the job
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task
#SBATCH --mem-per-cpu=50M                ### Number of cpus/cores to be launched per Task

# Allocate one task for each step
# Each task has one CPU

for i in {1..10}; do
    srun --ntasks=1 test.sh $i &
done
wait
```

This job requests 10 CPUS, allocates 1 CPU per task, and then
launches 10 concurrent `srun` subjobs.

The `&` at the end of `srun --ntasks=1 test.sh $i &` is crucial!
The ampersand tells Bash to *run this command in the background*. Without the ampersand
this loop would wait for one iteration to finish before moving to the next one.

```bash
sbatch parallel_steps.sbatch`
```

If you look at the output log, you'll notice something important.

The job steps **don't** complete in order!

```bash
cat parallel_steps.out
```

```output
I am printing this from job step 9
I am printing this from job step 2
I am printing this from job step 1
I am printing this from job step 8
I am printing this from job step 3
I am printing this from job step 7
I am printing this from job step 5
I am printing this from job step 6
I am printing this from job step 10
I am printing this from job step 4
```
The steps in the parallel job **did not run in order** or **finish in order**. You lose that guarantee in a parallel context.

How much faster did the parallel job run?

The serial job ran in 70 seconds.
Use `sacct` to see how much faster the parallel job ran.

```bash
sacct
```

```output
31638231     parallel_+            2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.ba+      batch          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.ex+     extern          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.0      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.1      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.2      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.3      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.4      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.5      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.6      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.7      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.8      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
31638231.9      test.sh          0 2025-02-18T14:30:36   00:00:10  COMPLETED      0:0 
```

The parallel job ran in 11 seconds.

This is an example of an *embarassingly parallel* job.
Each job is completely independent of another.

But what if each step in your pipeline is identical? For example, imagine applying the same processing pipeline over and over to thousands of input files.

## Array Jobs

Slurm has a special option for bulk scheduling of nearly identical jobs: [array jobs](https://slurm.schedmd.com/job_array.html).

Take a close look at `array.sbatch` with `cat`.

```bash
cat array.sbatch
```

```bash
#!/bin/bash
#SBATCH --partition=compute              ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=python_array          ### Job Name
#SBATCH --output=%x-%A-%a.out            ### File in which to store job output
#SBATCH --error=%x-%A-%a.err             ### File in which to store job error messages

#SBATCH --time=0-00:05:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                        ### Number of nodes needed for the job
#SBATCH --mem=50M                        ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task

#SBATCH --array=1-10

### Load needed modules
module purge
module load python3/3.11.4

### Run your actual program
srun python3 random_test.py $SLURM_ARRAY_TASK_ID $SLURM_ARRAY_JOB_ID $SLURM_JOB_ID
```
Array jobs require the `#SBATCH --array=` parameter.
In this example, the array parameter is `{0, 1, 2, ... 10}`.
The `%a` stands for the id of the *task* and the `%A` stands for the main array.

This job runs random_test.py and passes in three arguments: the $SLURM_ARRAY_TASK_ID, the $SLURM_ARRAY_JOB_ID and $SLURM_JOB_ID. `$SLURM_ARRAY_TASK_ID $SLURM_ARRAY_JOB_ID $SLURM_JOB_ID` are environment variables.

To learn more about Bash (and Slurm) environment variables, see [this bonus lesson](https://carpentries-incubator.github.io/hpc-intro/14-environment-variables/index.html).


Let's look in more detail at `random_test.py`.

```bash
cat random_test.py
```

```python
#!/usr/bin/env python3

import random
import sys

if __name__ == "__main__":
    args = sys.argv[1:]
    seed = args[0]
    array_job_id = args[1]
    job_id = args[2]
    print(f"Array Job ID: {array_job_id}  Array Task ID: {seed}  Job ID: {job_id}") 
    print("SEED: {}".format(seed))

    random.seed(seed)
    for i in range(0, 3):
        print(random.random())
```

It takes three arguments, prints its job id, its array job id,
its array task id, and then prints three random numbers.
This script will run once for each subjob in the array: 10 times.

Go ahead and submit the array job.

```bash
sbatch array.sbatch
```

```output
Submitted batch job 31638324
```

Array jobs are **parallel** jobs. 

You have no guarantee that all the array jobs will schedule at the same time.

Let's check progress the job's progress with `squeue`.

```bash
squeue --me
```

```output
   31638482_[1-10]   compute python_a    emwin PD       0:00      1 (None)
```

Each *subjob* in an array job creates in own log.

To look at the last five lines of each of these logs, you can use the `tail` command.

```bash
tail py*.out
```

```output
==> python_array-31638482-8.out <==
Array Job ID: 31638482  Array Task ID: 8  Job ID: 31638490
SEED: 8
0.22991307664292304
0.7963928625032808
0.7965374772142675

==> python_array-31638482-9.out <==
Array Job ID: 31638482  Array Task ID: 9  Job ID: 31638491
SEED: 9
0.1666321090310302
0.02959962356249568
0.629304132385857
```
Notice that each job has its own array task id, its own job id, and a shared array job id.

### When to Use Array Jobs
If you can format your job as an array job, do so! Array jobs are the **preferred way** to parallelize your embarassingly parallel jobs. 
Array jobs are easy for the scheduler to manage. 

### What About Multinode Jobs
Unless you're using an MPI or message passing interface, you should use the default
sbatch param of  `--nodes=1`.
Your jobs can't communicate between computers (nodes) without
code compatible  with MPI. Fall into this case? RACS has a [guide for configuring jobs that use MPI on Talapas](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756676/How-to+Submit+a+MPI+Job).

## Parallelism vs. Serial Execution
Array jobs do not "talk to each other". 
They do not run in a guaranteed order.
They don't finish in a guaranteed order.

### Race Conditions
No two jobs should not be used to write to or read from the same inputs concurrently.
No two jobs should manipulate the same data.
The create **race conditions** in which your code and filesystem can behave unpredictably.

*This is why we make sure everyone in the class copies example files to their home directory
before modifying them and executing them.*

## More Array Job Examples
In the `books` folder, you should have the text of five books.

Use `ls` to list the books stored there.

```bash
ls books
```

```output
alice_in_wonderland.txt         moby_dick.txt            romeo_and_juliet.txt
complete_works_shakespeare.txt  pride_and_prejudice.txt
```

Let's look at an example array job that executes on each of the five books.

```bash
nano books.sbatch
```

```bash
#!/bin/bash

#SBATCH --partition=compute              ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=books_wc_array        ### Job Name
#SBATCH --output=logs/%x-%A-%a.out            ### File in which to store job output
#SBATCH --error=logs/%x-%A-%a.err             ### File in which to store job error messages

#SBATCH --time=0-00:05:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                        ### Number of nodes needed for the job
#SBATCH --mem=50M                        ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task

#SBATCH --array=0-4

BOOKS=(books/*)
srun wc -c ${BOOKS[$SLURM_ARRAY_TASK_ID]}
```

This gives us the `wc` or "word count" in characters of each of the five books.

Also note that this job writes logs to a logs directory.

```bash
sbatch books.sbatch
```

```output
Submitted batch job 31645909`
```

This job will finish very quickly. Check the status using `sacct`.

```bash
sacct
```

```output
`31645909_0   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
31645909_0.+      batch            racs_trai+          1  COMPLETED      0:0 
31645909_0.+     extern            racs_trai+          1  COMPLETED      0:0 
31645909_0.0         wc            racs_trai+          1  COMPLETED      0:0 
31645909_1   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
31645909_1.+      batch            racs_trai+          1  COMPLETED      0:0 
31645909_1.+     extern            racs_trai+          1  COMPLETED      0:0 
31645909_1.0         wc            racs_trai+          1  COMPLETED      0:0 
31645909_2   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
31645909_2.+      batch            racs_trai+          1  COMPLETED      0:0 
31645909_2.+     extern            racs_trai+          1  COMPLETED      0:0 
31645909_2.0         wc            racs_trai+          1  COMPLETED      0:0 
31645909_3   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
31645909_3.+      batch            racs_trai+          1  COMPLETED      0:0 
31645909_3.+     extern            racs_trai+          1  COMPLETED      0:0 
31645909_3.0         wc            racs_trai+          1  COMPLETED      0:0 
31645909_4   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
31645909_4.+      batch            racs_trai+          1  COMPLETED      0:0 
31645909_4.+     extern            racs_trai+          1  COMPLETED      0:0
```

Each book (array job) in the array has its own log in the `logs` folder.

```bash
ls logs
```
```output
books_wc_array-31645909-0.err  books_wc_array-31645909-2.err  books_wc_array-31645909-4.err
books_wc_array-31645909-0.out  books_wc_array-31645909-2.out  books_wc_array-31645909-4.out
books_wc_array-31645909-1.err  books_wc_array-31645909-3.err
books_wc_array-31645909-1.out  books_wc_array-31645909-3.out
```
Let's use `tail` and a `*` wildcard to check the last few lines of each of the output logs.

```bash
tail logs/books*.out
```

```output
==> logs/books_wc_array-31645909-0.out <==
174357 books/alice_in_wonderland.txt

==> logs/books_wc_array-31645909-1.out <==
5638516 books/complete_works_shakespeare.txt

==> logs/books_wc_array-31645909-2.out <==
1276288 books/moby_dick.txt

==> logs/books_wc_array-31645909-3.out <==
772419 books/pride_and_prejudice.txt

==> logs/books_wc_array-31645909-4.out <==
169541 books/romeo_and_juliet.txt
```

You can also use step sizes in the array parameter.

```bash
#SBATCH --array=0-50:10
```

Or you can manually iterate through a list of arguments.

```bash
ARGS=(10 20 30 40 50)
```

### Don't Overwhelm the Scheduler!
Do not make 50,000 individual jobs with `srun` and a for-loop.
Array jobs are the only context where this is appropriate.

You can also tell the array to submit only x jobs at a time.
For example, adding this `#SBATCH --array=0-50000%100` parameter submits only 100 subjobs to Slurm at a time.


## Batch Job Defaults on Talapas
 I have selected a few common parameters. These constraints are subject to change and do not necessarily apply on condo nodes.

| Parameter    | Default| Description | Notes
| -------- | ------- | ---------- | ----------------|
| ntasks | 1 | tasks per job | |
| cpus-per-task | 1 | number of  cpu threads per task | |
| mem-per-cpu | 4G  | memory per cpu thread        | |
| mem | 4G* | memory per node | cannot be used with mem-per-cpu |
| nodes |   1  | nodes allocated for the job | [requires mpi](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756676/How-to+Submit+a+MPI+Job) |
| gpus   | 0   | gpus per job | must be >1 to use gpus|

In summary, parallelism on Talapas is enabled by
* using code that *supports* multiple tasks, threads, or nodes
* enabling additional tasks, cores, or nodes as sbatch parameters
* OR using sbatch array jobs to launch simultaneous, independent jobs

Without meeting these requirements, your jobs will run serially.

## Interactive Jobs: `srun`
Interactive jobs are great for development work if you need a better resourced machine for a couple hours. 

For example, interactive jobs are a great place to try out configuring GPU jobs.

```bash
srun --partition=gpu --account=racs_training --time=30 --gpus=1 --constraint=gpu-10gb --pty bash
```

In this example constraint, you're requesting a GPU with 10GB of VRAM.

```output
srun: job 31651195 queued and waiting for resources
```

Check that you're on the interactive node using `hostname`.

```bash
hostname
```

```output
n0151.talapas.uoregon.edu
```

Use the `nvidia-smi` command to get GPU information for the current node.


```bash
nvidia-smi
```

```output
Tue Feb 18 15:10:20 2025       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.54.15              Driver Version: 550.54.15      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100 80GB PCIe          On  |   00000000:E8:00.0 Off |                   On |
| N/A   38C    P0             75W /  300W |     675MiB /  81920MiB |     N/A      Default |
|                                         |                        |              Enabled |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| MIG devices:                                                                            |
+------------------+----------------------------------+-----------+-----------------------+
| GPU  GI  CI  MIG |                     Memory-Usage |        Vol|      Shared           |
|      ID  ID  Dev |                       BAR1-Usage | SM     Unc| CE ENC DEC OFA JPG    |
|                  |                                  |        ECC|                       |
|==================+==================================+===========+=======================|
|  0   14   0   0  |              12MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |                 0MiB / 16383MiB  |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
                                                                                         
+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

If you requested GPU resources correctly, you should be able to run `nvidia-smi` and see information about the GPU available to you.


Exit your interactive job.

```bash
exit
```

Check that you're back on a login node.

```bash
hostname
```

```output
login2.talapas.uoregon.edu
```

Check `sacct` to see the status of your finished interactive job.

```bash
sacct
```

```output
31651195           bash        gpu racs_trai+          1  COMPLETED      0:0 
31651195.ex+     extern            racs_trai+          1  COMPLETED      0:0 
31651195.0         bash            racs_trai+          1  COMPLETED      0:0
```

This `nvidia-smi` command will fail in contexts where a GPU is unavailable or has not been allocated.

For example, if we run it on the login node, you'll get a message like this.

```bash
nvidia-smi
```

```output
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running
```

RACS has a guide 
to [configuring interactive jobs on Talapas](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756536/How-to+Start+an+Interactive+Job) if you'd like to read more.

Remember to exit interactive jobs when you're finished to free up resources for your colleagues!

## Conda Environments
Conda environments allow you to preserve consistent packages between different operating systems and devices using environment files. 

RACS has a detailed guide for [building, creating, and loading conda environments](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2916614147/How-To+Creating+personalized+conda+environments) on Talapas. We will walk through some of these steps in this exercise.

To start playing around with Conda, let's launch an interactive session with `srun`.

```bash 
srun --partition=interactive --account=racs_training --time=60 --pty bash
```

Check the current hostname with `hostname` to confirm we're on an interactive node.

```bash
hostname
```

```output
n0302.talapas.uoregon.edu
```

Let's peek inside the `/projects/racs_training/conda_example/` directory with `ls`.

```bash
ls /projects/racs_training/conda_example/
```
```output
basic_r.R  environment.yml  hello_world.py
```

### Why `conda` 
[Conda](https://docs.conda.io/projects/conda/en/stable/user-guide/getting-started.html) is an environment manager for 
Python that creates a manages Python packages and depdencies through virtual environments..

You can switch between different environments on the same computer. 
Many researchers will maintain different conda environments for different contexts.

### What About `pip`?
Pip is an [alternative package manager](https://www.anaconda.com/blog/understanding-conda-and-pip) and it can be paired with `venv` to create Python virtual environments. Using `pip` within a `conda` environment is not recommended,
but it can be a necessary evil for packages available in pip but not conda.

### Using .yml Files 
Let's use the copy command to copy these files to our home directory.

Navigate to your home directory using the `~` symbol then copy the `conda_example` folder into it.
Don't forget the recursive `-r` flag~

```bash
cd ~
cp -r /projects/racs_training/conda_example/ .
```

Navigate inside our new `conda_example` directory using `cd`.

```bash
cd conda_example/
ls
```

```output
basic_r.R  environment.yml  hello_world.py
```

## An Example Dual Python and R Environment
You can install R into a conda environment, which allows you to manage [R
package versions](https://docs.anaconda.com/working-with-conda/packages/using-r-language/) for your R projects.
Let's look at the example skeleton environment.yml in `nano`.
It's missing crucial pieces that we will fill out.

```bash
nano environment.yml
```

```python
name:
channels:
  - 
  -
  -
dependencies:
  - python=3.12
  - r-base=
  - r-tidyverse
  - r-ggplot2
  - r-dplyr
  - r-readr
  - r-devtools
  - r-lubridate
  - r-data.table
```

Every Conda environment file has a
**name**, **channels**, and **dependencies**.

Let's name this `r-4.3.3_environment`.
Set your r-base to 4.3.3 to match.

For channels, we need to both R and Python packages.
* default (default for Python and R)
* conda-forge (more versions of conda packages)
* r (for a larger list of r packages)

For any r package, you need a special `r` channel.

### Channels
[Channels](https://docs.conda.io/projects/conda/en/stable/user-guide/concepts/channels.html) are distributions of packages where conda will pull as it tries to resolve dependencies.
Channels listed first are higher priority, and packages will be sourced from higher priority channels if they are available
in multiple.

Here's our final base environment. You can install more packages to this later.

```python
name: r-4.3.3_environment
channels:
  - defaults
  - conda-forge
  - r
dependencies:
  - python=3.12
  - pandas
  - r-base=4.3.3
  - r-tidyverse
  - r-ggplot2
  - r-dplyr
  - r-readr
  - r-devtools
  - r-lubridate
  - r-data.table
```

To use `conda` on Talapas, we need to load its software module.

```bash
module load miniconda3/202410
```

Then, use the `conda env create` command to 
[create a new environment](https://docs.conda.io/projects/conda/en/stable/commands/env/create.html) from the `environment.yml` file.

```bash
conda env create -f environment.yml
```

Please wait while your environment is built.
Once it's done, you will get a message like this.

```output
Preparing transaction: done                                                                     
Verifying transaction: done                                                                     
Executing transaction: done                                                                     
#                                                                                               
# To activate this environment, use                                                             
#                                                                                               
#     $ conda activate r-4.3.3_environment                                                      
#                                                                                               
# To deactivate an active environment, use                                                      
#                                                                                               
#     $ conda deactivate                                                                                   
```

Activate your new environment using the `conda activate` command.

```bash
conda activate r-4.3.3_environment
```

You can use `conda` to manage several environments, but you will only need to **create** each environment once. 

### Where Are My Conda Environments?
Conda environments are saved to your `.conda` directory in your home directory, so it is available to you on login and compute nodes alike.

You can see a list of all the conda environments you've created on Talapas by using the following `ls` command:

```bash
ls ~/.conda/envs/
```

```output
r-4.3.3_environment
```

### Checking Installed Packages with `conda list`
Use `conda list` to check the packages inside for the r packages of interest.

```bash
conda list
```

```output
...
r-askpass                 1.2.1             r43h2b5f3a1_0    conda-forge
r-assertthat              0.2.1             r43hc72bb7e_5    conda-forge
r-backports               1.5.0             r43hb1dbf0f_1    conda-forge
r-base                    4.3.3               h5074ccb_16    conda-forge
r-base64enc               0.1_3           r43hb1dbf0f_1007    conda-forge
...
```

Inside a conda environment, you will use a Python instance in `~/.conda/[env_name]` that you have full control over. You can arbitrarily install packages as you need them inside this environment.

You can see which Python environment you're using with the `which` command/.

```bash
which python
```

```output
~/.conda/envs/r-4.3.3_environment/bin/python
```
### Veryifying your Python Environment
Use `python` to start the version of python you installed.

```bash
python
```

```output
Python 3.12.9 | packaged by conda-forge | (main, Feb 14 2025, 08:00:06) [GCC 13.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
```

If your conda environment was built correctly, you should have access to the [pandas package](https://pandas.pydata.org/docs/). Let's test by importing it and creating a DataFrame.

```python
import pandas as pd
my_frame = pd.DataFrame(
  {"height": [5, 2], "weight": [1, 3]})
my_frame
```

```output
   height  weight
0       5       1
1       2       3
```
With that, you've confirmed pandas was installed.
Exit by Python by typing and pressing <kbd>Enter</kbd>.

```python
exit()
```

### Verifying Your R Environment
Use `R` to start the version of R you installed.

`bash
R
`

Inside the R terminal, you can load R packages you installed through the conda environment using `library()`.
Let's load the [tidyverse](https://www.tidyverse.org/) library.

```R
> library(tidyverse)
── Attaching core tidyverse packages ── tidyverse 2.0.0 ──
✔ dplyr     1.1.4     ✔ readr     2.1.5
✔ forcats   1.0.0     ✔ stringr   1.5.1
✔ ggplot2   3.5.1     ✔ tibble    3.2.1
✔ lubridate 1.9.4     ✔ tidyr     1.3.1
✔ purrr     1.0.2     
── Conflicts ──────────────────── tidyverse_conflicts() ──
✖ dplyr::filter() masks stats::filter()
✖ dplyr::lag()    masks stats::lag()
ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors
```

Let's make a quick `data.frame`.

```R
>  data <- data.frame(a = 1:3, b = letters[1:3])
> data
  a b
1 1 a
2 2 b
3 3 c
```

When you've finished, exit the R interpreter using `q()`.

```R
q()
```

Choose not to save the workspace image with `n` and exit.


You can now write `sbatch` jobs that use the conda environments you create using statements like this.

### Batch Python Example
```bash
module load miniconda3/202410
conda activate r-4.3.3_environment
python [my-special-python-script].py
```

### Batch R Example
```bash
module load miniconda3/202410
conda activate r-4.3.3_environment
R [my-special-r-script].R
```

## Today's Slurm Commands

| command | description | example usage |
| ----------- | --------------- | ------------- |
| sbatch [jobfile] | queues a Slurm job | `sbatch my_job.sh` |
| sacct | lists (your) recent jobs and subjobs | `sacct` |
| squeue -u [user] | gets status of running or queued slurm jobs for user | `squeue -u [yourDuckID]` |
| scancel [jobid] | cancels job jobid | `scancel [jobid]` |
| srun | launches interactive shell session on a compute node | `srun --partition=compute --account=racs_training --pty bash` |
