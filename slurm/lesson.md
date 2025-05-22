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
This lesson introduces Slurm job configuration in detail.

## Lesson Setup
For this lesson, you will need to connect to a Talapas login node through a shell application of your choice.

For convenience, we recommend the [Talapas OnDemand shell](https://ondemand.talapas.uoregon.edu/pun/sys/shell/ssh/login1.talapas.uoregon.edu).

To start, make sure you are in your home directory.

```bash
cd ~
```

Copy the `/projects/racs_training/intro-hpc-s25/slurm` folder into your home directory. Don't forget `-r` to recursively copy the contents!

```bash
cp -r /projects/racs_training/intro-hpc-s25/slurm/ . 
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

## Troubleshooting a Slurm Job
Let's inspect `hello.sbatch`.

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

### Load needed modules
module purge
module load python3/3.11.4
module list

### Run your actual program
python3 hello.py
```
Let's parse this file using the comments as a guide! To learn more, see the [Slurm documentation](https://slurm.schedmd.com/sbatch.html) for `sbatch`.

* This job runs for a maximum of five minutes
(`0-00:05:00`) on one node and one CPU core of that node.
* It requests 500MB of RAM.
* The job output will be written `hello_world_python-[jobid].out` and `hello_world_python-[jobid].err` respectively.
* It loads the `python3/3.11.4` module.
* It lists the current modules loaded with `module list` and prints it to standard output.
* It runs a file in the `slurm_examples` directory called `hello.py` that prints to standard output. 

To submit your job file, run the `sbatch` command.

```bash
sbatch hello.sbatch
```

Look at your recent job history with `sacct`.

```bash
sacct
```

```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
34658037     hello_wor+    compute racs_trai+          1     FAILED      1:0 
34658037.ba+      batch            racs_trai+          1     FAILED      1:0 
34658037.ex+     extern            racs_trai+          1  COMPLETED      0:0
```

There's an error! Check the error log using `cat`.

```bash
cat hello_world_python-[YOURJOBID].err
```

```output

Currently Loaded Modules:
  1) miniconda-t2/20230523   2) python3/3.11.4

 
Traceback (most recent call last):
  File "/gpfs/home/emwin/part1/hello.py", line 2, in <module>
    print(dictionary['b']) # Should Raise KeyError !!!
          ~~~~~~~~~~^^^^^
KeyError: 'b'
```

This is [KeyError](https://docs.python.org/3/library/exceptions.html#KeyError) a common mistake when using Python dictionaries. 
Check the original Python file in a command line text editor like `nano`.

```bash
nano hello.py
```

Looking at the Python code, there's a bug inside `hello.py`.
```python
print("Hello World !!")

dictionary = {'a': 1}
print(dictionary['b']) # This should raise a KeyError
```
This can be fixed by checking the dictionary for a valid entry: `a`.
Change the last line to the following in `nano`, then use <kbd>Ctrl</kbd>+<kbd>O</kbd> and <kbd>Ctrl</kbd>+<kbd>X</kbd> 
to modify the file.

```python
print(dictionary['a']) # This will not raise a KeyError
```

Now that the file is saved, rerun your fixed job.

```bash
sbatch hello.sbatch
```

```output
Submitted batch job 34704219
```

Running `sacct` will show your job completed successfully.

```bash
sacct
```

```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
34704219     hello_wor+    compute racs_trai+          1  COMPLETED      0:0 
34704219.ba+      batch            racs_trai+          1  COMPLETED      0:0 
34704219.ex+     extern            racs_trai+          1  COMPLETED      0:0 
```

Check the output log to see the results of your job. You'll want to pick the *second* and later run's log. Note that because each log is named for the jobid, and jobids are unique, your logs will not overwrite each other.

```bash
1
Hello Wold !!
```

To check the status of your queued jobs, use the `squeue` command. Do not use squeue without an argument unless you want to see *all* the queued jobs on Talapas.

```bash
squeue --me
```

```output
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

Notice that because each job creates two log files based on the job id, you will have at least four log files (`%x-%j.err`, `%x-%j.out` ) in your current working directory.

## Cancelling Longer Jobs

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
#SBATCH --mem=500M                       ### Total Memory for job in MB -- can do K/M/G/T for KB/M
B/GB/TB
#SBATCH --ntasks-per-node=1              ### Number of tasks to be launched per Node
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task
### Run your actual program
for i in {1..100}
do
    echo "This is loop iteration $i"
    sleep 10
done
```

This job features a Bash for-loop will print one line of text and sleep for ten seconds 100 times, once for each iteration of the loop. 
This means it will run for 1000 seconds or for about 15 minutes.

Inspect the job in the queue.

```bash
squeue --me
```

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
34704248 computelo long_hel    emwin  R       0:03      1 n0136
```

Let's practice cancelling a slow job with `scancel`.

```bash
scancel 34704248
```

Looking at the queue again, it's now empty, even though the job would have otherwise run for longer.

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

When you cancel a job, Slurm will write an error message to the error file for that file.

```bash
cat long_hello_world-34704248.err
```

```output
slurmstepd: error: *** JOB 34704248 ON n0136 CANCELLED AT 2025-05-21T16:42:49 ***
```

### Optimization with `seff`

Want to know how much of the resources you requested a finished job used? Try the `seff` command.

```bash
seff 34700623
```

```output
Job ID: 34700623
Cluster: talapas
User/Group: emwin/uoregon
State: CANCELLED (exit code 0)
Cores: 1
CPU Utilized: 00:00:00
CPU Efficiency: 0.00% of 00:02:23 core-walltime
Job Wall-clock time: 00:02:23
Memory Utilized: 456.00 KB
Memory Efficiency: 0.09% of 500.00 MB
```

Adjust your resource requests accordingly after your job runs. Be courteous and jobs won't be deprioritized. 
Remember that your job's runtime is its **queue time + execution time**, so asking for too many resources can make your job run *slower*
because it will spend longer in the queue.

If your job runs out of memory, try doubling the amount you request.
If you job uses less memory than you expected, you can use `seff` to *lower* the amount of memory requested and have your job
schedule faster next time.

This job, which spends its CPU time "sleeping" or waiting, uses very little of its requested resources.

## Helper Scripts From RACS

Need to decide which nodes (and which partitions) are appropriate for your jobs? RACS has a script that shows available CPU and RAM on each node.

```bash
/packages/racs/bin/slurm-show-cpu-mem
```

This script shows the name of the node, the number of CPU cores on the node, and the amount of RAM (in MB) in that order for each node on Talapas.

```output
...
n0187 128 515048
n0188 128 515048
n0189 128 515048
n0190 128 515048
n0191 128 515048
n0192 128 515048
n0193 128 515048
...
```

Nodes on the compute partition typically have 128 CPU cores and 500GB of RAM.

To check which partition a node belongs to, find its name in the output of the `sinfo` command,

```bash
sinfo
```

As you can see, `n0191` is one of the nodes in the **compute** partition.
```output
compute              up 1-00:00:00      1   plnd n0195
compute              up 1-00:00:00      6  drain n[0112-0117]
compute              up 1-00:00:00     35    mix n[0111,0118-0135,0180-0194,0196]
```

You may also want to know what hardware is associated with each node.
```bash
/packages/racs/bin/slurm-show-features
```

This script shows the node name, processor type, processor architecture, process

Let's inspect `n0191` again.

```output
...
n0185 amd,milan,7713
n0186 amd,milan,7713
n0187 amd,milan,7713
n0188 amd,milan,7713
n0189 amd,milan,7713
n0190 amd,milan,7713
n0191 amd,milan,7713
n0192 amd,milan,7713
n0193 amd,milan,7713
...
```

Nodes with extra RAM are indicated with `mem-[amount]` tags.
```output
n0372 intel,broadwell,e7-4830,mem-1t
n0373 intel,broadwell,e7-4830,mem-1tb
n0374 intel,broadwell,e7-4830,mem-4tb
n0375 intel,broadwell,e7-4830,mem-4tb
n0376 intel,broadwell,e7-4830,mem-2tb
n0377 intel,broadwell,e7-4830,mem-2tb
n0378 intel,broadwell,e7-4830,mem-2tb
n0379 intel,broadwell,e7-4830,mem-2tb
```

Nodes with GPUs are also indicated in this list.
```output
n0149 amd,milan,7413,a100,gpu-10gb
n0150 amd,milan,7413,a100,gpu-40gb
n0151 amd,milan,7413,a100,gpu-10gb
n0152 amd,milan,7413,a100,3xgpu-80gb,no-mig
n0153 amd,milan,7413,a100,gpu-80gb,2xgpu-80gb,no-mig
n0154 amd,milan,7413,a100,gpu-80gb,2xgpu-80gb,no-mig
n0155 amd,milan,7413,a100,gpu-40gb
n0156 amd,milan,7413,a100,gpu-40gb
n0157 amd,milan,7413,a100,gpu-40gb
n0158 amd,milan,7413,a100,gpu-40gb
```
These indicators are called features.
We can use **constraints** to 
direct Slurm to schedule jobs on nodes with specific features (extra RAM, GPU, CPU architecture) based on code hardware requirements.

## GPU Jobs
Let's examine an example GPU job in detail: `gpu.sbatch`.

Only three partitions are compatible with GPUs:
* gpu
* gpulong
* interactivegpu

Each node in a GPU partition has **at most** 4 GPUS.

Open `gpu.sbatch` in nano and modify it as follows.

```bash
nano gpu.sbatch
```

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
#SBATCH --constraint=gpu-10gb
#SBATCH --mail-type=BEGIN,END
#SBATCH --mail-user=YOURDUCKID@uoregon.edu     

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

### Emailing Job Status 
Running a really long job? Tired of checking `squeue`?

SLURM can even email you when jobs reach certain states:

```bash
### #SBATCH --mail-type=BEGIN,END,FAIL      
### #SBATCH --mail-user=<duckID>@uoregon.edu
```

Edit these lines in your sbatch script with *your* DuckID. You should get an email from Slurm when the job begins and finishes (whether in success or failure.) 

Schedule the GPU job and wait for results in your email.

```bash
sbatch gpu.sbatch
```

```output
Submitted batch job 34702879
```

The `nvidia-smi` command gives you the status of GPUs on your system, if any. It will err out if there are no GPUs on the system.

```bash
cat gpu_hello_world-34702879.out
```

```output
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 550.54.15              Driver Version: 550.54.15      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100 80GB PCIe          On  |   00000000:E9:00.0 Off |                   On |
| N/A   39C    P0             74W /  300W |     792MiB /  81920MiB |     N/A      Default |
|                                         |                        |              Enabled |
+-----------------------------------------+------------------------+----------------------+
+-----------------------------------------------------------------------------------------+
| MIG devices:                                                                            |
+------------------+----------------------------------+-----------+-----------------------+
| GPU  GI  CI  MIG |                     Memory-Usage |        Vol|      Shared           |
|      ID  ID  Dev |                       BAR1-Usage | SM     Unc| CE ENC DEC OFA JPG    |
|                  |                                  |        ECC|                       |
|==================+==================================+===========+=======================|
|  0    9   0   0  |              12MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
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

The `MIG devices:` section of this output indicates that the job was scheduled on a 9728MiB (10GB) slice of a GPU with 80GB of total VRAM.
The 10GB constraint worked.


## Serial Processing: One Core and One Task at A Time
Navigate inside the `part2` folder using `cd`.

```bash
cd ../part2
```

Check that you see the following files inside with `ls`.

```bash
ls -F
```

```output
array.sbatch*  books/  books.sbatch*  hello.sbatch*  parallel_steps.sbatch*  random.py*  serial_steps.sbatch*  steps.sh*
```


Examine `serial_steps.sbatch` job with `cat.

```bash
cat serial_steps.sbatch
```

```bash
##!/bin/bash

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

This job runs a Bash for-loop. To learn more about Bash loops, [check out this quick tutorial](../talapas-scripting/loops-variables.html).

This script uses `srun` to create 10 sequential (serial) steps in one job. By default, each iteration (here `i = 1, 2, 3 ... 10`) must finish before the loop moves to the next.

This is because the `srun` denotes a step within a job.
A job with multiple `srun` commands will create a separate step (with the same jobid) in the `sacct`
for each step.

```bash
sbatch serial_steps.sbatch
```

```output
Submitted batch job 34704292
```

Check the queue for the status of your job.
```bash
squeue --me
```

```output
34704292   compute serial_s    emwin  R       0:23      1 n0135
```
As you can observe, the serial job is running and corresponds to exactly one row in the Slurm queue.

However, each step of serial steps (each subjob) is treated as a separaterow in the `sacct` command.,.

```bash
sacct
```

```output
34704292.0     steps.sh            racs_trai+          1  COMPLETED      0:0 
34704292.1     steps.sh            racs_trai+          1  COMPLETED      0:0 
34704292.2     steps.sh            racs_trai+          1  COMPLETED      0:0 
34704292.3     steps.sh            racs_trai+          1  COMPLETED      0:0 
34704292.4     steps.sh            racs_trai+          1  COMPLETED      0:0 
34704292.5     steps.sh            racs_trai+          1  COMPLETED      0:0 
34704292.6     steps.sh            racs_trai+          1  COMPLETED      0:0 
34704292.7     steps.sh            racs_trai+          1  COMPLETED      0:0 
34704292.8     steps.sh            racs_trai+          1    RUNNING      0:0
```

This job is on loop 8 of the serial steps.
34704292.9 can't begin until 34704292.8 finishes.
**Serial jobs require that steps be completed one at a time 
AND in a specific order.**

Let's look at the time spent on each stage in more detail using a specially configured `sacct` command. You can learn more about `sacct` [formatting on the Slurm documentation](https://slurm.schedmd.com/sacct.html).

```bash
sacct -u $(whoami) --units=G --format=JobID,jobname,MaxRSS,Start,Elapsed,State,ExitCode
```

```output
34704292.0     steps.sh          0 2025-05-21T16:45:47   00:00:11  COMPLETED      0:0 
34704292.1     steps.sh          0 2025-05-21T16:45:58   00:00:10  COMPLETED      0:0 
34704292.2     steps.sh          0 2025-05-21T16:46:08   00:00:10  COMPLETED      0:0 
34704292.3     steps.sh          0 2025-05-21T16:46:18   00:00:10  COMPLETED      0:0 
34704292.4     steps.sh          0 2025-05-21T16:46:28   00:00:10  COMPLETED      0:0 
34704292.5     steps.sh          0 2025-05-21T16:46:38   00:00:10  COMPLETED      0:0 
34704292.6     steps.sh          0 2025-05-21T16:46:48   00:00:10  COMPLETED      0:0 
34704292.7     steps.sh          0 2025-05-21T16:46:58   00:00:10  COMPLETED      0:0 
34704292.8     steps.sh          0 2025-05-21T16:47:08   00:00:10  COMPLETED      0:0 
34704292.9     steps.sh          0 2025-05-21T16:47:18   00:00:11  COMPLETED      0:0 
```
As you can see, all ten steps (numbered 0-9) are now marked as `COMPLETED.` Each subjob has a different *start* time.
The subjobs start roughly ten seconds apart, after the previous subjob has finished.

There are some jobs where certain steps or subcomponents must be completed in a serial fashion.
However, serial jobs do not take advantage of the computational power of Talapas or its support for parallelism.
Think of serial computation as a last resort when designing code on Talapas.

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
Submitted batch job 34704407
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
sacct -u $(whoami) --units=G --format=JobID,jobname,MaxRSS,Start,Elapsed,State,ExitCode
```

```output
JobID           JobName     MaxRSS               Start    Elapsed      State ExitCode 
34704407     parallel_+            2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.ba+      batch          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.ex+     extern          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.0     steps.sh          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.1     steps.sh      0.00G 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.2     steps.sh          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.3     steps.sh          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.4     steps.sh          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.5     steps.sh          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.6     steps.sh      0.00G 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.7     steps.sh      0.00G 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.8     steps.sh          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
34704407.9     steps.sh          0 2025-05-21T16:51:53   00:00:10  COMPLETED      0:0 
```

The parallel job ran in 11 seconds, significantly faster than the 70 second *serial* version of the job.
This is an example of an *embarassingly parallel* job, because each subjob is completely independent of the other subjobs.


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
srun python3 random.py $SLURM_ARRAY_TASK_ID $SLURM_ARRAY_JOB_ID $SLURM_JOB_ID
```

Array jobs require the `#SBATCH --array=` parameter.
In this example, the array parameter is the list `{0, 1, 2, ... 10}`.
The `%a` stands for the id of the *task* and the `%A` stands for the main array.

This will create one main job and a subjob for each index in the array. Each array index will inherit the
requested parameters specified in `cpus-per-task` and `ntasks-per-node`. This means each subjob asks for 1 CPU core.

This job runs `random_test.py` and passes in three arguments: the $SLURM_ARRAY_TASK_ID, the $SLURM_ARRAY_JOB_ID and $SLURM_JOB_ID. `$SLURM_ARRAY_TASK_ID $SLURM_ARRAY_JOB_ID $SLURM_JOB_ID` are environment variables.

The advantage of array jobs is that Slurm does not wait for all requested cores to be available
for the first subjobs of the array jobs to be scheduled. *This is not the case for parallel jobs that utilize `srun` to distribute
subjobs among multiple requested cores.*

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

This Python script takes three arguments (array_job_id, array_task_id, job_id) and and then prints three random numbers
This script will run once for each subjob in the array: 10 times.

Go ahead and submit the array job.

```bash
sbatch array.sbatch
```

```output
Submitted batch job 34704494
```

Array jobs are **parallel** jobs. 

You have no guarantee that all the array jobs will schedule or run at the same time.

Let's check progress the job's progress with `squeue`.

```bash
squeue --me
```

Unlike the paralllel job creatd with `srun`, each array subjob gets its own row in the results of `squeue`.

```output
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
34704494_1   compute python_a    emwin  R       0:01      1 n0135
34704494_2   compute python_a    emwin  R       0:01      1 n0135
34704494_3   compute python_a    emwin  R       0:01      1 n0135
34704494_4   compute python_a    emwin  R       0:01      1 n0135
34704494_5   compute python_a    emwin  R       0:01      1 n0135
34704494_6   compute python_a    emwin  R       0:01      1 n0135
34704494_7   compute python_a    emwin  R       0:01      1 n0135
34704494_8   compute python_a    emwin  R       0:01      1 n0135
34704494_9   compute python_a    emwin  R       0:01      1 n0135
34704494_10   compute python_a    emwin  R       0:01      1 n0135
```

Each *subjob* in an array job creates in own log.

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
...
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

## Parallelism vs. Serial Execution
Array jobs do not "talk to each other". They do not run in a guaranteed order, nor do they finish in a guaranteed order.

Regardless of how your job is configured, no two jobs or subjobs should not be used to write to or read from the same inputs concurrently.
This create **race conditions** in which your code and filesystem can behave unpredictably.

*Race conditions are why we make sure everyone in the class copies example files from shared directories to their home directory
before modifying them and executing them.*

## Interactive Jobs: `srun`
Interactive jobs are great for development work if you need a better resourced machine for a couple hours or want to test
a Slurm configuration on a small sample before deploying it to a batch job.

Let's try the simplest possible interactive job.

```bash
srun --partition=compute --account=racs_training --pty bash
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
n0135.talapas.uoregon.edu
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
sacct -j 34703737
```

```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
34703737           bash    compute racs_trai+          1  COMPLETED      0:0 
34703737.ex+     extern            racs_trai+          1  COMPLETED      0:0 
34703737.0         bash            racs_trai+          1  COMPLETED      0:0
```

Interactive jobs are also a great place to try out configuring GPU jobs. Let's be more polite and request only 30 minutes.
By default, interactive jobs run with a maximum of 4GB of RAM, so your job will be killed in either 30 minutes or
if you exceed our allotted RAM.

```bash
srun --partition=gpu --account=racs_training --time=30 --gpus=1 --constraint=gpu-10gb --pty bash
```

In this example constraint, you're requesting a GPU with 10GB of VRAM.

```output
srun: job 34704657 queued and waiting for resources
srun: job 34704657 has been allocated resources
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
| NVIDIA-SMI 550.54.15              Driver Version: 550.54.15      CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100 80GB PCIe          On  |   00000000:E8:00.0 Off |                   On |
| N/A   38C    P0             70W /  300W |     677MiB /  81920MiB |     N/A      Default |
|                                         |                        |              Enabled |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| MIG devices:                                                                            |
+------------------+----------------------------------+-----------+-----------------------+
| GPU  GI  CI  MIG |                     Memory-Usage |        Vol|      Shared           |
|      ID  ID  Dev |                       BAR1-Usage | SM     Unc| CE ENC DEC OFA JPG    |
|                  |                                  |        ECC|                       |
|==================+==================================+===========+=======================|
|  0   12   0   0  |              12MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
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
* Most jobs should have `--ntasks-per-node=1` and `--cpus-per-task=1` if they do not reference tools or code that is explicitly multiprocessor code.
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
