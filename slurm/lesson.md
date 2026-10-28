---
title: Lesson
layout: page
permalink: /slurm/lesson.html
parent: Running Jobs with Slurm
nav_enabled: true
nav_order: 2
---

# Running Jobs with Slurm
[Slurm](https://slurm.schedmd.com/slurm.html) is job scheduling software that apportions resources to jobs distributed across the hundreds of nodes on Talapas.

To use Talapas, you must request resources in the form of Slurm jobs. There are two types of Slurm jobs you can launch on the cluster: batch jobs and interactive jobs. This lesson demonstrates each one and describes how to configure them.

## Lesson Setup
Connect to a Talapas login node through a shell application of your choice. We recommend the [Talapas OnDemand shell](https://ondemand.talapas.uoregon.edu/pun/sys/shell/ssh/login2.talapas.uoregon.edu) for this exercise.

To start, make sure you are in your home directory.

```bash
cd ~
```

If you. haven't already, copy the `/projects/racs_training/intro-hpc-f26/slurm` folder into your home directory. Don't forget `-r` to recursively copy the contents!

```bash
cp -r /projects/racs_training/intro-hpc-f26/slurm/ . 
```

Navigate inside the `slurm` directory and inspect the contents.
```bash
cd slurm
ls -F
```
```output
alphafold3/  hello-world/  julia-set/
```

Navigate inside the `hello-world/single-node` folder using `cd`, then inspect its contents using `ls`.

```bash
cd hello-world/single-node
ls
```
```output
hello.srun
```

## Requesting Multiple CPU Cores with `ntasks`
Examine the `hello.srun` Slurm job with `cat`.
```bash
cat hello.srun
```
```bash
#!/bin/bash
#SBATCH --partition=compute           ### Partition (which queue you want to submit to)
#SBATCH --job-name=hello-world        ### Job Name
#SBATCH --output=hello.out            ### File in which to store job output
#SBATCH --error=hello.err             ### File in which to store job error mes
#SBATCH --time=0-00:00:59             ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                     ### Number of nodes needed for the job
#SBATCH --ntasks=10                   ### Total number of cores you want 
#SBATCH --account=racs_training       ### Account used for job submission (This will be your PIRG in the future) 

srun bash -c 'echo "Hello from task $SLURM_PROCID on $(hostname)"'
```

Let’s parse this file using the comments as a guide.

* This job runs for a maximum of 59 seconds (0-00:00:59) on one node.
* The job will be named `hello-world`.
* It runs on the `compute` partition, the best option for jobs without special hardware requirements.
* The job's output log and error log will be written to `hello.out` and `hello.err` respectively.
* It requests 10 tasks with the `#SBATCH --ntasks=10` parameter.
    - Unless otherwise specified, Slurm will allocate one CPU core per task.
    - This mean this job requests a total of 10 CPU cores on one node.
* It runs using the `racs_training` PIRG. Remember, you will need to use your own research group's PIRG after this session.
* Each task (launched by `srun`) is allocated a `SLURM_PROCID` that is printed to the output logs, followed by the name of node running the task.
* Normally, each task would get its own log. However, because there is exactly one output log, all ten tasks will write to the same file in turn.


Launch the job using `sbatch`.

```bash
sbatch hello.srun
``` 
```output
Submitted batch job 48130719
```

This job will finish quickly. Use `sacct` to confirm the job has completed.
```bash
sacct
```
```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
48130719     hello-wor+    compute racs_trai+         10  COMPLETED      0:0 
48130719.ba+      batch            racs_trai+         10  COMPLETED      0:0 
48130719.ex+     extern            racs_trai+         10  COMPLETED      0:0 
48130719.0         bash            racs_trai+         10  COMPLETED      0:0 
```
Note that the `.0` indicates the job step created by `srun`.

First inspect the error log. If it is empty, the job completed successfully.
```bash
cat hello.err
```

Now inspect the output log.
```bash
cat hello.out
```
```output
Hello from task 3 on n0117.talapas.uoregon.edu
Hello from task 4 on n0117.talapas.uoregon.edu
Hello from task 7 on n0117.talapas.uoregon.edu
Hello from task 5 on n0117.talapas.uoregon.edu
Hello from task 9 on n0117.talapas.uoregon.edu
Hello from task 8 on n0117.talapas.uoregon.edu
Hello from task 6 on n0117.talapas.uoregon.edu
Hello from task 0 on n0117.talapas.uoregon.edu
Hello from task 2 on n0117.talapas.uoregon.edu
Hello from task 1 on n0117.talapas.uoregon.edu
```

Observe that the ten, independent tasks ran on the same node. This is because Slurm jobs are single-node jobs by default.

Keep in mind when working with multiple tasks that...

* Tasks are independent and not launched in order.
* Copies of the same task will finish at different times.
* Never assume that your parallelized tasks will finish in a specific order.

## Multi-Node Jobs
Most jobs on Talapas do not require multiple nodes, and only [specialized software can take advantage of APIs required for nodes to communicate with each other](https://uoracs.github.io/talapas2-knowledge-base/docs/slurm/how-to_submit_an_mpi_job/#mpi-compilers). 

Navigate to the `multi-node` subfolder using `cd` and inspect the contents with `ls`.

```bash
cd ../multi-node/
ls
```
```output
hello.srun
```

Let's inspect the multi-node version of `hello.srun` with `cat`.

```bash
#!/bin/bash
#SBATCH --partition=compute       ### Partition (which queue)
#SBATCH --job-name=hello-multi    ### Job Name
#SBATCH --output=hello.out        ### File in which to store job output
#SBATCH --error=hello.err         ### File in which to store job error mes
#SBATCH --time=0-00:00:59         ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=5                 ### Number of nodes needed for the job
#SBATCH --ntasks-per-node=2       ### Number of tasks to be launched per Node
#SBATCH --account=racs_training   ### Account used for job submission

srun bash -c 'echo "Hello from task $SLURM_PROCID on $(hostname)"'
```

There are three key differences in this job configuration:

* This job requests 5 nodes with the `--nodes=5` parameter.
* It requests `--ntasks-per-node=2` for a total of 10 tasks distributed across 5 nodes.
* It has the name `hello-multi` to distinguish it from the single-node `hello.srun` job.


Launch the job with `sbatch`.

```bash
sbatch hello.srun
```
```output
Submitted batch job 48130866
```

Check that the job has completed with `sacct`.
```bash
sacct
```
```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- --------   
48130866     hello-mul+    compute racs_trai+         10  COMPLETED      0:0 
48130866.ba+      batch            racs_trai+          2  COMPLETED      0:0 
48130866.ex+     extern            racs_trai+         10  COMPLETED      0:0 
48130866.0         bash            racs_trai+         10  COMPLETED      0:0 
```

The job was allocated 10 CPU cores.

Inspect your output with `cat`.

```bash
cat hello.out
```
```output
Hello from task 1 on n0135.talapas.uoregon.edu
Hello from task 6 on n0188.talapas.uoregon.edu
Hello from task 7 on n0188.talapas.uoregon.edu
Hello from task 0 on n0135.talapas.uoregon.edu
Hello from task 4 on n0186.talapas.uoregon.edu
Hello from task 5 on n0186.talapas.uoregon.edu
Hello from task 2 on n0185.talapas.uoregon.edu
Hello from task 3 on n0185.talapas.uoregon.edu
Hello from task 8 on n0190.talapas.uoregon.edu
Hello from task 9 on n0190.talapas.uoregon.edus
```
As you can see, 10 tasks (numbered 0 through 9) were distributed evenly among the 5 nodes.

Use the `seff` command followed by the job number to evaluate the resource effeciency of the job.

```bash
seff 48130866
```
```output
Job ID: 48130866
Cluster: talapas
User/Group: emwin/uoregon
State: COMPLETED (exit code 0)
Nodes: 5
Cores per node: 2
CPU Utilized: 00:00:00
CPU Efficiency: 0.00% of 00:00:10 core-walltime
Job Wall-clock time: 00:00:01
Memory Utilized: 20.73 MB
Memory Efficiency: 0.05% of 40.00 GB (4.00 GB/core)
The task which had the largest memory consumption differs by 105.90% from the average task max memory consumption
```

Looking at the effeciency statistics of this job, this job would *not* justify requesting multiple nodes and so much RAM. 

## GPU Jobs
Next, let's examine the configuration of a GPU job. Because GPUs are specialized and expensive hardware, only four public partitions on Talapas have GPUs available:
* gpu
* gpulong
* interactivegpu
* preempt

Slurm will refuse to schedule any job that requests a GPU using the `--gpus=` parameter on a partition without GPUs available.

To demonstrate a GPU use case, we're going to use the GPU-powered protein-folding software [AlphaFold3](https://blog.google/innovation-and-ai/products/google-deepmind-isomorphic-alphafold-3-ai-model/).

Navigate into the `alphafold3` example folder and inspect the contents.

```bash
cd ../../alphafold3
ls
```
```output
gpu-constraints  gpu-job
```

We'll start with the default `gpu-job` example before delving into using GPU-related constraints.

```bash
cd gpu-job
ls
```
```output
2pv7-test.json  alphafold3.srun
```

* `alphafold3.srun` is a batch job that uses AlphaFold3 and requires a GPU.
* `2pv7-test.json` is a metadata file that tells AlphaFold3 how to run. While you don't need to understand the contents of the file for our test, actual researchers would manipulate this `.json` file to instruct AlphaFold.

Inspect `alphafold3.srun` with `cat`.

```bash
cat alphafold3.srun
```
```bash
#!/bin/bash
#SBATCH --partition=gpu        
#SBATCH --job-name=af3-gpu-job-test    
#SBATCH --output=af3.out       
#SBATCH --error=af3.err        
#SBATCH --time=0-03:00:00 
#SBATCH --nodes=1          
#SBATCH --ntasks-per-node=8
#SBATCH --gpus=1 ### ALWAYS specify a number of GPUs even if it is 1
#SBATCH --mem=10g
#SBATCH --account=racs_training 

module purge
module load alphafold3/3.0.4
WORKING_DIR=$PWD
alphafold3 \
    --json_path="$WORKING_DIR/2pv7-test.json" \
    --output_dir="$WORKING_DIR"
```

Let's unpack this job:

* It is scheduled on the `gpu` partition.
* It requests exactly one GPU.
    - Slurm will not allocate you a GPU on the `gpu` partition if you do not set the `--gpus` parameter.
* It loads the `alphafold3` module.
* It runs `alphafold3` using the `2pv7-test.json` config file and outputs the results in the current directory.

Launch the job with `sbatch`.

```bash
sbatch alphafold3.srun
```
```output
Submitted batch job 48133618
```

Use `squeue --me` to find the name of the node where the job was scheduled.

```bash
squeue --me
```
```output
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          48133618       gpu af3-gpu-    emwin  R       0:04      1 n0149
```

Let's examine the job in action and make sure the GPU we requested is being used. Remote to the compute node identified your queue using `ssh` followed by the node name.

```bash
ssh n0149
```

* This command will only succeed if you have an active job running on that particular node.

Run `nvidia-smi` to check activity on the GPU.

```bash
nvidia-smi
```
```output
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.178.04             Driver Version: 580.178.04     CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA A100 80GB PCIe          On  |   00000000:E8:00.0 Off |                   On |
| N/A   35C    P0             64W /  300W |    9586MiB /  81920MiB |     N/A      Default |
|                                         |                        |              Enabled |
+-----------------------------------------+------------------------+----------------------+
|   1  NVIDIA A100 80GB PCIe          On  |   00000000:E9:00.0 Off |                   On |
| N/A   33C    P0             43W /  300W |     249MiB /  81920MiB |     N/A      Default |
|                                         |                        |              Enabled |
+-----------------------------------------+------------------------+----------------------+
|   2  NVIDIA A100 80GB PCIe          On  |   00000000:EA:00.0 Off |                   On |
| N/A   33C    P0             44W /  300W |     249MiB /  81920MiB |     N/A      Default |
|                                         |                        |              Enabled |
+-----------------------------------------+------------------------+----------------------+
|   3  NVIDIA A100 80GB PCIe          On  |   00000000:EB:00.0 Off |                   On |
| N/A   37C    P0             47W /  300W |     249MiB /  81920MiB |     N/A      Default |
|                                         |                        |              Enabled |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| MIG devices:                                                                            |
+------------------+----------------------------------+-----------+-----------------------+
| GPU  GI  CI  MIG |              Shared Memory-Usage |        Vol|        Shared         |
|      ID  ID  Dev |                Shared BAR1-Usage | SM     Unc| CE ENC  DEC  OFA  JPG |
|                  |                                  |        ECC|                       |
|==================+==================================+===========+=======================|
|  0    7   0   0  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  0    8   0   1  |            9373MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  0    9   0   2  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  0   11   0   3  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  0   12   0   4  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  0   13   0   5  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  0   14   0   6  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  1    7   0   0  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  1    8   0   1  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  1    9   0   2  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  1   10   0   3  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  1   11   0   4  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  1   12   0   5  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  1   13   0   6  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  2    7   0   0  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  2    8   0   1  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  2    9   0   2  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  2   10   0   3  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  2   11   0   4  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  2   12   0   5  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  2   13   0   6  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  3    7   0   0  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  3    8   0   1  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  3    9   0   2  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  3   11   0   3  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  3   12   0   4  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  3   13   0   5  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+
|  3   14   0   6  |              36MiB /  9728MiB    | 14      0 |  1   0    0    0    0 |
|                  |               0MiB /  6135MiB    |           |                       |
+------------------+----------------------------------+-----------+-----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0    8    0           460298      C   python                                 9330MiB |
+-----------------------------------------------------------------------------------------+
```
The `python` process launched by AlphaFold is currently using 9330MB of VRAM.

Once you've confirmed your job is running on the GPU, exit the node. Confirm you are back on a login node with the `hostname` command.

```bash
exit
```
```bash
hostname
```
```output
login2.talapas.uoregon.edu
```

Because we only launched the AlphaFold3 job for testing purposes, we want to cancel the job before it finishes to free up resources. 

The [scancel](https://slurm.schedmd.com/scancel.html) command followed by a job id will cancel an active Slurm job.

```bash
scancel 48133618
```

Check your queue with `squeue` to confirm the job was canceled.

```bash
squeue --me
```
```output
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```

## Job Constraints
Now, let's examine the `gpu-constraints` example.

```bash
cd ../gpu-constraints/
ls
```
```output
2pv7-test.json  alphafold3.srun
```

Like before, we will ignore the `.json` config in favor of examining the `alphafold3.srun` batch job.

```bash
cat alphafold3.srun
```
```bash
#!/bin/bash
#SBATCH --partition=gpu        
#SBATCH --job-name=af3-constraint-test    
#SBATCH --output=af3.out       
#SBATCH --error=af3.err        
#SBATCH --time=0-03:00:00 
#SBATCH --ntasks-per-node=1
#SBATCH --gpus=1
#SBATCH --mem=10g
#SBATCH --constraint=gpu-10gb ### Add a constraint to request a specific architecture   
#SBATCH --account=racs_training 

module purge
module load alphafold3/3.0.4
WORKING_DIR=$PWD
alphafold3 \
    --json_path="$WORKING_DIR/2pv7-test.json" \
    --output_dir="$WORKING_DIR"
```

This job is slightly different: it introduces a resource constraint with `--constraint`. 

If your code has specific hardware constraints, the `--constraint` parameter allows you to specify certain nodes within a partition based on constraints defined by RACS. 

* In this case `--constraint=gpu-10gb` restricts the job to nodes with the `gpu-10gb` feature.
* This means the job will be allocated to either a GPU with 10GB of VRAM or allocated to a [MIG slice](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/latest/concepts.html#partitioning) of a larger GPU. For example, this could mean a 10GB slice of an NVIDIA A100 with 80GB of VRAM.

### Identifying Candidate Job Constraints
To get a list of potential job constraints, use the following command.
```bash
sinfo -o "%25N %50f 
```
```output
NODELIST                  AVAIL_FEATURES                                                                                      
n[0049-0050,0052-0096,020 intel,broadwell,e5-2690                                                                             
n[0111-0136,0173-0196]    amd,milan,7713                                                                                      
n[0150-0151,0155-0158,016 amd,milan,7413,a100,gpu-40gb                                                                        
n[0152-0154,0159-0160,016 amd,milan,7413,a100,gpu-80gb,no-mig                                                                 
n0301                     amd,rome,7542,a100,gpu-40gb,gpu-80gb,no-mig                                                         
n[0149,0161-0163]         amd,milan,7413,a100,gpu-10gb                                                                        
n[0302-0313,0398-0399]    intel,broadwell,e5-2690v4                                                                           
n[0141-0142]              intel,icelake,6348,avx512,mem-4tb                                                                   
n[0143-0146]              intel,icelake,6348,avx512,mem-2tb                                                                   
n[0147-0148,0266-0267]    intel,icelake,6348,avx512,mem-1tb                                                                   
n[0374-0375]              intel,broadwell,e7-4830,mem-4tb                                                                     
n[0376-0379]              intel,broadwell,e7-4830,mem-2tb                                                                     
n[0372-0373]              intel,broadwell,e7-4830,mem-1tb                                                                     
n0197                     intel,sapphirerapids,6448y,avx512,mem-1tb,l40s,gpu-48gb                                             
n0996                     amd,turin,9555,avx512,mem-1tb,h200,gpu-141gb,no-mig                                                 
n[0045-0046]              amd,genoa,9354p,avx512,a40,gpu-48gb                                                                 
n[0097-0098]              amd,turin,9655p,avx512                                                                              
n0105                     amd,milan,7763,mem-2tb                                                                              
n0106                     intel,emeraldrapids,6548y,avx512,mem-1tb,l40s,gpu-48gb                                              
n0108                     intel,graniterapids,6737p,avx512,rtx6000pro,gpu-96gb,no-mig                                         
n0109                     intel,icelake,6338n,avx512                                                                          
n[0221-0226,0229,0317-033 intel,cascadelake,6248,avx512                                                                       
n[0268-0269]              intel,icelake,6342,avx512                                                                           
n[0270,0368-0371]         amd,milan,7413                                                                                      
n0316                     amd,milan,7543,a100,gpu-40gb,no-mig                                                                 
n[0332-0335]              amd,milan,7513                                                                                      
n[0336,0385-0387]         intel,cascadelake,6248,avx512,v100,gpu-32gb                                                         
n0363                     amd,milan,7713,a100,gpu-80gb,no-mig                                                                 
n0364                     intel,cascadelake,8280l,avx512,mem-6tb                                                              
n0380                     amd,milan,7413,mem-1tb                                                                              
n[0997-0999]              intel,sapphirerapids,6442y,avx512,mem-1tb,h100,gpu-80gb,no-mig                                      
n1000                     intel,sapphirerapids,8480cl,avx512,mem-2tb,h100,gpu-80gb,no-mig                                     
n[0037-0044,0213-0216,021 intel,skylake,6148,avx512                                                                           
n0107                     intel,graniterapids,6737p,avx512,mem-1tb,h200,gpu-141gb,no-mig                                      
n0110                     amd,genoa,9354p,avx512,l40,gpu-48gb                                                                 
n[0314-0315]              amd,milan,7543,a100,gpu-20gb                                                                        
n[0390-0397]              intel,cascadelake,6230,avx512,tmp-9tb
```


* Some features relate to the CPU, like `amd`, `milan`, and `cascadelake`.
* Others relate to the GPU, if any, like `gpu-80gb` and `no-mig`.
* Only request special constraints if your software or workflow requires them. 
    - Your job will schedule faster (and better share resources) if you only ask for what you need.

## Array Jobs
Slurm has a special option for bulk scheduling of nearly identical jobs: [array jobs](https://slurm.schedmd.com/job_array.html).

Let's navigate to the `array` folder of the `julia-set` example.

```bash
cd ~/slurm/julia-set/array
ls
```
```output
images  julia.py  julia.srun  logs
```

Inspect this version of the `julia.py` file.
```bash
cat julia.py
```
```python
import numpy as np
import matplotlib.pyplot as plt
import sys


def generate_julia(width=1000, height=1000, zoom=1, move_x=0, move_y=0,
                       c_real=-0.7, c_imag=0.27015, max_iter=300,
                       output_file="julia.png"):
    """
    Generate and save a Julia set fractal image.

    Parameters:
        width, height (int): Image dimensions in pixels.
        zoom (float): Zoom factor (>0).
        move_x, move_y (float): Pan offsets.
        c_real, c_imag (float): Complex constant defining the Julia set.
        max_iter (int): Maximum iterations for escape time algorithm.
        output_file (str): Filename to save the image.
    """
    # Validate parameters
    if width <= 0 or height <= 0:
        raise ValueError("Image dimensions must be positive integers.")
    if zoom <= 0:
        raise ValueError("Zoom factor must be positive.")
    if max_iter <= 0:
        raise ValueError("max_iter must be a positive integer.")

    # Create a complex grid
    x = np.linspace(-1.5, 1.5, width) * zoom + move_x
    y = np.linspace(-1.5, 1.5, height) * zoom + move_y
    X, Y = np.meshgrid(x, y)
    Z = X + 1j * Y

    # Julia set calculation
    C = complex(c_real, c_imag)
    img = np.zeros(Z.shape, dtype=int)

    for i in range(max_iter):
        mask = np.abs(Z) < 4
        Z[mask] = Z[mask] ** 2 + C
        img[mask] = i

    # Plot and save
    plt.figure(figsize=(8, 8))
    plt.imshow(img, cmap="inferno", extent=(-1.5, 1.5, -1.5, 1.5))
    plt.axis("off")
    plt.tight_layout()
    plt.savefig("images/" + output_file, dpi=300, bbox_inches='tight', pad_inches=0)
    plt.close()

    print("Julia set image saved as images/" + output_file)
    
if __name__ == "__main__":
    try:
        # Example: Generate a Julia set with parameters
        generate_julia(width=1000, height=1000, zoom=1, move_x=0, move_y=0,
                       c_real=float(sys.argv[1]), c_imag=float(sys.argv[2]), max_iter=300,
                       output_file=""+sys.argv[3])
    except Exception as e:
        print(f"Error: {e}")
```

* At the bottom, you can see that this job takes in arguments from the command line.
* This job takes parameters at run time.

Inspect the `julia.srun` batch job.

```bash
cat julia.srun 
```
```bash
#!/bin/bash
#SBATCH --partition=compute              ### Partition
#SBATCH --job-name=julia_test            ### Job Name
#SBATCH --output=logs/output_%A_%a.out   ### Unique output per task
#SBATCH --error=logs/error_%A_%a.err     ### Unique error log per task
#SBATCH --time=0-03:00:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --ntasks=1                       ### Number of tasks/cores (There's a lot of different ways to do this in SLURM!)
#SBATCH --mem=2g                         ### Total memory for the job
#SBATCH --account=racs_training          ### Account used for job submission
#SBATCH --array=0-11                     ### Each array job runs with the above 

module purge
module load python3/3.11.4
index=$(echo $SLURM_ARRAY_TASK_ID | bc) # Convert the Task ID to a float

filename="julia${index}.png" # Name file based off the task ID to keep them straight

x_cor=$(echo "$index * 0.2 - 1" | bc)   # -1.0 to 1.0
y_cor=$(echo "$index * 0.2 - 1" | bc)   # -1.0 to 1.0
# Move (x, y) coordinates up diagonally based on index: (-1, -1), (-0.8, -0.8), ... (1, 1)

python3 julia.py "$x_cor" "$y_cor" "$filename"
```

* `--array=` indicates this is an array job, with `0-11` indicating the range of array job ids to be created.
* Each job in the array gets its own log in the `logs` folders.
    * `%A` = Thread number, `%a` = job number.

Launch the job with `sbatch`.
```bash
sbatch julia.srun
```
```output
Submitted batch job 48028038
```

Use `sacct` to check the job's status.
```bash
sacct
```
```output
48028038_0   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_0.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_0.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_1   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_1.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_1.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_2   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_2.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_2.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_3   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_3.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_3.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_4   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_4.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_4.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_5   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_5.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_5.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_6   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_6.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_6.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_7   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_7.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_7.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_8   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_8.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_8.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_9   julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_9.+      batch            racs_trai+          1    RUNNING      0:0 
48028038_9.+     extern            racs_trai+          1    RUNNING      0:0 
48028038_10  julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_10+      batch            racs_trai+          1    RUNNING      0:0 
48028038_10+     extern            racs_trai+          1    RUNNING      0:0 
48028038_11  julia_test    compute racs_trai+          1    RUNNING      0:0 
48028038_11+      batch            racs_trai+          1    RUNNING      0:0 
48028038_11+     extern            racs_trai+          1    RUNNING      0:0 
```

Once the job finishes, check one of the output logs with `cat`.

```bash
cat logs/output*9.out
```
```output
Julia set image saved as images/julia9.png
```

We can use a different wildcard expression to inspect all of the logs simultaneously.

```bash
cat logs/output*.out
```
```output
Julia set image saved as images/julia0.png
Julia set image saved as images/julia10.png
Julia set image saved as images/julia11.png
Julia set image saved as images/julia1.png
Julia set image saved as images/julia2.png
Julia set image saved as images/julia3.png
Julia set image saved as images/julia4.png
Julia set image saved as images/julia5.png
Julia set image saved as images/julia6.png
Julia set image saved as images/julia7.png
Julia set image saved as images/julia8.png
Julia set image saved as images/julia9.png
```

### A More Complex Array Job: 100 Julia Fractals

With a little math, we can use the `array=` parameter to launch one hundred concurrent jobs that run the same Python script with different hyperparameters. 

Let's say we want to use to create different Julia fractals at different positions within a 10 by 10 grid.

Navigate to the `advanced_array` folder with `cd` and inspect the folder's contents.

```bash
cd ../advanced_array
ls
```
```output
images  julia.py  julia.srun
```

Examine this version of the `julia.py` fractal generator.
```bash
cat julia.py
```
```python

import numpy as np
import matplotlib.pyplot as plt
import sys

def generate_julia(width=1000, height=1000, zoom=1, move_x=0, move_y=0,
                       c_real=-0.7, c_imag=0.27015, max_iter=300,
                       output_file="julia.png"):
    """
    Generate and save a Julia set fractal image.

    Parameters:
        width, height (int): Image dimensions in pixels.
        zoom (float): Zoom factor (>0).
        move_x, move_y (float): Pan offsets.
        c_real, c_imag (float): Complex constant defining the Julia set.
        max_iter (int): Maximum iterations for escape time algorithm.
        output_file (str): Filename to save the image.
    """
    # Validate parameters
    if width <= 0 or height <= 0:
        raise ValueError("Image dimensions must be positive integers.")
    if zoom <= 0:
        raise ValueError("Zoom factor must be positive.")
    if max_iter <= 0:
        raise ValueError("max_iter must be a positive integer.")

    # Create a complex grid
    x = np.linspace(-1.5, 1.5, width) * zoom + move_x
    y = np.linspace(-1.5, 1.5, height) * zoom + move_y
    X, Y = np.meshgrid(x, y)
    Z = X + 1j * Y

    # Julia set calculation
    C = complex(c_real, c_imag)
    img = np.zeros(Z.shape, dtype=int)

    for i in range(max_iter):
        mask = np.abs(Z) < 4
        Z[mask] = Z[mask] ** 2 + C
        img[mask] = i

    # Plot and save
    plt.figure(figsize=(8, 8))
    plt.imshow(img, cmap="inferno", extent=(-1.5, 1.5, -1.5, 1.5))
    plt.axis("off")
    plt.tight_layout()
    plt.savefig("images/" + output_file, dpi=300, bbox_inches='tight', pad_inches=0)
    plt.close()

    print("Julia set image saved as images/" + output_file)
    
if __name__ == "__main__":
    try:
        # Example: Generate a Julia set with parameters
        generate_julia(width=1000, height=1000, zoom=1, move_x=0, move_y=0,
                       c_real=float(sys.argv[1]), c_imag=float(sys.argv[2]), max_iter=300,
                       output_file=""+sys.argv[3])
    except Exception as e:
        print(f"Error: {e}")
```

Examine `julia.srun` with `cat.
```bash
cat julia.srun
```
```bash
#!/bin/bash
#SBATCH --partition=compute     ### Partition (like a queue in PBS)
#SBATCH --job-name=julia_test    ### Job Name
#SBATCH --output=j.out         ### File in which to store job output
#SBATCH --error=j.err          ### File in which to store job error messages
#SBATCH --time=0-03:00:00       ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1               ### Number of nodes needed for the job
#SBATCH --ntasks-per-node=1     ### Number of tasks to be launched per Node
#SBATCH --mem=2g
#SBATCH --account=racs_training          ### Account used for job submission
#SBATCH --array=0-99
#SBATCH --open-mode=append

module purge
module load python3/3.11.4
index=$(echo $SLURM_ARRAY_TASK_ID | bc)
filename="julia${index}.png"

x_id=$(( $index % 10 )) # index remainder 10 to count horizontally
y_id=$(( $index / 10 )) # index divided by 10 (truncated/floored) to count up vertically

x_cor=$(echo "$x_id * 0.2 - 0.9" | bc -l)
y_cor=$(echo "$y_id * 0.2 - 0.9" | bc -l)
# Running from 0 -> 99 will run for (-0.9, -0.9), (-0.7, -0.9), ..., (-0.9, -0.7), (-0.7, -0.7), ..., (0.8, 0.8)  

python3 julia.py "$x_cor" "$y_cor" "$filename"
```

This version of the `julia.srun` job requests 100 array jobs and uses a mathematical trick with division to derive 100 coordinate pairs to pass to `julia.py` as arguments.

Launch the job with `sbatch`.
```bash
sbatch julia.srun
```
```output
Submitted batch job 48150477
```

Confirm the jobs completed successfully with `sacct`.

```bash
sacct
```
```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode
------------ ---------- ---------- ---------- ---------- ---------- --------
48150477_0   julia_test    compute racs_trai+          1  COMPLETED      0:0 
48150477_0.+      batch            racs_trai+          1  COMPLETED      0:0 
48150477_0.+     extern            racs_trai+          1  COMPLETED      0:0 
48150477_1   julia_test    compute racs_trai+          1  COMPLETED      0:0 
48150477_1.+      batch            racs_trai+          1  COMPLETED      0:0 
48150477_1.+     extern            racs_trai+          1  COMPLETED      0:0 
48150477_2   julia_test    compute racs_trai+          1  COMPLETED      0:0 
48150477_2.+      batch            racs_trai+          1  COMPLETED      0:0 
48150477_2.+     extern            racs_trai+          1  COMPLETED      0:0 
48150477_3   julia_test    compute racs_trai+          1  COMPLETED      0:0 
48150477_3.+      batch            racs_trai+          1  COMPLETED      0:0 
48150477_3.+     extern            racs_trai+          1  COMPLETED      0:0 
48150477_4   julia_test    compute racs_trai+          1  COMPLETED      0:0 
48150477_4.+      batch            racs_trai+          1  COMPLETED      0:0 
...
```


Inspect the fractals generated by the array jobs are available in the `images` folder with `ls`.

```bash
ls images/
```
```output
julia0.png   julia21.png  julia33.png  julia45.png  julia57.png  julia69.png  julia80.png  julia92.png
julia10.png  julia22.png  julia34.png  julia46.png  julia58.png  julia6.png   julia81.png  julia93.png
julia11.png  julia23.png  julia35.png  julia47.png  julia59.png  julia70.png  julia82.png  julia94.png
julia12.png  julia24.png  julia36.png  julia48.png  julia5.png   julia71.png  julia83.png  julia95.png
julia13.png  julia25.png  julia37.png  julia49.png  julia60.png  julia72.png  julia84.png  julia96.png
julia14.png  julia26.png  julia38.png  julia4.png   julia61.png  julia73.png  julia85.png  julia97.png
julia15.png  julia27.png  julia39.png  julia50.png  julia62.png  julia74.png  julia86.png  julia98.png
julia16.png  julia28.png  julia3.png   julia51.png  julia63.png  julia75.png  julia87.png  julia99.png
julia17.png  julia29.png  julia40.png  julia52.png  julia64.png  julia76.png  julia88.png  julia9.png
julia18.png  julia2.png   julia41.png  julia53.png  julia65.png  julia77.png  julia89.png
julia19.png  julia30.png  julia42.png  julia54.png  julia66.png  julia78.png  julia8.png
julia1.png   julia31.png  julia43.png  julia55.png  julia67.png  julia79.png  julia90.png
julia20.png  julia32.png  julia44.png  julia56.png  julia68.png  julia7.png   julia91.png
```

You have now generated 100 fractals, one for each combination of the `x, y` coordinates stored in the `x_cor` and `y_cor` variables. 

You can generalize this technique to other applications like batch processing, hyperparameter optimization tasks, or testing different combinations of model parameters.

### When to Use Array Jobs
If you can format your job as an array job, do so! Array jobs are the **preferred way** to parallelize your embarassingly parallel jobs. 
Array jobs are easy for the scheduler to manage. 

### Parallelism Tips with Array Jobs
Array jobs do not run in a guaranteed order, nor do they finish in a guaranteed order.

Regardless of how your job is configured, no two jobs or subjobs should not be used to write to the same outputs or read from the same inputs concurrently.
This create **race conditions** in which your code and filesystem can behave unpredictably.

## Interactive Jobs with `salloc`
Interactive jobs are great for development work if you need a better resourced machine for a couple hours or want to test
a Slurm configuration on a small sample before deploying it to a batch job.

You can use the command [salloc](https://slurm.schedmd.com/salloc.html) to jump directly to a compute node through an interactive job. Observe commands to launch this job use the same parameters as an SBATCH job.

Let's try the simplest possible interactive job, which requires only an account (PIRG) and a valid partition.

* If `--mem=` isn't specified, the job will default to 4GB of RAM and the maximum time limit of the requested partition.
* The default time limit of an interactive job depends on the partition, so we recommend being specific.
* The default number of nodes, cores and tasks is 1.

Keep in mind that the time limit of an interactive job can *not* be extended, so save your work frequently.

```bash
salloc --account=racs_training --partition=interactive
```
```output
salloc: Pending job allocation 48130331
salloc: job 48130331 queued and waiting for resources
salloc: job 48130331 has been allocated resources
salloc: Granted job allocation 48130331
salloc: Nodes n0302 are ready for job
```

Once the allocation is granted, a Bash terminal on a compute node will open. Use `hostname` to confirm.s
```bash
hostname
```
```output
n0302.talapas.uoregon.edu
```

Use the `scontrol show jobs` command followed by the job id to get more information about the resources allocated to your job.
```bash
scontrol show jobs 48130331
```
```output
JobId=48130331 JobName=interactive
   UserId=emwin(274700) GroupId=uoregon(131) MCS_label=N/A
   Priority=54195 Nice=0 Account=racs_training QOS=normal
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=0 Restarts=0 BatchFlag=0 Reboot=0 ExitCode=0:0
   RunTime=00:03:10 TimeLimit=12:00:00 TimeMin=N/A
   SubmitTime=2026-10-27T10:05:56 EligibleTime=2026-10-27T10:05:56
   AccrueTime=2026-10-27T10:05:56
   StartTime=2026-10-27T10:05:57 EndTime=2026-10-27T22:05:57 Deadline=N/A
   PreemptEligibleTime=2026-10-27T10:05:57 PreemptTime=None
   SuspendTime=None SecsPreSuspend=0 LastSchedEval=2026-10-27T10:05:57 Scheduler=Backfill
   Partition=interactive AllocNode:Sid=login2:4083992
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=n0302
   BatchHost=n0302
   NumNodes=1 NumCPUs=1 NumTasks=1 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   ReqTRES=cpu=1,mem=4G,node=1,billing=1
   AllocTRES=cpu=1,mem=4G,node=1,billing=1
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:* CoreSpec=*
   MinCPUsNode=1 MinMemoryCPU=4G MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   OverSubscribe=OK Contiguous=0 Licenses=(null) LicensesAlloc=(null) Network=(null)
   Command=/bin/sh
   SubmitLine=salloc --account=racs_training --partition=interactive
   WorkDir=/gpfs/home/emwin
```

To unpack this:

* This job is running on the `interactive` partition.
* It defaulted to 4G of RAM.
* It's running on one node: `n0302`.
* It uses the default time limit on the `interactive` partition of 12 hours.

Let's say you want to debug an R script on Talapas before deploying it at scale as a batch job. You can load modules in an interactive job just as you would in a batch job.

```bash
module load R/4.6.1
```

Launch an R interpreter with the `R`.

```bash
R
```

After typing a few test commands, type `quit()` to exit R.
```R
quit()
```

To exit an interactive job, type `exit`.

You will see a message that indicates you have relinquished the resources for that job.

```bash
exit
```
```output
salloc: Relinquishing job allocation 48130331
```

Run `sacct`. You will see the interactive job you requested marked as `COMPLETED` in your job history.

```bash
sacct
```
```output
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
48130331     interacti+ interacti+ racs_trai+          1  COMPLETED      0:0 
48130331.in+ interacti+            racs_trai+          1  COMPLETED      0:0 
48130331.ex+     extern            racs_trai+          1  COMPLETED      0:0
```

### Debugging GPU Allocation with Interactive Jobs
Interactive jobs are also a great way to test GPU jobs. Let's be more polite and request only 30 minutes.


```bash
salloc --partition=gpu --account=racs_training --time=30 --gpus=1 --constraint=gpu-10gb
```

In this example, the `--constraint=gpu-10g` flag requests a GPU (or slice of a GPU) with 10GB of VRAM.

```output
srun: job 34704657 queued and waiting for resources
srun: job 34704657 has been allocated resources
```

Check that you're on the interactive node using `hostname`. No `ssh` step is required for interactive jobs.

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

If you requested GPU resources correctly, you should be able to run `nvidia-smi` and see information about the GPU available to you. This output indicates the GPU resources were allocated correctly.


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

It prints this failure message because nodes without GPUs don't have an NVIDIA driver installed.

RACS has a guide 
on [configuring interactive jobs on Talapas](https://uoracs.github.io/talapas2-knowledge-base/docs/how-to_articles/how-to_start_an_interactive_job/) if you'd like to read more.

## Optimization with `seff`

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

* If your job runs out of memory, try doubling the amount you request.
* If you job uses less memory than you expected, you can use `seff` to determine how much memory to request and have your job schedule faster next time.

This job, which spends its CPU time "sleeping" or waiting, uses very little of its requested resources.

### Emailing Job Status 
Running a really long job? Tired of checking `squeue`?

SLURM can even email you when jobs reach certain states:

```bash
### #SBATCH --mail-type=BEGIN,END,FAIL      
### #SBATCH --mail-user=<duckID>@uoregon.edu
```

Edit these lines in your sbatch script with *your* DuckID. You should get an email from Slurm when the job begins and finishes (whether in success or failure.) 

### Quick Slurm Scheduling Tips

* Read the resource descriptions carefully: jobs that exceed their request time, memory, or processor usage will be automatically killed.
* Need to run for more than 24 hours? You must use `computelong`, `gpulong`, or `memorylong`.
* Most jobs should have `--ntasks-per-node=1` and `--cpus-per-task=1` if they do not reference tools that take advantage of multiple CPU cores.
* Remember to exit interactive jobs when you're finished to free up resources for your colleagues.

## Today's Slurm Commands

| command | description | example usage |
| ----------- | --------------- | ------------- |
| sbatch [jobfile] | queues a Slurm job | `sbatch my_job.sh` |
| sacct | lists (your) recent jobs and subjobs | `sacct` |
| squeue -u [user] | gets status of running or queued slurm jobs for user | `squeue -u [yourDuckID]` |
| scancel [jobid] | cancels job jobid | `scancel [jobid]` |
| salloc | launches an interactive job on a compute node | `salloc --partition=compute --account=racs_training` |
| scontrol show job [jobid] | shows resource allocations for job | `scontrol show job [jobid]` | 
| seff [jobid] | gets job effeciency for a finished job | `seff [jobid]` |
| sinfo | lists information about the cluster | `sinfo` | 
