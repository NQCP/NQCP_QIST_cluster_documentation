# NQCP_QIST_cluster_documentation
This documentation is meant for QIST users on Steno. It explains how to get an account, log in, which partitions to use, and how to set up a working environment.

For SLURM specifics see the [excellent documentation](https://slurm.schedmd.com/documentation.html) as well as plenty of guides. See for example the following:
* https://psteinb.github.io/hpc-in-a-day/ (text)
* https://www.youtube.com/watch?v=NH_Fb7X6Db0 (video)

Both are comprehensive so feel free to only watch/read as much as you feel the need to have. SLURM is a powerful tool, but day-to-day usage involves only a few commands.

## Creating an account
Go to the following link and follow the instructions: https://hpc.ku.dk/account.html

Fill out the requested information. Barring the most obvious ones (like "First Name" etc), I'll list default information for some of the fields:

```text
Preferred login name: Username for your account on Steno. It's used for login, but is also the username that everyone else sees so please choose a descriptive name.
Preferred shell: Bash is fine
Firewall ip [1-3]: If you don't know your home IP-address, just input whatever. For example, "123.456.789.012". These three fields whitelist IP-addresses for remote access. It's possible to change these afterwards (see section "Accessing Steno from outside UCPH")
Next entitlement check: When they're supposed to check if you should still have access to Steno. Choose whichever option best matches your project length. For example, if you're bachelor, choose "6 months".
```
When the form has been filled out, press "Submit".

When you have filled out the form, you need to send an email to support at HPC/UCPH where you kindly ask for access to Steno. REMEMBER to write which queue you need access to and to attach the rules-of-conduct you've signed! Gemma needs to be CC'ed so that IT-support knows that she has allowed you access.

```email
support@hpc.ku.dk
```
Now you just need patience! You should get an email within a day or two where they confirm that your account has been created.

Happy calculating!

## Partitions
The partitions that QIST people should have access to are: `kemi_gemma3` and `qist-gpu`.
> [!CAUTION]
> Do *not* run CPU-only jobs on `qist-gpu`. If all CPU cores are used, no one can run GPU jobs even if the GPUs are idle.

As there is no central administrator or automated process to keep track of usage, please help each other to make the best of the available resources. And if you notice someone accidentally having only CPU jobs on the GPUs, please write them (or Nina Glaser) to let them know.

The following resources are available on those partitions:

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

First, we need to set up some basic configurations in your `.bashrc` file. The file `~/.bashrc` lives in your home directory. You can edit it from the terminal (`nano ~/.bashrc`, `vim ~/.bashrc`) or via WinSCP
1. Open your `.bashrc`. This can be done either through WinSCP or through the terminal with vim/nano.

2. Insert the following text below what is already present:

```bash
alias q='squeue -u $USER'
alias wq="watch -n 3 'squeue -u $USER'"
alias qgpu='squeue -p qist-gpu'
alias q3='squeue -p kemi_gemma3'
alias show='scontrol show node'
alias job='scontrol show job'

# show availability for nodes node[240,321-334] (QIST nodes)
alias qav='sinfo -N -n node[240,321-334]'
```

## Accessing Steno from outside UCPH
Steno can, in general, not be accessed without being connected to the cabled internet of UCPH. However, there are ways to access Steno without being on UCPH premises:
1. Whitelist an IP-address in the firewall

While already logged in on Steno, run the following command:
```bash
hpc-setup-firewall.sh
```
Follow the instructions shown.

2. Setup the use of multi-factor authentication

Follow the setup in this link: https://hpc.ku.dk/documentation/otp.html. This will automatically whitelist your current IP-address remotely.

### Installing Python
The default version of Python on Steno is 3.9.21. That may or may not be sufficient for your workload. Furthermore, as `pip` often has trouble handling libraries that has complex installation procedures (such as compiling C/C++/CUDA libraries).
Other package managers solve some of the issues with `pip`. There are many -- `uv`, `poetry`, `pixi`, etc. -- and each has their own (dis)advantages. I will explain the installation of `conda` as that can install non-Python libraries (such as C or Fortran compilers) which can be very useful.

Follow the installation instructions given here: <https://www.anaconda.com/docs/getting-started/miniconda/main>

## Cluster announcements
Announcements from Steno about maintenance windows, upgrades, and unplanned outages can be obtained by signing up to the following mailing list: <https://mailman.nbi.ku.dk/mailman/listinfo/dcsc-ku-announce>

## Troubleshooting & reporting issues
When something breaks, the best way to get it fixed (and documented) is to open a GitHub issue in this repository. That way, others can benefit from your struggles.

### Before opening an issue

1. Check this documentation quickly to see if the problem is already covered.
2. Check existing Issues (both **Open** and **Closed**) for something similar.
3. If you still don’t see it -> open a new issue.

