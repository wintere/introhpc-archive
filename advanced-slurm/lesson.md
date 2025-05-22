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
| error | slurm-%j.err | default error log location | |
| output | slurm-%j.out | default output log location | |

In summary, parallelism on Talapas is enabled by
* using code that *supports* multiple tasks, threads, or nodes
* enabling additional tasks, cores, or nodes as sbatch parameters
* OR using sbatch array jobs to launch simultaneous, independent jobs

Without meeting these requirements, your jobs will run serially.

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

### What About Multinode Jobs
Unless you're using an MPI or message passing interface, you should use the default
sbatch param of  `--nodes=1`.
Your jobs can't communicate between computers (nodes) without
code compatible  with MPI. Fall into this case? RACS has a [guide for configuring jobs that use MPI on Talapas](https://hpcrcf.atlassian.net/wiki/spaces/TW/pages/2755756676/How-to+Submit+a+MPI+Job).


