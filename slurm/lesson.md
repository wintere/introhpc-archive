---
title: Lesson
layout: page
permalink: /slurm/lesson.html
parent: Slurm on Talapas
nav_enabled: true
nav_order: 2
---

# Slurm on Talapas

[Slurm](https://slurm.schedmd.com/slurm.html) is job scheduling software that apportions resources to jobs distributed across the hundreds of nodes on Talapas.
To use Talapas, you must request resources and define job parameters in the form of Slurm jobs.
This lesson introduces Slurm job configuration.

## Lesson Setup
For this lesson, you will need to connect to a Talapas login node through a shell application of your choice.

For convenience, we recommend the [Talapas OnDemand shell](https://ondemand.talapas.uoregon.edu/pun/sys/shell/ssh/login1.talapas.uoregon.edu).

To start, make sure you are in your home directory.

```bash
cd ~
```

Copy the `/projects/racs_training/intro-hpc-s25/slurm` folder into your home directory. Don't forget `-r` to recursively copy the contents!

```bash
cp -r /projects/racs_training/intro-hpc-f25/slurm .
```

Navigate inside the newly created `slurm` directory.
```bash
cd slurm
ls -F
```

```output
part1/  part2/
```

Navigate inside the `part1` folder using `cd`, then inspect its contents using `ls`.

```bash
cd part1
ls
```

```output
gpu.sbatch  hello.py  hello.sbatch  long.sbatch
```


## Troubleshooting a Batch Slurm Job
Let's inspect `hello.sbatch`.

This is an imperfect Slurm job: it's missing an important step!
We will start with what it does correctly, then fix
the common mistake.

```bash
cat hello.sbatch
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

python hello.py
```
Let's parse this file using the comments as a guide! To learn more, see the [Slurm documentation](https://slurm.schedmd.com/sbatch.html) for `sbatch`.

* This job runs for a maximum of five minutes.
(`0-00:05:00`) on one node and one CPU core of that node.
* It requests 500MB of RAM.
* The job output will be written to `hello_world_python-[jobid].out` and `hello_world_python-[jobid].err` respectively.
* It runs a file in the `slurm_examples` directory called `hello.py` that prints to standard output. 

Given that this task is relatively trivial, these parameters are appropriate.

Inspect the `hello.py` script called by the batch script with `cat`.

```bash
cat hello.py
```

```output
print("Hello Wold !!")
```

For those unfamiliar with Python, this is a trivial "Hello World" job that prints
the phrase "Hello Wold !!" to standard out.

To submit your job file, run the `sbatch` command.

```bash
sbatch hello.sbatch
```

```output
Submitted batch job 39456543
```

To check the status of your queued jobs, use the `squeue` command. Do not use squeue without an argument unless you want to see *all* the queued jobs on Talapas.

If you check your queue, you will probably see that this job has
already terminated.

```bash
squeue --me
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

Because `squeue` only tracks jobs in progress, you will need to check your 
recent jobs regardless of status with `sacct`.

```bash
sacct
```

```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
39456543     hello_wor+    compute racs_trai+          1     FAILED    127:0 
39456543.ba+      batch            racs_trai+          1     FAILED    127:0 
39456543.ex+     extern            racs_trai+          1  COMPLETED      0:0
```

The parent job, 39456543, failed. Check the error log using `cat`.

```bash
cat hello_world_python-*.err
```

```output
/var/spool/slurm/job39456543/slurm_script: line 17: python: command not found
```

Remember modules? Python is *not* on your path on Talapas default.
This means that the job on the compute node must load
a version of Python before it can interpret the Python script.

### A Corrected Batch Script
Correct the script by inserting two lmod statements: `module purge` and `module load python3/3.11.4`.

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

module purge # Best practice, unloads all modules
module load python3/3.11.4 # Loads Python
python hello.py
```

Remove the old error logs.

```bash
rm hello_world_python-*.err
rm hello_world_python-*.out
```

Resubmit the job with the corrected batch script.

```bash
sbatch hello.sbatch
```

```output
Submitted batch job 39456578
```

Running `sacct` will show your job completed successfully.

```bash
sacct
```

```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- --------
39456578     hello_wor+    compute racs_trai+          1  COMPLETED      0:0 
39456578.ba+      batch            racs_trai+          1  COMPLETED      0:0 
39456578.ex+     extern            racs_trai+          1  COMPLETED      0:0
```

Check the output log to see the results of your job. Note that because each log is named for the jobid, and jobids are unique, your logs will not overwrite each other. 

Check the contents of the output logs with `cat`.

```bash
cat hello_world_python*
```

```output
Hello Wold !!
```

## Managing Runnning Jobs

Inspect `long.sbatch`.

```bash
cat long.sbatch
```

```output
#!/bin/bash

#SBATCH --partition=computelong          ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=long_hello_world      ### Job Name
#SBATCH --output=%x-%j.out               ### File in which to store job output
#SBATCH --error=%x-%j.err                ### File in which to store job error messages

#SBATCH --time=0-00:20:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                        ### Number of nodes needed for the job
#SBATCH --mem=500M                       ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task


### Run your actual program
for i in {1..100}
do
    echo "This is loop iteration $i"
    sleep 10
done
```

This job features a [Bash for-loop](../talapas-scripting/lesson.html#loops-easy-as-for-do-done)
that will print one line of text and sleep for ten seconds 100 times, once for each iteration of the loop. 
This means it will run for at least 1000 seconds or for about 15 minutes.

Submit the job with `sbatch`.

```bash
sbatch long.sbatch
```

```output
Submitted batch job 39456611
```

Inspect the job in the queue with `squeue`.

```bash
squeue --me
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          39456611 computelo long_hel    emwin  R       0:32      1 n0185
```

Let's unpack this status:
- The job has the status R for running.
- It's running on node n0135 on the computelong partition.
- The job is running as emwin.
- It has been running for 32 seconds.

To learn more about `squeue` results, see the [Slurm documentation](https://slurm.schedmd.com/squeue.html).

### Canceling Running Batch Jobs
Let's practice cancelling a slow job with `scancel`, which requires the job number of the job
to be cancelled.

```bash
scancel 39456611
```

Looking at the queue again, it's now empty, even though the job would have otherwise run for longer.

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

When you cancel a job, Slurm will write an error message to the error file for that file.

```bash
cat long_hello_world*.err
```

```output
slurmstepd: error: *** JOB 39456611 ON n0185 CANCELLED AT 2025-10-23T13:32:09 ***
```

### Optimization with `seff`

Want to know how much of the resources you requested a finished job used? Try the `seff` command.

```bash
seff 39456611
```

```output
Job ID: 39456611
Cluster: talapas
User/Group: emwin/uoregon
State: CANCELLED (exit code 0)
Cores: 1
CPU Utilized: 00:00:00
CPU Efficiency: 0.00% of 00:01:47 core-walltime
Job Wall-clock time: 00:01:47
Memory Utilized: 460.00 KB
Memory Efficiency: 0.09% of 500.00 MB
```

Adjust your resource requests accordingly after your job runs. Be courteous and jobs won't be deprioritized. 
Remember that your job's runtime is its **queue time + execution time**, so asking for too many resources can make your job run *slower*
because it will spend longer in the queue.

If your job runs out of memory, try doubling the amount you request.
If you job uses less memory than you expected, you can use `seff` to *lower* the amount of memory requested and have your job
schedule faster next time.

This job, which spends its CPU time "sleeping" or waiting, uses very little of its requested resources.

Another way to get resource usage is to
pass in formatting arguments to the `sacct` command.

We recommend inspecting **MaxRSS** (maximum memory usage) and **ReqMem** (memory requested)
to determine memory effeciency of finished jobs.

Let's use `sacct` with the following parameters.

```bash
sacct --units=G --format=JobId,jobname,MaxRSS,ReqMem,Start,Elapsed,State
```

```output
JobID           JobName     MaxRSS     ReqMem               Start    Elapsed      State 
------------ ---------- ---------- ---------- ------------------- ---------- ---------- 
39456543     hello_wor+                 0.49G 2025-10-23T13:12:44   00:00:00     FAILED 
39456543.ba+      batch          0            2025-10-23T13:12:44   00:00:00     FAILED 
39456543.ex+     extern          0            2025-10-23T13:12:44   00:00:00  COMPLETED 
39456570     hello_wor+                 0.49G 2025-10-23T13:18:52   00:00:00     FAILED 
39456570.ba+      batch          0            2025-10-23T13:18:52   00:00:00     FAILED 
39456570.ex+     extern          0            2025-10-23T13:18:52   00:00:00  COMPLETED 
39456578     hello_wor+                 0.49G 2025-10-23T13:23:53   00:00:01  COMPLETED 
39456578.ba+      batch          0            2025-10-23T13:23:53   00:00:01  COMPLETED 
39456578.ex+     extern          0            2025-10-23T13:23:53   00:00:01  COMPLETED 
39456611     long_hell+                 0.49G 2025-10-23T13:30:22   00:01:47 CANCELLED+ 
39456611.ba+      batch      0.00G            2025-10-23T13:30:22   00:01:48  CANCELLED 
39456611.ex+     extern      0.00G            2025-10-23T13:30:22   00:01:48  COMPLETED
```

## Introducing GPU Jobs: Debugging Activity
Let's examine an example GPU job in detail: `gpu.sbatch`.

Only three partitions are compatible with GPUs:
* gpu
* gpulong
* interactivegpu

Each node in a GPU partition has **at most** 4 GPUS.

Examine `gpu.sbatch`.

```bash
cat gpu.sbatch
```

```bash
#!/bin/bash

#SBATCH --partition=compute              ### Partition (like a queue in PBS)
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

#SBATCH --constraint=gpu-10gb

### SLURM can even email you when jobs reach certain states:
### #SBATCH --mail-type=BEGIN,END,FAIL       ### accepted types are NONE,BEGIN,END,FAIL,REQUEUE,ALL (does all)
### #SBATCH --mail-user=<duckID>@uoregon.edu


### Load needed modules
module purge
module load cuda/12.4.1
module list

### Run your actual program
nvidia-smi
```

* You request at least one gpu with the `--gpus` argument. It's not enough to run on a GPU partition.
* If you need to select certain nodes within a partition, the `--constraint=` flag, can allow you to specify certain nodes based on job constraints. In this case `--constraint=gpu-10gb` restricts the job to nodes with the `gpu-10gb` feature.
* That is, it gives a GPU with 10GB of VRAM.
* Every GPU requires a CPU too!

If you try to `sbatch` this job, Slurm will refuse to queue it. There's a problem with the Slurm configuration!

```bash
sbatch gpu.sbatch 
```

```output
sbatch: error: Batch job submission failed: Requested node configuration is not availablesbatch: error: Batch job submission failed: Requested node configuration is not available
```

You cannot request `--gpus=1` from a partition without nodes with GPUs.
Fix the job by requesting one of the three partitions that have GPUs available: `gpu`, `interactivegpu` or `gpulong`.

```bash
nano gpu.sbatch
```

```bash
#!/bin/bash

#SBATCH --partition=gpu              ### Partition (like a queue in PBS)
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

#SBATCH --constraint=gpu-10gb

### SLURM can even email you when jobs reach certain states:
### #SBATCH --mail-type=BEGIN,END,FAIL       ### accepted types are NONE,BEGIN,END,FAIL,REQUEUE,ALL (does all)
### #SBATCH --mail-user=<duckID>@uoregon.edu


### Load needed modules
module purge
module load cuda/12.4.1
module list

### Run your actual program
nvidia-smi
```

### Emailing Job Status 
Running a really long job? Tired of checking `squeue`?

SLURM can even email you when jobs reach certain states:

```bash
### #SBATCH --mail-type=BEGIN,END,FAIL      
### #SBATCH --mail-user=<duckID>@uoregon.edu
```

Edit these lines in your sbatch script with *your* DuckID. You should get an email from Slurm when the job begins and finishes (whether in success or failure.) 

Schedule the GPU job and wait for results in your email. It should complete successfully.

```bash
sbatch gpu.sbatch
```

```output
Submitted batch job 39459903
```

Once you receive an email confirmation, check the status with `sacct`.

```bash
sacct
```

```output
39459903     gpu_hello+        gpu racs_trai+          1  COMPLETED      0:0 
39459903.ba+      batch            racs_trai+          1  COMPLETED      0:0 
39459903.ex+     extern            racs_trai+          1  COMPLETED      0:0
```

The `nvidia-smi` command gives you the status of GPUs on your system, if any. 
It will error out if there are no GPUs on the system.
Use `head` to peek at the message.

```bash
head gpu_hello_world*.out
```

```output
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.163.01             Driver Version: 550.163.01     CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100 80GB PCIe          On  |   00000000:E8:00.0 Off |                   On |
| N/A   37C    P0             72W /  300W |     706MiB /  81920MiB |     N/A      Default |
```


## Serial Processing: One Core and One Task at A Time
Navigate inside the `part2` folder of the `slurm` folder using `cd`.

```bash
cd ../part2
```

Check that you see the following files inside with `ls`.

```bash
ls -F
```

```output
array.sbatch*  books.sbatch*  parallel_steps.sbatch*  serial_steps.sbatch*
books/         hello.sbatch*  random.py*              steps.sh*
```

### Making Job Steps with Srun

Examine `hello.sbatch` with `cat`.

```bash
#SBATCH --partition=compute              ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=hello                 ### Job Name
#SBATCH --output=%x.out                  ### File in which to store job output
#SBATCH --error=%x.err                   ### File in which to store job error messages

#SBATCH --time=0-00:05:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                        ### Number of nodes needed for the job
#SBATCH --mem=50M                        ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task

srun echo "Hello From a SLURM Job Step!" 
```

Submit this job with `sbatch`.

```bash
sbatch hello.sbatch
```

```output
Submitted batch job 39459774
```

This job creates a single Slurm [job step](https://slurm.schedmd.com/heterogeneous_jobs.html#job_steps) within the 
parent sbatch job. All components of a job step will have the same step ID value.

Check the job status with `sacct`.

```bash
sacct
```

```output
39459774          hello    compute racs_trai+          1  COMPLETED      0:0 
39459774.ba+      batch            racs_trai+          1  COMPLETED      0:0 
39459774.ex+     extern            racs_trai+          1  COMPLETED      0:0 
39459774.0         echo            racs_trai+          1  COMPLETED      0:0
```

Observe the fourth line! The `echo` command get its own job step `39459774.0`, a child
of the parent `hello` job.

### An Example Serial Job
Examine `serial_steps.sbatch` job with `cat.

```bash
cat serial_steps.sbatch
```

```bash
#!/bin/bash

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
    srun steps.sh $i
done
```

Examine the `step.sh` script called by the batch script.

```bash
cat step.sh
```

```bash
#!/usr/bin/bash

echo "I am printing this from job step $1" & sleep 10
```

This job runs a Bash for-loop. To learn more about Bash loops, [check out this quick tutorial](../talapas-scripting/loops-variables.html).

`srun` [creates a step within a job](https://slurm.schedmd.com/srun.html).
A job with multiple `srun` commands will create a separate step (with the same jobid) in the `sacct`
for each step.

This script uses `srun` to create 10 sequential (serial) steps in one job. By default, each iteration (here `i = 1, 2, 3 ... 10`) must finish before the loop moves to the next.

```bash
sbatch serial_steps.sbatch
```

```output
Submitted batch job 39456708
```

Check the queue for the status of your job.
```bash
squeue --me
```

```output
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          39456708   compute serial_s    emwin  R       0:07      1 n0185
```
As you can observe, the serial job is running and corresponds to exactly one row in the Slurm queue.

However, each step of serial steps (each subjob) is treated as a separaterow in the `sacct` command.

```bash
sacct
```

```output
39456708.0     steps.sh          0            2025-10-23T14:09:01   00:00:10 
39456708.1     steps.sh          0            2025-10-23T14:09:11   00:00:10 
39456708.2     steps.sh      0.00G            2025-10-23T14:09:21   00:00:10 
39456708.3     steps.sh          0            2025-10-23T14:09:31   00:00:10 
39456708.4     steps.sh          0            2025-10-23T14:09:41   00:00:10 
39456708.5     steps.sh                       2025-10-23T14:09:51   00:00:04
```

This job is on loop 5 of the serial steps.
39456708.6 can't begin until 39456708.5 finishes.
Serial jobs require that steps be completed one at a time and in a specific order.

Let's look at the time spent on each stage in more detail using a specially configured `sacct` command. You can learn more about `sacct` [formatting on the Slurm documentation](https://slurm.schedmd.com/sacct.html).

```bash
sacct --units=G --format=JobID,jobname,MaxRSS,Start,Elapsed,State,ExitCode
```

```output
39456708     serial_st+            2025-10-23T14:09:01   00:01:41  COMPLETED      0:0 
39456708.ba+      batch      0.00G 2025-10-23T14:09:01   00:01:41  COMPLETED      0:0 
39456708.ex+     extern      0.00G 2025-10-23T14:09:01   00:01:41  COMPLETED      0:0 
39456708.0     steps.sh          0 2025-10-23T14:09:01   00:00:10  COMPLETED      0:0 
39456708.1     steps.sh          0 2025-10-23T14:09:11   00:00:10  COMPLETED      0:0 
39456708.2     steps.sh      0.00G 2025-10-23T14:09:21   00:00:10  COMPLETED      0:0 
39456708.3     steps.sh          0 2025-10-23T14:09:31   00:00:10  COMPLETED      0:0 
39456708.4     steps.sh          0 2025-10-23T14:09:41   00:00:10  COMPLETED      0:0 
39456708.5     steps.sh          0 2025-10-23T14:09:51   00:00:11  COMPLETED      0:0 
39456708.6     steps.sh          0 2025-10-23T14:10:02   00:00:10  COMPLETED      0:0 
39456708.7     steps.sh      0.00G 2025-10-23T14:10:12   00:00:10  COMPLETED      0:0 
39456708.8     steps.sh          0 2025-10-23T14:10:22   00:00:10  COMPLETED      0:0 
39456708.9     steps.sh          0 2025-10-23T14:10:32   00:00:10  COMPLETED      0:0
```

As you can see, all ten steps (numbered 0-9) are now marked as `COMPLETED.` Each subjob has a different *start* time.
The subjobs start roughly ten seconds apart, after the previous subjob has finished.

Now that the job has completed, inspect the output logs.

```bash
cat serial_steps.out
```

```output
I am printing this from job step 1
I am printing this from job step 2
I am printing this from job step 3
I am printing this from job step 4
I am printing this from job step 5
I am printing this from job step 6
I am printing this from job step 7
I am printing this from job step 8
I am printing this from job step 9
I am printing this from job step 10
```
Each subjob completes in a linear, serial fashion.

There are some jobs where certain steps or subcomponents must be completed in a serial fashion.
However, serial jobs do not take full advantage of the computational power of Talapas or its support for parallelism.
Think of serial computation as a last resort when using Talapas.

How can we make this job run in parallel? Will it be faster?

## Parallelism with `sbatch`

Let's examine `parallel-steps.sbatch` with `cat`.

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

for i in {1..10}; do
    srun --ntasks=1 steps.sh $i &
done
wait
```

This job requests 10 CPUS, allocates 1 CPU per task, and then
launches 10 concurrent `srun` subjobs.

The `&` at the end of `srun --ntasks=1 test.sh $i &` is crucial!
The ampersand tells Bash to *run this command in the background*. Without the ampersand
this loop would wait for one iteration to finish before moving to the next one.

Let's launch this command with `sbatch`.
```bash
sbatch parallel_steps.sbatch
```

```output
Submitted batch job 39456813
```

This job will finish very quickly.

If you look at the output log, you'll notice something important.

The job steps **don't** complete in order!

```bash
cat parallel_steps.out
```

```output
I am printing this from job step 7
I am printing this from job step 5
I am printing this from job step 3
I am printing this from job step 9
I am printing this from job step 10
I am printing this from job step 2
I am printing this from job step 6
I am printing this from job step 8
I am printing this from job step 1
I am printing this from job step 4
```
The steps in the parallel job **did not run in order** or **finish in order**. You lose that guarantee in a parallel context.

How much faster did the parallel job run?

The serial job ran in 70 seconds.
Use `sacct` to see how much faster the parallel job ran.

```bash
sacct --units=G --format=JobID,jobname,MaxRSS,Start,Elapsed,State,ExitCode
```

```output
JobID           JobName     MaxRSS               Start    Elapsed      State ExitCode 
39456813     parallel_+            2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.ba+      batch          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.ex+     extern          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.0     steps.sh          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.1     steps.sh          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.2     steps.sh          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.3     steps.sh          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.4     steps.sh      0.00G 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.5     steps.sh      0.00G 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.6     steps.sh          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.7     steps.sh          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.8     steps.sh          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0 
39456813.9     steps.sh          0 2025-10-23T14:14:32   00:00:10  COMPLETED      0:0
```

The parallel job ran in 11 seconds, significantly faster than the 70 second *serial* version of the job.
This is an example of an *embarassingly parallel* job, because each subjob is completely independent of the other subjobs.

Before starting our next activity, clear out old logs with `rm`.

```bash
rm *.err *.out
```

## Array Jobs

Slurm has a special option for bulk scheduling of nearly identical jobs: [array jobs](https://slurm.schedmd.com/job_array.html).
This approach is less flexible than `srun`, but it can be faster to configure.

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
srun python3 random_array.py $SLURM_ARRAY_TASK_ID $SLURM_ARRAY_JOB_ID $SLURM_JOB_ID
```

Array jobs require the `#SBATCH --array=` parameter.
In this example, the array parameter is the list `{0, 1, 2, ... 10}`.
The `%a` stands for the id of the *task* and the `%A` stands for the main array.

This will create one main job and a subjob for each index in the array. Each array index will inherit the
requested parameters specified in `cpus-per-task` and `ntasks-per-node`. This means each subjob asks for 1 CPU core.

This job runs `random_array.py` and passes in three arguments: the $SLURM_ARRAY_TASK_ID, the $SLURM_ARRAY_JOB_ID and $SLURM_JOB_ID. `$SLURM_ARRAY_TASK_ID $SLURM_ARRAY_JOB_ID $SLURM_JOB_ID` are environment variables.

The advantage of array jobs is that Slurm does not wait for all requested cores to be available
for the first subjobs of the array jobs to be scheduled. *This is not the case for parallel jobs that utilize `srun` to distribute
subjobs among multiple requested cores.*

Let's look in more detail at `random_array.py`.

```bash
cat random_array.py
```

```python
#!/usr/bin/env python3

import random
import sys

if __name__ == "__main__":
    args = sys.argv[1:]
    seed = args[0] # or Array Task ID
    array_job_id = args[1]
    job_id = args[2]
    print(f"Array Job ID: {array_job_id}  Array Task ID: {seed}  Job ID: {job_id}") 
    print(f"SEED: {seed}")

    random.seed(seed)
    for i in range(0, 3):
        print(random.random())
```

This Python script takes three arguments (array_job_id, array_task_id, job_id) and and then prints three random numbers
This script will run once for each subjob in the array: 10 times.

Go ahead and submit the array job.

```bash
sbatch array.sbatch
```

```output
Submitted batch job 39459879
```

Array jobs are **parallel** jobs. 

You have no guarantee that all the array jobs will schedule or run at the same time.

Let's check progress the job's progress with `sacct`.

```bash
sacct
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
39459879_1   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_1.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_1.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_1.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_2   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_2.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_2.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_2.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_3   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_3.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_3.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_3.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_4   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_4.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_4.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_4.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_5   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_5.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_5.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_5.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_6   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_6.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_6.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_6.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_7   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_7.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_7.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_7.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_8   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_8.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_8.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_8.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_9   python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_9.+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_9.+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_9.0    python3            racs_trai+          1  COMPLETED      0:0 
39459879_10  python_ar+    compute racs_trai+          1  COMPLETED      0:0 
39459879_10+      batch            racs_trai+          1  COMPLETED      0:0 
39459879_10+     extern            racs_trai+          1  COMPLETED      0:0 
39459879_10+    python3            racs_trai+          1  COMPLETED      0:0
```

Each *subjob* in an array job creates in own log. Use a wildcard and `ls` to list them.

```bash
ls python_array*
```

```output
python_array-34704494-10.err  python_array-34704494-2.out  python_array-34704494-5.err  python_array-34704494-7.out
python_array-34704494-10.out  python_array-34704494-3.err  python_array-34704494-5.out  python_array-34704494-8.err
python_array-34704494-1.err   python_array-34704494-3.out  python_array-34704494-6.err  python_array-34704494-8.out
python_array-34704494-1.out   python_array-34704494-4.err  python_array-34704494-6.out  python_array-34704494-9.err
python_array-34704494-2.err   python_array-34704494-4.out  python_array-34704494-7.err  python_array-34704494-9.out
```

To look at the last five lines of each of the output logs logs, you can use the `tail` command.

```bash
tail py*.out
```

```output
==> python_array-39459879-10.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 10  Job ID: 39459879
SEED: 10
0.8038117609963674
0.6395838575221652
0.10813824794140992

==> python_array-39459879-1.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 1  Job ID: 39459880
SEED: 1
0.4782479962566343
0.044242767098090496
0.11703586901195051

==> python_array-39459879-2.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 2  Job ID: 39459881
SEED: 2
0.5558450440322502
0.637051760769432
0.32591821829634604

==> python_array-39459879-3.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 3  Job ID: 39459882
SEED: 3
0.8677521680526462
0.5835116078800329
0.6899116355769067

==> python_array-39459879-4.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 4  Job ID: 39459883
SEED: 4
0.16000523129035404
0.3018180364144196
0.6137173239349033

==> python_array-39459879-5.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 5  Job ID: 39459884
SEED: 5
0.35258858106537283
0.7805236423847258
0.5172954265015355

==> python_array-39459879-6.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 6  Job ID: 39459885
SEED: 6
0.3898339501015142
0.9554066922708754
0.3550297799037352

==> python_array-39459879-7.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 7  Job ID: 39459886
SEED: 7
0.7124376338974682
0.8779104020178983
0.9507485116206883

==> python_array-39459879-8.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 8  Job ID: 39459887
SEED: 8
0.22991307664292304
0.7963928625032808
0.7965374772142675

==> python_array-39459879-9.out <==

FILE
    /packages/miniconda-t2/20230523/envs/python-3.11.4/lib/python3.11/random.py


Array Job ID: 39459879  Array Task ID: 9  Job ID: 39459888
SEED: 9
0.1666321090310302
0.02959962356249568
0.629304132385857
```
Notice that each job has its own array task id, its own job id, and a shared array job id.

### When to Use Array Jobs
If you can format your job as an array job, do so! Array jobs are the **preferred way** to parallelize your embarassingly parallel jobs. 
Array jobs are easy for the scheduler to manage. 

## Parallelism vs. Serial Execution
Array jobs do not "talk to each other". They do not run in a guaranteed order, nor do they finish in a guaranteed order.

Regardless of how your job is configured, no two jobs or subjobs should not be used to write to or read from the same inputs concurrently.
This create **race conditions** in which your code and filesystem can behave unpredictably.

*Race conditions are why we make sure everyone in the class copies example files from shared directories to their home directory
before modifying them and executing them.

## Example Array Job: Books
In the `books` folder, you should have the text of five books. Let's say you
want to get the word count of each book.

```bash
ls books
```

```output
alice_in_wonderland.txt         moby_dick.txt            romeo_and_juliet.txt
complete_works_shakespeare.txt  pride_and_prejudice.txt
```

Let's examine books.sbatch, which computes the word counts of each book using the `wc` command.

```bash
cat books.sbatch
```

```bash
#!/bin/bash

#SBATCH --partition=compute              ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=books_wc_array        ### Job Name
#SBATCH --output=logs/%x-%A-%a.out       ### File in which to store job output
#SBATCH --error=logs/%x-%A-%a.err        ### File in which to store job error messages

#SBATCH --time=0-00:05:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                        ### Number of nodes needed for the job
#SBATCH --mem=50M                        ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task

#SBATCH --array=0-4

BOOKS=(books/*)
srun wc -c ${BOOKS[$SLURM_ARRAY_TASK_ID]}
```

Submit the job with `sbatch`.
```bash
sbatch books.sbatch
```

```output
Submitted batch job 39457114
```

Let's check the job status.

```bash
sacct
```

```output
39457114_0   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
39457114_0.+      batch            racs_trai+          1  COMPLETED      0:0 
39457114_0.+     extern            racs_trai+          1  COMPLETED      0:0 
39457114_0.0         wc            racs_trai+          1  COMPLETED      0:0 
39457114_1   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
39457114_1.+      batch            racs_trai+          1  COMPLETED      0:0 
39457114_1.+     extern            racs_trai+          1  COMPLETED      0:0 
39457114_1.0         wc            racs_trai+          1  COMPLETED      0:0 
39457114_2   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
39457114_2.+      batch            racs_trai+          1  COMPLETED      0:0 
39457114_2.+     extern            racs_trai+          1  COMPLETED      0:0 
39457114_2.0         wc            racs_trai+          1  COMPLETED      0:0 
39457114_3   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
39457114_3.+      batch            racs_trai+          1  COMPLETED      0:0 
39457114_3.+     extern            racs_trai+          1  COMPLETED      0:0 
39457114_3.0         wc            racs_trai+          1  COMPLETED      0:0 
39457114_4   books_wc_+    compute racs_trai+          1  COMPLETED      0:0 
39457114_4.+      batch            racs_trai+          1  COMPLETED      0:0 
39457114_4.+     extern            racs_trai+          1  COMPLETED      0:0 
39457114_4.0         wc            racs_trai+          1  COMPLETED      0:0 
```

These following lines of `sbatch` configuration direct the output logs to the `logs` folder.
The `%x-%A-%a` notation creates log files named `(job name)-(array parent job)-(array index)`.out.

```bash
#SBATCH --output=logs/%x-%A-%a.out       ### File in which to store job output
#SBATCH --error=logs/%x-%A-%a.err        ### File in which to store job error messages
```

Inspect the logs folder with `ls`. You should see output and error logs for each book.

```bash
ls logs
```

```output
books_wc_array-34763686-0.err
books_wc_array-34763686-0.out
books_wc_array-34763686-1.err
books_wc_array-34763686-1.out
books_wc_array-34763686-2.err
books_wc_array-34763686-2.out
books_wc_array-34763686-3.err
books_wc_array-34763686-3.out
books_wc_array-34763686-4.err
books_wc_array-34763686-4.out
```
Let's use `tail` and a `*` wildcard to check the last few lines of each of the output logs.

```bash
tail logs/books*.out
```

As expected, the output logs contain the results of the `wc -c` followed by
the book that was listed as input.

```output
==> logs/books_wc_array-34763686-0.out <==
174357 books/alice_in_wonderland.txt

==> logs/books_wc_array-34763686-1.out <==
5638516 books/complete_works_shakespeare.txt

==> logs/books_wc_array-34763686-2.out <==
1276288 books/moby_dick.txt

==> logs/books_wc_array-34763686-3.out <==
772419 books/pride_and_prejudice.txt

==> logs/books_wc_array-34763686-4.out <==
169541 books/romeo_and_juliet.txt
```


## Interactive Jobs: `srun`
Interactive jobs are great for development work if you need a better resourced machine for a couple hours or want to test
a Slurm configuration on a small sample before deploying it to a batch job.

Let's try an interactive job that requests the following resources:
- run on the `compute` partition
- requests 1 CPU
- requests 50MB of RAM
- runs for 10 minutes
- opens a `bash` terminal

```bash
srun --partition=compute --account=racs_training
--cpus-per-task=1 --mem=50M --time=10 --pty  bash
```

```output
srun: job 34703737 queued and waiting for resources
srun: job 34703737 has been allocated resources
```

You are now in a Bash terminal, but this terminal is no longer on the compute node.

```bash
hostname
```

```output
n0185.talapas.uoregon.edu
```

By default, your interactive jobs have maximum of 24 hours. Make sure to exit and cancel your job when you finish to free up resources,

```bash
exit
hostname
```

```output
login1.talapas.uoregon.edu
```

You'll noticed that the interactive job is marked as cancelled after using the `exit` command when checking 
the job id with the `sacct` command.

```bash
sacct -j 39457157 
```

```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
39457157           bash    compute racs_trai+          1  COMPLETED      0:0 
39457157.ex+     extern            racs_trai+          1  COMPLETED      0:0 
39457157.0         bash            racs_trai+          1  COMPLETED      0:0
```

Interactive jobs are also a great place to try out configuring GPU jobs. Let's be more polite and request only 30 minutes.
By default, interactive jobs run with a maximum of 4GB of RAM, so your job will be killed in either 30 minutes or
if you exceed our allotted RAM.

```bash
srun --partition=gpu --account=racs_training --time=30 --gpus=1 --constraint=gpu-10gb --pty bash
```

In this example constraint, you're requesting a GPU with 10GB of VRAM.

```output
srun: job 39459894 queued and waiting for resources
srun: job 39459894 has been allocated resources
```

Check that you're on the interactive node using `hostname`.

```bash
hostname
```

```output
n0162.talapas.uoregon.edu
```

Use the `nvidia-smi` command to get GPU information for the current node.


```bash
nvidia-smi
```

```output   
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.163.01             Driver Version: 550.163.01     CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100 80GB PCIe          On  |   00000000:E8:00.0 Off |                   On |
| N/A   37C    P0             72W /  300W |     829MiB /  81920MiB |     N/A      Default |
|                                         |                        |              Enabled |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| MIG devices:                                                                            |
+------------------+----------------------------------+-----------+-----------------------+
| GPU  GI  CI  MIG |                     Memory-Usage |        Vol|      Shared           |
|      ID  ID  Dev |                       BAR1-Usage | SM     Unc| CE ENC DEC OFA JPG    |
|                  |                                  |        ECC|                       |
|==================+==================================+===========+=======================|
|  0   12   0   0  |              13MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
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
This output indicates the GPU resources were allocated correctly.


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

This `nvidia-smi` command will fail in contexts where a GPU is unavailable or has not been allocated.

For example, if you run it on the login node, you'll get a message like this.

```bash
nvidia-smi
```

```output
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running
```

It prints this failure message because nodes without GPUs don't have a driver installed.

RACS has a guide 
on [configuring interactive jobs on Talapas](https://uoracs.github.io/talapas2-knowledge-base/docs/how-to_articles/how-to_start_an_interactive_job/) if you'd like to read more.

### Quick Slurm Scheduling Tips
* Read the resource descriptions carefully: jobs that exceed their request time, memory, or processor usage will be automatically killed.
* Need to run for more than 24 hours? You must use `computelong`, `gpulong`, or `memorylong`.
* Most jobs should have `--ntasks-per-node=1` `--cpus-per-task=1` if they do not reference tools or code that is explicitly multiprocessor code.
* Remember to exit interactive jobs when you're finished to free up resources for your colleagues.

## Today's Slurm Commands

| command | description | example usage |
| ----------- | --------------- | ------------- |
| sbatch [jobfile] | queues a Slurm job | `sbatch my_job.sh` |
| sacct | lists (your) recent jobs and subjobs | `sacct` |
| squeue -u [user] | gets status of running or queued slurm jobs for user | `squeue -u [yourDuckID]` |
| scancel [jobid] | cancels job jobid | `scancel [jobid]` |
| srun | launches interactive shell session on a compute node | `srun --partition=compute --account=racs_training --pty bash` |
| sinfo | lists information on available partitions | sinfo | 