---
title: Lesson
layout: page
permalink: /advanced-slurm/lesson.html
parent: Advanced Slurm
nav_enabled: true
nav_order: 2
---

# Advanced Slurm
This lesson expands on the principles of Slurm job configuration and parallelization introduced in the first Slurm lesson. 

## Lesson Setup
For this lesson, you will need to connect to a Talapas login node through a shell application of your choice.

For convenience, we recommend the [Talapas OnDemand shell](https://ondemand.talapas.uoregon.edu/pun/sys/shell/ssh/login1.talapas.uoregon.edu).

Make sure you are in your home directory.

```bash
cd ~
```

Copy the `slurm_day2` folder to your home directory.
```bash
cp -r /projects/racs_training/intro-hpc-f25/slurm_day2/ .
```

Navigate inside the `slurm_day2` directory you copied over.

```bash
cd slurm_day2
```

## Evaluating Resource Usage on Running Jobs with `htop`
Remember the `seff` command? That's a great way to evaluate resource usage of a *finished* job. But what about resource usage for running jobs?

Navigate to the `python_pi_example` directory and inspect the contents.
```bash
cd python_pi_example
ls
```

```output
calculate_pi.py  calculate_pi.sbatch
```

Inspect `calculate_pi.sbatch`.

```bash
nano calculate_pi.sbatch
```

```bash
#!/bin/bash

#SBATCH --partition=compute              ### Partition (like a queue in PBS)
#SBATCH --account=racs_training          ### Account used for job submission

### NOTE: %u=userID, %x=jobName, %N=nodeID, %j=jobID, %A=arrayMain, %a=arraySub
#SBATCH --job-name=digits_of_pi          ### Job Name
#SBATCH --output=%x.log                  ### File in which to store job output

#SBATCH --time=0-00:60:00                ### Wall clock time limit in Days-HH:MM:SS
#SBATCH --nodes=1                        ### Number of nodes needed for the job
#SBATCH --mem=100M                       ### Total Memory for job in MB -- can do K/M/G/T for KB/MB/GB/TB
#SBATCH --cpus-per-task=1                ### Number of cpus/cores to be launched per Task

### Load needed modules
module purge
module load python3/3.11.4
module list

### Run your actual program
python3 calculate_pi.py 50000
```

This is a serial job that loads the **python3** module and executes a Python script that computes the first 5000 digits of pi.

```bash
cat calculate_pi.py
```

```python
#!/usr/bin/env python3

import sys

def calcPi(limit):  # Generator function
    """
    Prints out the digits of PI
    until it reaches the given limit
    """

    q, r, t, k, n, l = 1, 0, 1, 1, 3, 3

    decimal = limit
    counter = 0

    while counter != decimal + 1:
            if 4 * q + r - t < n * t:
                    # yield digit
                    yield n
                    # insert period after first digit
                    if counter == 0:
                            yield '.'
                    # end
                    if decimal == counter:
                            break
                    counter += 1
                    nr = 10 * (r - n * t)
                    n = ((10 * (3 * q + r)) // t) - 10 * n
                    q *= 10
                    r = nr
            else:
                    nr = (2 * q + r) * l
                    nn = (q * (7 * k) + 2 + (r * l)) // (t * l)
                    q *= k
                    t *= l
                    l += 2
                    k += 1
                    n = nn
                    r = nr


def main():  # Wrapper function

    num_digits = sys.argv[1]

    for d in calcPi(int(num_digits)):
        print(d, end='')

if __name__ == '__main__':
    main()
```

Launch the Slurm job with `sbatch`.

```bash
sbatch calculate_pi.sbatch
```

```output
Submitted batch job 34799867
```

Check your queue. Note the node number (hostname) of the compute node where the job is currently running.
```bash
squeue --me
```

```output
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          34799867   compute digits_o    emwin  R       0:02      1 n0135
```

As we can see, we are currently running on node n0135.

Because we have an active Slurm task on node n0135, we can *remote into that compute node* from the login node with `ssh`.

```bash
ssh n0135
```

Enter the password to your DuckID when prompted. Confirm that you are connected with the `hostname` command.
```bash
hostname
```

```output
n0135.talapas.uoregon.edu
```

While you are on the compute node, the [**htop command**](https://linux.die.net/man/1/htop) can be used to evaluate the resources usage of your running processes. 

```bash
htop -u $USER
```

![A screenshot of the htop command on a Talapas node](../images/htop.png)

The `RES` column measures RAM per process in KB.
The `TIME` column indicates how long the process has been running.

Your connection to the compute node will terminate when your job terminates.
After the job has finished, you can use `seff` to get more granular information about your job.

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

This script shows the node name, processor type, processor architecture, and GPU parameters for each node.

Let's inspect `n0191` again.

```output
...
n0191 amd,milan,7713
n0192 amd,milan,7713
n0193 amd,milan,7713
n0194 amd,milan,7713
n0195 amd,milan,7713
n0196 amd,milan,7713
n0197 intel,sapphirerapids,6448y,mem-1tb,l40s,gpu-48gb
...
```

As you can see, `no191` has an AMD processor and it does not have a GPU.

Nodes with extra RAM are indicated with `mem-[amount]` tags.
```output
n0372 intel,broadwell,e7-4830,mem-1t
n0373 intel,broadwell,e7-4830,mem-1tb
n0374 intel,broadwell,e7-4830,mem-4tb
```

Nodes with GPUs are also indicated in this list.
```output
n0149 amd,milan,7413,a100,gpu-10gb
n0150 amd,milan,7413,a100,gpu-40gb
n0151 amd,milan,7413,a100,gpu-10gb
n0152 amd,milan,7413,a100,3xgpu-80gb,no-mig
```
These indicators are called features.
We can use **constraints** to 
direct Slurm to schedule jobs on nodes with specific features (extra RAM, GPU, CPU architecture) based on code hardware requirements.

### Don't Overwhelm the Scheduler!
Do not make 50,000 individual jobs with `srun` and a for-loop.
The maximum number of concurrent array jobs per user on Talapas is 12,000.

To get around these restrictions, you can tell Slurm to limit the nubmer of jobs submitted to the queue at a time.
Using `#SBATCH --array=0-50000%100` parameter submits only 100 subjobs to Slurm at a time.

### The Slurm `nodes` Parameter
Unless you're using an MPI or message passing interface, you should use the default
sbatch param of  `--nodes=1`.
Your jobs can't communicate between computers (nodes) without
code compatible  with MPI. Fall into this case? RACS has a [guide for configuring jobs that use MPI on Talapas](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756676/How-to+Submit+a+MPI+Job).

## Slurm Defaults on Talapas
 I have selected a few common parameters. These constraints are subject to change and do not necessarily apply on condo nodes.

| Parameter    | Default| Description | Notes
| -------- | ------- | ---------- | ----------------|
| ntasks | 1 | tasks per job | |
| cpus-per-task | 1 | number of  cpu threads per task | |
| mem-per-cpu | 4G  | memory per cpu thread        | |
| mem | 4G* | memory per node | cannot be used with mem-per-cpu |
| nodes |   1  | nodes allocated for the job | [requires mpi](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756676/How-to+Submit+a+MPI+Job) |
| gpus   | 0   | gpus per job | must be >1 to use gpus|
| error | slurm-%j.err | default error log location | |
| output | slurm-%j.out | default output log location | |

In summary, parallelism on Talapas is enabled by
* using code that *supports* multiple tasks, threads, or nodes
* enabling additional tasks, cores, or nodes as sbatch parameters
* OR using sbatch array jobs to launch simultaneous, independent jobs

Without meeting these requirements, your jobs will run serially.

## Conda Environments
RACS has a detailed guide for [building, creating, and loading conda environments](https://uoracs.github.io/talapas2-knowledge-base/docs/how-to_articles/how-to_create_personal_conda_envs) on Talapas.
We will go over some of the more advanced
capabiltiies of conda today.

Conda is especially popular in data science and scientific computing because it handles complex dependency situations like:
  - Python packages that require older versions of Python
  - non-Python libraries like R and Julia
  - packages from different channels (conda, conda-forge, pip)

### Benefits of Using Conda
1. Conda environments allow for reproducibility and consistency when running code on different devices and operating systems.
2. Conda only loads the packages you need for that specific workflow (helps reduce the amount of "clutter" in your environment)
3. Each Conda environment is a self-contained workspace, so you can: 
    - Use different Python versions side-by-side (e.g. Python 3.10 & 3.12)
    - Avoid dependency conflicts between projects

Many researchers maintain separate conda environments for different projects and contexts.
We highly recommend this approach for reproducibility, consistency, and ease of replicating environmental configurations
among colleagues.

### Conda Options on Talapas 
We have two main conda distributions available to users: 
- **miniconda3/20240410** offical source, maintained by anaconda, uses defaults as default channel, minimal installer for the Anaconda ecosystem
- **miniforge3/20240410** open-source distribution, maintained by community, uses conda-forge as default channel, fully open-source conda installer using community packages. 

### Building a Conda Environment from Scratch
Copy the `conda` folder to your home directory and navigate inside.

```bash
cp -r /projects/racs_training/conda/  ~
cd ~/conda
```

Look inside with `ls`. There's a `hello_world.py` script that requires a Python instance of some kind.
```bash
ls
```

```output
basic_r.R  conda_notes.md  hello_world.py  jupyter.yml
```

Let's load the `miniconda3/20240410` module.
```bash
module load miniconda3/20240410
```

Before creating an environment to run it, inspect the
Python script `hello_world.py` with `nano`.

```bash
nano hello_world.py
```

```python
import subprocess
import numpy
import time
import pandas as pd

def average_age():
    data = {'Name': ['Alice', 'Bob', 'Charlie'], 'Age': [25, 30, 35]}
    df = pd.DataFrame(data)
    return df['Age'].mean()



def compute_sum():
    res = 4 + 4
    return res

def main():
    print("Hello World!")
    output = compute_sum()
    print(output)
    print(average_age())
    


if __name__ == "__main__":
    main()
```

For those unfamiliar with Python, this script executes the following statements:
  * prints "Hello World!"
  * assigns of the results of the `compute_sum()` function to the variable output
  * prints the value stored at output
  * prints the result of the `average_age()` function, which operates on a [**Pandas DataFrame**](https://pandas.pydata.org/docs/)

A login node is *not* an appopriate location to run non-trivial scripts, but this toy script is safe to test there.

Let's create a new environment named `myenv` that will be stored inside the `.conda` folder of your home directory.
You can specify which python version is used through the `python=` argument.

```bash
conda create --name myenv python=3.12 numpy matplotlib
```

This command creates an environment with the **numpy** and **matplotlib** packages. When Conda finishes building the environment, you will see a message like this.

```output
Preparing transaction: done                                 
Verifying transaction: done                                 
Executing transaction: done                                 
#                                                           
# To activate this environment, use                         
#                                                           
#     $ conda activate myenv                                
#                                                           
# To deactivate an active environment, use                  
#                                                           
#     $ conda deactivate 
```

To activate *myenv*, run the `conda activate` command.

```bash
conda activate myenv
```

Use the `python` command to run the version of Python in the active conda environment.

```bash
python hello_world.py
```

```output
Traceback (most recent call last):
  File "/gpfs/home/emwin/conda/hello_world.py", line 4, in <module>
    import pandas as pd
ModuleNotFoundError: No module named 'pandas'
```

This fails because *myenv* is missing the `pandas` module. 
We need to install pandas to the current environment using `conda install`.

```bash
conda install pandas
```

When the installation is finished, rerun the script.
```bash
python hello_world.py 
```

```output
Hello World!
8
30.0
```
This is the output we expected from inspecting the source code.

### R in Conda

You can also install R and R packages to conda environments.
```bash
conda install R=4.3
```

Let's examine `basic_r.R`

```bash
cat basic_r.R
```

```R
add_numbers1 <- 4 * 2
add_numbers6 <- 4 * 8
add_numbers3 <- 4 * 9
add_numbers4 <- 4 * 12
add_numbers5 <- 9 * 7
add_numbers2 <- 5 * 8

results <- list(
  add_numbers1 = add_numbers1,
  add_numbers2 = add_numbers2,
  add_numbers3 = add_numbers3,
  add_numbers4 = add_numbers4,
  add_numbers5 = add_numbers5,
  add_numbers6 = add_numbers6
)

# Write results to a file
writeLines(paste(names(results), results, sep=": "), "results.txt")
```

Because R has been installed inside the *myenv* Conda environment, the `Rscript` is available on our path when *myenv* is activated.

```bash
which Rscript
```

```output
~/.conda/envs/myenv/bin/Rscript
```

Launch the `basic_r.R` script using `Rscript`.

```bash
Rscript basic_r.R
```

Inspect the results file from the script, `results.txt`.
```output
add_numbers1: 8
add_numbers2: 40
add_numbers3: 36
add_numbers4: 48
add_numbers5: 63
add_numbers6: 32
```

### Exporting a Conda Environment
To share our Conda environment, we can create a textual representation of the environment in the form of a special `.yml` configuration file. 
Conventionally, conda environments bundled with source code are named `environment.yml`.

The `conda env export` command [exports the environment](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#sharing-an-environment), which is written to `myenv-environment.yml`. 

```bash
conda env export > myenv-environment.yml
```
Inspect the contents with `cat`.

```bash
cat myenv-environment.yml
```
```output
name: myenv
channels:
  - defaults
dependencies:
  - _libgcc_mutex=0.1=main
  - _openmp_mutex=5.1=1_gnu
  - _r-mutex=1.0.0=anacondar_1
  - _sysroot_linux-64_curr_repodata_hack=3=haa98f57_10
  - binutils_impl_linux-64=2.40=h5293946_0
  - binutils_linux-64=2.40.0=hc2dff05_2
  - blas=1.0=openblas
  - bottleneck=1.4.2=py312ha883a20_0
  - brotli-python=1.0.9=py312h6a678d5_9
  - bwidget=1.9.16=h9eba36c_0
  - bzip2=1.0.8=h5eee18b_6
  - c-ares=1.19.1=h5eee18b_0
  - ca-certificates=2025.2.25=h06a4308_0
  - cairo=1.16.0=hb05425b_5
...
```
This list is so lengthy because it includes the packages manually installed (numpy, matplotlib, pandas, R) *and* their dependencies.

If you use the `--from-history` flag, conda will only export the packages you indicated, leaving the conda solver to choose versions and dependencies.

```bash
conda env export --from-history
```

```output
name: myenv
channels:
  - defaults
dependencies:
  - matplotlib
  - numpy
  - python=3.12
  - pandas
  - r=4.3
```

### Creating A Conda Environment from a .yml File

Let's inspect an environment file called `jupyter.yml` before creating the environment defined there.

```bash
cat jupyter.yml
```

```output
name: jupyter-racs-s25
channels:
  - conda-forge
dependencies:
  - numpy
  - pandas
  - matplotlib
  - seaborn
  - jupyter
  - r-base
  - r-essentials
  - r-irkernel
  - nodejs
  - ipywidgets
  - pip
  - pip:
    - git+https://github.com/conery/nbscan.git
```

### Conda Channels
This environment file defines an environment named *jupyter-racs-s25*. 
It uses the **conda-forge** channel. Channels are repositories where packages are downloaded from.

Conda defaults to a channel called **defaults**. The **conda-forge** channel has a larger repository of packages from the broader community.

Conda looks for packages in channels in the order that the channels appear.

This environment also uses pip to install a **nbscan** package that is not available through conda.

If you do not specify version numbers, conda will select the latest *compatible* version of a given package. 

Before we can create our new environment, we need to deactivate our current one.

Then, we can create our new environment as defined in `juptyer.yml` using `conda env create`.

```bash
conda env create -f jupyter.yml
```

When the environment building finishes, you will get a message like this.

```output
#                                                               
# To activate this environment, use                             
#                     
#     $ conda activate jupyter-racs-s25
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

No two conda environments can have the same name, so this command fail if you try to run it a second time.

We will be using the *jupyter-racs-s25* environment in the JupyterLab session.

### Using Conda Environments in Batch Scripts

To use a Conda environment you created in a Slurm batch script, you can add the following lines to your script.
Don't forget to load `miniconda3/202410` first!

For Python files:
```bash
module load miniconda3/202410
conda activate [env-name]
python [my-special-python-script].py
```

Or for R:
```bash
module load miniconda3/202410
conda activate [env-name]
R [my-special-r-script].R
```

### Debugging Conda
Observing strange behavior with Conda on Talapas? Make sure you're not loading into a conda environment through your .bashrc file.

A default `.bashrc` looks something like this:
```bash
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi

# User specific environment
if ! [[ "$PATH" =~ "$HOME/.local/bin:$HOME/bin:" ]]
then
    PATH="$HOME/.local/bin:$HOME/bin:$PATH"
fi
export PATH

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
```

Do not run the `conda init` command on Talapas to avoid modifying your .bashrc file unintentionally.
Instead, use `module load` to load either `miniconda3` or `miniforge3`.

### What About `pip`?
Pip is an [alternative package manager](https://www.anaconda.com/blog/understanding-conda-and-pip) and it can be paired with `venv` to create Python virtual environments. Using `pip` within a `conda` environment can introduce problems,
but it can be a necessary evil for packages available in pip but not in conda. 

### Useful Conda Commands

| Command | Description |
| ---------- | ---------------------------------|
| `which python`| see which python you are currently using, helpful for sanity checks | 
| `conda env list`| list out all available conda environments from loaded conda module | 
| `conda activate <environment name>` | activates specified conda environment |
| `pip list` | see packages and their versions that were installed via pip package manager |
| `conda list` | see packages and their versions installed, includes both pip and conda packages | 
| `conda search <package name>` | searches for packages within your environment | 
| `conda list --name ENVNAME --show-channel-url` | useful when trying to figure out what channel was used to install what package | 
| `conda env remove --name <environment name> --all` | deletes a conda environment (make sure you have created a backup) |



