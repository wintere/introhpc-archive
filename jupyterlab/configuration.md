---
title: Configuration
layout: page
permalink: /jupyterlab/config.html
parent: JupyterLab on Talapas
nav_enabled: true
nav_order: 2
---

# Configuration

## Setup
For this lesson, you will need to connect to a Talapas login node through a shell application of your choice.

For convenience, we recommend the [Talapas OnDemand shell](https://ondemand.talapas.uoregon.edu/pun/sys/shell/ssh/login1.talapas.uoregon.edu).

Copy today's examples from `/projects/racs_training/resources/jupyter_examples/` into your Talapas home directory using `cp`.

```bash
cp -r  /projects/racs_training/resources/jupyter_examples/ .
```
Navigate inside the `jupyter_examples` directory.

```bash
cd jupyter_examples/
```

Inspect `jupyter_rt.yml` using `nano`.

```bash
nano jupyter_rt.yml
```

```python
name: jupyter_rt
channels:
  - conda-forge
dependencies:
  - numpy
  - pandas
  - seaborn
  - jupyter
  - pickleshare
  - r-base
  - r-essentials
  - r-irkernel
  - nodejs
  - ipywidgets
  - pip
  - pip:
    - nibabel
    - bash_kernel
    - git+https://github.com/conery/nbscan.git
```

All custom conda environments
require the Jupyter module to be installed in the environment to be compatible with the JupyterLab OnDemand app.

You don't need to understand all of the packages listed
for today's exercise. Everyone needs `jupyter`. Most need 
ubiquitous pacages like `numpy` and `seaborn`. Others, for example, like `nibabel` are specific to neuroimaging.

To build this environment, you will use a Slurm script. Use `cat` to inspect its contents.

```bash
cat build_jupyter_rt.srun
```

```bash
#!/bin/bash
#SBATCH --job-name=build_jupyter_rt
#SBATCH --account=racs_training
#SBATCH --partition=compute
#SBATCH --output=logs/%x-%A.out
#SBATCH --error=logs/%x-%A.err
#SBATCH --time=0:30:0

module purge
module load miniconda3/20240410 
conda env create -f jupyter_rt.yml --solver=libmamba
conda activate jupyter_rt
python -m bash_kernel.install
```

When you are satisfied, submit the script using `sbatch`.

```bash
sbatch build_jupyter_rt.srun
```

```output
Submitted batch job 31728410
```

### Solvers
The right conda solver accelerates the resolution of dependencies when creating virtual environments.
Use `--solver=libmamba` whenever you create conda environments.

When the environment configuration job finishes, 
check the log directory for the output logs.


```bash
ls logs
```

```output
build_jupyter_rt-31728410.err  build_jupyter_rt-31728410.out
```

Let's check the last few lines of the output log with tail.

```bash
tail logs/build_jupyter_rt-31728410.out
```

```output
# To activate this environment, use
#
#     $ conda activate jupyter_rt
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

Looks good! This means you have an environment ready
to activate use either in batch jobs *or* (for today's purposes)
in the OnDemand JupyterLab app.

## Configuring the OnDemand JupyterLab App

Navigate to the [OnDemand homepage](https://ondemand.talapas.uoregon.edu/pun/sys/dashboard) and login with your **DuckID**.

From the **Interactive Apps** dropdown menu, select **Jupyter**.

![jupyter app](../images/jupyter-config.png)*An example JupyterLab configuration screen*

**PIRG**: During the training period, you can use `racs_training`, but you should use *your lab's PIRG* after these trainings finish.

**SLURM Partition:** You have two choices: `interactive` (CPU-based jobs) or `interactivegpu` (if you need GPUs). Only request a GPU if your code or software pipeline can use one.

**Number of hours:** Be considerate and ask only for as much time as you need. You can ask for up to 12 hours on `interactive`, 8 hours on `interactivegpu`.

**CPU Cores:** You must ask for at least `1`. Only code or software packages that are configured to do so can take advantage of multiple cores. 

**Total memory:** We typically recommend 4GB per CPU core. You can ask for up to 100GB.

**GPU enabled:** Only check if you require a GPU. To use a GPU, this box must be checked **AND** you must run on the `interactivegpu` partition.

**Reservation:** If you don't know what that means, please leave this box blank.

**Alternate Conda Environment**: To use Jupyter OnDemand with a custom environment, you need to pass in the *name* of the conda environment. For this exercise, use the `jupyter_rt` environment. To create your own conda environments, follow this tutorial.

When you've adjusted your job settings, click the **Launch** button.

## Launching JupyterLab from Sessions

You will automatically be navigated to the **My Interactive Sessions** page. 

![jupyter sessions](../images/jupyter-launch-menu.png)

When your interactive job has been scheduled, the **Connect to Jupyter** button will display.

If the button does not appear, check the logs in the **Session ID** link. Keep in mind that the more resources you ask for, the longer it will to schedule.

## Selecting a Kernel
JupyterLab supports interaction through Jupyter notebooks in the **Notebook** menu, through an interactive Python interpreter in the **Console** menu, and text editing functionality in the **Other** menu.

![jupyter notebook](../images/jupyter-notebook-launcher.png)

A single `conda` environment can have multiple kernels -- for example, we created one that has both an R and Python kernel. 

To run Jupyter Notebooks, you will need to select a [kernel](https://jupyterlab.readthedocs.io/en/stable/user/running.html). You should select the **Python 3 (ipykernel)** from the **Notebook** menu to create, edit, and execute Jupyter Notebooks.

After clicking, you will be prompted to select a specific Python kernel. 
![kernel select](../images/jupyter-kernel-select.png)

After selecting a kernel, open the `jupyter_examples` folder in your home directory [from within the JupyterLab interface](https://jupyterlab.readthedocs.io/en/stable/user/files.html).

## Exercise Notebooks

Today's lesson continues in the [Notebooks](../jupyterlab/notebooks.html) section.
