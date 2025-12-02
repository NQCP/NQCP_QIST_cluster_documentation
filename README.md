# NQCP_QIST_cluster_documentation
This documentation is mostly meant to introduce and explain the layout of Steno (our local high-performance computing (HPC) cluster), but not the usage of SLURM (the job queuing system).

For SLURM specifics see the [excellent documentation](https://slurm.schedmd.com/documentation.html) as well as plenty of guides. See for example the following:
* https://psteinb.github.io/hpc-in-a-day/ (text)
* https://www.youtube.com/watch?v=NH_Fb7X6Db0 (video)

Both are comprehensive so feel free to only watch/read as much as you feel the need to have. SLURM is a powerful tool, but day-to-day usage involves only a few commands.


## Partitions
The partitions that QIST people should have access to are: `kemi_gemma3` and `qist-gpu`. Please make sure to run CPU jobs on `kemi_gemma3`! If all CPUs are currently in use on `qist-gpu`, no one will be able to run GPU workloads -- even if all the GPUs are unused. The following resources are available on those partitions:

### `kemi_gemma3` (CPU partition)

- **Nodes:** 13
- **CPUs per node:** 2 x 16 cores (32 cores total)
- **Memory per node:** approx. 256 GB
- **Use case:** Medium–large CPU-only jobs (DFT, post-HF, classical MD, preprocessing, etc.)

### `qist-gpu` (GPU partition)

- **Nodes:** 1
- **GPUs per node:** 4 x H200 NVL 141 GB per GPU
  - **Driver:** 580.95.05
  - **CUDA:** 13.0
- **CPUs / memory per node:** 2 x 48 cores, 1.5 TB RAM
- **Use case:** GPU-accelerated workloads (neural network wavefunctions, ML potentials, and other CUDA/JAX/PyTorch jobs).

## Set up workspace on Steno
When you've managed to log in to your account on Steno, you need to set up your workspace.

Which programs you have to install depends on which types of calculations you need to run. In the following, I'll walk through each installation independently and it is up to you to tailor it to your specific needs.

First, we need to set up some basic configurations in your .bashrc file.
1. Open your .bashrc file. This can be done either through WinSCP or through the terminal with vim/nano.

2. Insert the following text below what is already present. Replace `XXX` with your username:

```bash
alias q='squeue -u XXX'
alias wq="watch -n 3 'squeue -u XXX'"
alias qgpu='squeue -p qist-gpu'
alias q3='squeue -p kemi_gemma3'
alias show='scontrol show node'
alias job='scontrol show job'
alias qav='sinfo -N -n node[240,321-334]'
```

## Accessing Steno
Steno can, in general, not be accessed without being connected to the cabled internet of UCPH. However, there are ways to access Steno without being on UCPH premises:
1. Whitelist an IP-address in the firewall

While already logged in on Steno, run the following command:
```bash
hpc-setup-firewall.sh
```
Follow the instructions shown.

2. Setup the use of multi-factor authentication

Follow the setup in this link: https://hpc.ku.dk/documentation/otp.html. This will automatically whitelist your current IP-address remotely.

### Installing python
The default version of Python on Steno is 3.9.21. That may or may not be sufficient for you workload. Furthermore, as `pip` often has trouble handling libraries that has complex installation procedures (such as compiling C/C++/CUDA libraries).
Other package managers solve some of the issues with `pip`. There are many -- `uv`, `poetry`, `pixi`, etc. -- and each has their own (dis)advantages. I will explain the installation of `conda` as that can install non-python libraries (such as C or Fortran compilers) which can be very useful.

Follow the installation instructions given here: https://www.anaconda.com/docs/getting-started/miniconda/main


# Announcements
Announcements from Steno about maintenance work or significant/unplanned downtime can be obtained by signing up to the following mailing list: https://mailman.nbi.ku.dk/mailman/listinfo/dcsc-ku-announce
