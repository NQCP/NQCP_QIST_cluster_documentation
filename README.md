# NQCP_QIST_cluster_documentation
This documentation is meant for QIST users on the HPC cluster. It explains how to get an account, log in, which partitions to use, and how to set up a working environment.

## General info on the QIST cluster
The QIST cluster contains several compute nodes (see [description below](#partitions)) that are hosted as part of the the SCIENCE HPC cluster.
The HPC cluster uses SLURM as a workload manager and job scheduling system. For SLURM specifics see the [excellent documentation](https://slurm.schedmd.com/documentation.html) as well as plenty of guides. See for example the following:
* https://psteinb.github.io/hpc-in-a-day/ (text)
* https://www.youtube.com/watch?v=NH_Fb7X6Db0 (video)

Both are comprehensive so feel free to only watch/read as much as you feel the need to have. SLURM is a powerful tool, but day-to-day usage requires only a few simple commands.

## Creating an account
If you do not yet have an UCPH HPC account, go to the following link and carefully follow the instructions: https://hpc.ku.dk/account.html

Fill out all fields with the requested information. Barring the most obvious ones (like "First Name" etc), here is the default information for some of the fields:

- Group: Select QIST
- Preferred login name: Username for your account. Used for login and the username that everyone else sees so please choose a descriptive name.
- Preferred shell: Bash is fine
- Firewall ip [1-3]: These three fields whitelist IP-addresses for remote access. It's possible to change these afterwards (see section "Accessing the HPC cluster from outside UCPH"). You can input a placeholder (e.g. "123.456.789.012") or check your current IP-address at <https://ifconfig.me>. Please note that using a VPN will change your IP address.
- Next entitlement check: When they're supposed to check if you should still have access. Choose whichever option best matches your project length. For example, if you're bachelor student, choose "6 months".

When the form has been filled out, press "Submit". **Do not forget to send in the rules-of-conduct form, as without this your account request will not be granted!**

Once your request has been granted (usually within a day or two), you should get an email with a temporary password to access the cluster for the first time and instructions on how to change you password to a permanent one.

Happy calculating!

## Setting up your workspace
When you've managed to log in to your account, you need to set up your workspace.

First, we need to set up some basic configurations in your `.bashrc` file. The file `~/.bashrc` lives in your home directory. You can edit it from the terminal (`nano ~/.bashrc`, `vim ~/.bashrc`) or via WinSCP.
Open your `.bashrc` and insert the following text below what is already present.

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

## Partitions
The partitions that QIST people can have access to are: `kemi_gemma3`, `qist-fast`, `qist-fat`, and `qist-gpu`.

To specify the partition on which you want to execute a given job, use `-p partition-name` where `partition-name` refers to a suitable partition for the job as listed below.
The 2 x X cores means that they are [hyperthreaded](https://en.wikipedia.org/wiki/Hyper-threading):

### `kemi_gemma3` (CPU partition)

- **Nodes:** 13
- **CPUs per node:** 2 x 16 cores (32 cores total)
- **Memory per node:** approx. 256 GB
- **Use case:** Small-medium CPU-only jobs (DFT, post-HF, classical MD, preprocessing, etc.)

### `qist-fast` (CPU partition)

- **Nodes:** 4
- **CPUs per node:** 2 x 48 cores, 3.65 GHz (96 cores total)
- **Memory per node:** 1.5 TB
- **Use case:** Medium–large CPU-only jobs (DFT, post-HF, classical MD, preprocessing, etc.)

> [!NOTE]
> The default time limit for a given job on `qist-fast` and `qist-fat` is 1 hour. If your job requires more time than that to complete, indicate the required time in your job submission with `--time DD-HH:MM:SS`, for instance `--time 4:00:00` for a 4 hour limit. Note that no jobs can be ran for longer than a month and will thus be killed automatically once that limit is reached. If you have a job that you anticipate taking longer than a month, contact the cluster administrator Nina Glaser.

### `qist-fat` (CPU partition)

- **Nodes:** 3
- **CPUs per node:** 2 x 96 cores, 2.6 GHz (192 cores total)
- **Memory per node:** 3 TB
- **Use case:** Medium–large CPU-only jobs (DFT, post-HF, classical MD, preprocessing, etc.)

### `qist-gpu` (GPU partition)

- **Nodes:** 1
- **GPUs per node:** 4 x H200 NVL 141 GB per GPU
  - **Driver:** 580.95.05
  - **CUDA:** 13.0
- **CPUs / memory per node:** 2 x 24 cores, 1.5 TB RAM
- **Use case:** GPU-accelerated workloads (neural network wavefunctions, ML potentials, and other CUDA/JAX/PyTorch jobs).

> [!CAUTION]
> Do *not* run CPU-only jobs on `qist-gpu`. If all CPU cores are used, no one can run GPU jobs even if the GPUs are idle.

As there is no central administrator or automated process to keep track of usage, please help each other to make the best of the available resources. And if you notice someone accidentally having only CPU jobs on the GPUs, please write them to let them know that they are blocking your calculations.
The real-life name of someone can often be found running the following command:
```bash
getent passwd <username>
```

### Info about partitions, nodes, and jobs.
SLURM can be queried for different information about partitions, nodes, and jobs. Questions such as "what is the default time limit for a job?", "how many CPUs have been allocated on a given node?", "how much memory is this job taking?", or more. The following lists commands that you can use to get information about these different levels:
* `scontrol show partition <partition_name>` will provide information about a **partition** such as `qist-gpu`.
* `scontrol show node <node_id>` will provide information about a **node** such as `node240`.
* `scontrol show job <job_id>` will provide information about a **job**

## Accessing the HPC cluster from outside UCPH
The HPC cluster can, in general, not be accessed without being connected to the cabled internet of UCPH. However, there are ways to access the HPC cluster without being on UCPH premises:
1. Whitelist an IP-address in the firewall

While already logged in, run the following command:
```bash
hpc-setup-firewall.sh
```
Follow the instructions shown.

2. Setup the use of multi-factor authentication

Follow the setup in this link: https://hpc.ku.dk/documentation/otp.html. This will automatically whitelist your current IP-address remotely.

## Software on the cluster
> [!NOTE]
> The QIST cluster is **self-maintained**, meaning that if programs are not already available on the cluster, users need to install them themselves as there is no software support.

Which programs you have to install depends on which types of calculations you need to run.

### Installing Python
The default version of Python the HPC cluster is 3.9.21. That may or may not be sufficient for your workload. Furthermore, `pip` often has trouble handling libraries that has complex installation procedures (such as compiling C/C++/CUDA libraries).
Other package managers solve some of the issues with `pip`. There are many -- `uv`, `poetry`, `pixi`, etc. -- and each has their own (dis)advantages. We will focus here on the installation of `conda` as it can install non-Python libraries (such as C or Fortran compilers) which can be very useful.

Follow the installation instructions given here: <https://www.anaconda.com/docs/getting-started/miniconda/main>

## Storage on the cluster
There is both backed-up as well as non-backed-up storage available for the QIST users on the cluster.
As our overall storage space is limited, please clean up after your calculations to avoid unnecessary cluttering and remove files as soon as they are no longer needed.

> [!NOTE]
> If you exceed the allocated storage limit, it will no longer be possible to write to disk, and your calculation will crash. Thus, think about your storage requirements in advance. If it is impossible to run this type of calculation within the allocated limits due to the inherent storage demands, contact the cluster administrator Nina Glaser.

### User home directory
Every QIST user has an automatically created home directory loacted at `/groups/qist/<your-username>` which is backed up and private by default. The directory has a default storage size quota of 50GB. Once you reach this size, you will be warned that you have exceeded your quota. In order for you to finish your (potentially lengthy) calculations, it will still be possible to write to your home for a grace period of one week. After this period expires, you will no longer be able to write new files until you clean up your home so that it's size falls below the allocated quota again. Please note that there is a hard limit of 200GB by default, and once that is reached all writes stop immediately.

### Shared QIST storage
For collaborative projects, there is a dedicated shared QIST storage available at `/lustre/hpc/project/qist/`. This is shared among all users, with everyone having access to the data by default, and is not backed up.
If you start a new collaborative project, please create a new directory in `/lustre/hpc/project/qist/` with the following name `<project-owner-username>_<descriptive-project-name>` so that it is visible who is responsible for that project and your collaborators can easily find it. The project owner is responsible for ensuring good data storage practices in that directory, and for freeing up the space again once the data is no longer needed.

## Cluster announcements
Announcements from the HPC cluster administrators about maintenance windows, upgrades, and unplanned outages can be obtained by signing up to the following mailing list: <https://mailman.nbi.ku.dk/mailman/listinfo/dcsc-ku-announce>

## Troubleshooting & reporting issues
When something breaks, the best way to get it fixed (and documented) is to open a GitHub issue in this repository. That way, others can benefit from your struggles.

### Before opening an issue

1. Check this documentation quickly to see if the problem is already covered.
2. Check existing Issues (both **Open** and **Closed**) for something similar.
3. If you still don’t see it -> open a new issue.

## Cluster access issues
If you suddenly can no longer access a specific partition, or experience any other cluster maintenance-related issues, you can contact the cluster support at
```email
support@hpc.ku.dk
```
Note that the cluster support will only help with general cluster issues, and not with specific software problems, etc.
