# QIST Cluster Documentation

This documentation is meant for QIST users on the HPC cluster. It covers everything from creating an account to submitting your first job.

**Getting started:**
[General info](#general-info-on-the-qist-cluster) · [Creating an account](#creating-an-account) · [Logging in](#logging-in) · [Setting up your workspace](#setting-up-your-workspace) · [Submitting your first job](#submitting-your-first-job)

**Reference:**
[Partitions](#partitions) · [Storage](#storage-on-the-cluster) · [Remote access](#accessing-the-hpc-cluster-from-outside-ucph) · [Software](#software-on-the-cluster) · [Announcements](#cluster-announcements) · [Troubleshooting](#troubleshooting--reporting-issues) · [Contacts](#contacts)

## General info on the QIST cluster
The QIST cluster contains several compute nodes (see [description below](#partitions)) that are hosted as part of the SCIENCE HPC cluster.
The HPC cluster uses SLURM as a workload manager and job scheduling system. For SLURM specifics see the [excellent documentation](https://slurm.schedmd.com/documentation.html) as well as plenty of guides. See for example the following:
* https://psteinb.github.io/hpc-in-a-day/ (text)
* https://www.youtube.com/watch?v=NH_Fb7X6Db0 (video)

Both are comprehensive so feel free to only watch/read as much as you feel the need to have. SLURM is a powerful tool, but day-to-day usage requires only a few simple commands.

## Creating an account
If you do not yet have an UCPH HPC account, go to the following link and carefully follow the instructions: https://hpc.ku.dk/account.html

Fill out all fields on that webpage with the requested information. Barring the most obvious fields (like "First Name" etc), here is the default information for some of the fields:

- Group: Select QIST
- Preferred login name: Username for your account. Used for login and the username that everyone else sees so please choose a descriptive name.
- Preferred shell: Bash is fine
- Firewall ip [1-3]: These three fields whitelist IP-addresses for remote access. It's possible to change these afterwards (see section "[Accessing the HPC cluster from outside UCPH](#accessing-the-hpc-cluster-from-outside-ucph)"). You can input a placeholder (e.g. "123.456.789.012") or check your current IP-address at <https://ifconfig.me>. Please note that using a VPN will change your IP address.
- Next entitlement check: When they're supposed to check if you should still have access. Choose whichever option best matches your project length. For example, if you're bachelor student, choose "6 months".

When the form has been filled out, press "Submit".

> [!WARNING]
> Do not forget to send in the rules-of-conduct form linked on the SCIENCE HPC webpage, as without this your account request will not be granted!

Once your request has been granted (usually within a day or two), you should get an email with a temporary password to access the cluster for the first time and instructions on how to change your password to a permanent one.

## Logging in

Once you have an account, connect to the cluster via SSH:

```bash
ssh <your-username>@fend01.hpc.ku.dk
```

The frontend nodes `fend01` through `fend04` are all available — you can use any of them (e.g. `fend01`, `fend03`). Frontend nodes are shared, so use them only for editing files, submitting jobs, and light tasks — not for running computations.

> [!TIP]
> **Off campus?** See [Accessing the HPC cluster from outside UCPH](#accessing-the-hpc-cluster-from-outside-ucph) for remote access options.
>
> **On Windows?** See the [Windows setup guide](windows/README.md) for SSH and file-transfer instructions.

## Setting up your workspace
When you've managed to log in to your account, you need to set up your workspace.

First, we need to set up some basic configurations in your `.bashrc` file. The file `~/.bashrc` lives in your home directory. You can edit it from the terminal (`nano ~/.bashrc`, `vim ~/.bashrc`) or via WinSCP.
Open your `.bashrc` and insert the following text below what is already present.

```bash
alias q='squeue -u $USER'               # Show your queued/running jobs
alias wq="watch -n 3 'squeue -u $USER'" # Live-refresh your job queue every 3 seconds
alias kemi_gemma3='squeue -p kemi_gemma3' # Show jobs on the kemi_gemma3 partition
alias qgpu='squeue -p qist-gpu'          # Show jobs on the qist-gpu partition
alias qfast='squeue -p qist-fast'        # Show jobs on the qist-fast partition
alias qfat='squeue -p qist-fat'          # Show jobs on the qist-fat partition
alias show='scontrol show node'           # Show details for a specific node
alias job='scontrol show job'             # Show details for a specific job

# Show availability for QIST nodes
alias qav='sinfo -N -n node[240-244,265-271,321-334]'
```

After saving the file, activate the changes in your current session:

```bash
source ~/.bashrc
```

## Submitting your first job

Jobs on the cluster are submitted through SLURM using `sbatch`. Here is a minimal example script — save it as `hello.sh`:

```bash
#!/bin/bash
#SBATCH --job-name=hello
#SBATCH --partition=qist-fast
#SBATCH --account=qist
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --output=seq.%j.out
#SBATCH --error=seq.%j.err
#SBATCH --time=00:05:00
#SBATCH --mem=2G

echo "Hello from $(hostname) at $(date)"
```

Submit, monitor, and check the output:

```bash
sbatch hello.sh          # Submit the job
squeue -u $USER          # Check job status (or use the alias: q)
cat seq.<job-id>.out   # View the output once the job completes
```

### Choosing a partition

| Partition | Best for | S:C:T | Physical cores | GPUs per node | Memory per node |
|---|---|---|---|---|---|
| `kemi_gemma3` | Small–medium CPU jobs, testing | 2:8:2 | 16 | - | ~256 GB |
| `qist-fast` | Medium–large CPU jobs | 2:48:2 | 96 | - | 1.5 TB |
| `qist-fat` | Memory-intensive CPU jobs | 2:128:2 | 256 | - | 3 TB |
| `qist-gpu` | GPU-accelerated workloads | 2:24:2 | 48 | 4 x H200 | 1.5 TB |

> **S:C:T** = Sockets : Cores per socket : Threads per core. See [Partitions](#partitions) for full hardware details including logical CPU counts.

> [!TIP]
> Start with `qist-fast` for testing and small jobs. Move to `qist-fat` when you need more cores or memory. `kemi_gemma3` is old and slow, thus should only be used if the other queues are already filled. Use `qist-gpu` only for GPU workloads.

Happy calculating!

## Partitions
The partitions that QIST people can have access to are: `kemi_gemma3`, `qist-fast`, `qist-fat`, and `qist-gpu`.

To specify the partition on which you want to execute a given job, use `-p partition-name` where `partition-name` refers to a suitable partition for the job as listed below.

> [!NOTE]
> All partitions use [hyperthreading](https://en.wikipedia.org/wiki/Hyper-threading): each physical core exposes two logical CPUs. The **S:C:T** notation shows Sockets : Cores per socket : Threads per core (as reported by `sinfo -N -l`).

> [!TIP]
> **Just getting started?** If you are running a single program (e.g. a Python script), the default SLURM settings (1 physical core) are fine — just submit your job without worrying about core counts. The physical core count matters when running multiple jobs simultaneously on a node.

### `kemi_gemma3` (CPU partition)

- **Nodes:** 13
- **Physical cores:** 16 (2 sockets × 8 cores)
- **Logical CPUs:** 32
- **Memory per node:** approx. 256 GB
- **Use case:** Small-medium CPU-only jobs (DFT, post-HF, classical MD, preprocessing, etc.)

### `qist-fast` (CPU partition)

- **Nodes:** 4
- **Physical cores:** 96 (2 sockets × 48 cores), 3.65 GHz
- **Logical CPUs:** 192
- **Memory per node:** 1.5 TB
- **Use case:** Medium–large CPU-only jobs (DFT, post-HF, classical MD, preprocessing, etc.)

> [!NOTE]
> The default time limit for a given job on `qist-fast` and `qist-fat` is 1 hour. If your job requires more time than that to complete, indicate the required time in your job submission with `--time DD-HH:MM:SS`, for instance `--time 4:00:00` for a 4 hour limit. Note that no jobs can be ran for longer than a month and will thus be killed automatically once that limit is reached. If you have a job that you anticipate taking longer than a month, contact the [cluster administrator](#contacts).

### `qist-fat` (CPU partition)

- **Nodes:** 3
- **Physical cores:** 256 (2 sockets × 128 cores), 2.6 GHz
- **Logical CPUs:** 512
- **Memory per node:** 3 TB
- **Use case:** Medium–large CPU-only jobs (DFT, post-HF, classical MD, preprocessing, etc.)

### `qist-gpu` (GPU partition)

- **Nodes:** 1
- **GPUs per node:** 4 x H200 NVL 141 GB per GPU
  - **Driver:** 580.95.05
  - **CUDA:** 13.0
- **Physical cores:** 48 (2 sockets × 24 cores)
- **Logical CPUs:** 96
- **Memory per node:** 1.5 TB
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

## Storage on the cluster
There is both backed-up as well as non-backed-up storage available for the QIST users on the cluster.
As our overall storage space is limited, please clean up after your calculations to avoid unnecessary cluttering and remove files as soon as they are no longer needed.

> [!NOTE]
> If you exceed the allocated storage limit, it will no longer be possible to write to disk, and your calculation will crash. Thus, think about your storage requirements in advance. If it is impossible to run this type of calculation within the allocated limits due to the inherent storage demands, contact the [cluster administrator](#contacts).

### User home directory
Every QIST user has an automatically created home directory located at `/groups/qist/<your-username>` which is backed up and private by default. The directory has a default storage size quota of 50GB. Once you reach this size, you will be warned that you have exceeded your quota. In order for you to finish your (potentially lengthy) calculations, it will still be possible to write to your home for a grace period of one week. After this period expires, you will no longer be able to write new files until you clean up your home so that it's size falls below the allocated quota again. Please note that there is a hard limit of 200GB by default, and once that is reached all writes stop immediately.

### Shared QIST storage
For collaborative projects, there is a dedicated shared QIST storage available at `/lustre/hpc/project/qist/`. This is shared among all users, with everyone having access to the data by default, and is not backed up.
If you start a new collaborative project, please create a new directory in `/lustre/hpc/project/qist/` with the following name `<project-owner-username>_<descriptive-project-name>` so that it is visible who is responsible for that project and your collaborators can easily find it. The project owner is responsible for ensuring good data storage practices in that directory, and for freeing up the space again once the data is no longer needed.

## Accessing the HPC cluster from outside UCPH
The HPC cluster can, in general, not be accessed without being connected to the cabled internet of UCPH. However, there are ways to access the HPC cluster without being on UCPH premises:

* Whitelist an IP-address in the firewall

   While already logged in, run the following command:
   ```bash
   hpc-setup-firewall.sh
   ```
   Follow the instructions shown.

* Setup the use of multi-factor authentication

   Follow the setup in this link: https://hpc.ku.dk/documentation/otp.html. This will automatically whitelist your current IP-address remotely. For the initial setup, you have to be on UCPH premises.

## Software on the cluster
> [!NOTE]
> The QIST cluster is **self-maintained**, meaning that if programs are not already available on the cluster, users need to install them themselves as there is no software support.

Which programs you have to install depends on which types of calculations you need to run.

### Installing Python
The default version of Python the HPC cluster is 3.9.21. That may or may not be sufficient for your workload. Furthermore, `pip` often has trouble handling libraries that has complex installation procedures (such as compiling C/C++/CUDA libraries).
Other package managers solve some of the issues with `pip`. There are many -- `uv`, `poetry`, `pixi`, etc. -- and each has their own (dis)advantages. We will focus here on the installation of `conda` as it can install non-Python libraries (such as C or Fortran compilers) which can be very useful.

Follow the installation instructions given here: <https://www.anaconda.com/docs/getting-started/miniconda/main>

## Cluster announcements
Announcements from the HPC cluster administrators about maintenance windows, upgrades, and unplanned outages can be obtained by signing up to the following mailing list: <https://mailman.nbi.ku.dk/mailman/listinfo/dcsc-ku-announce>

## Troubleshooting & reporting issues
When something breaks, the best way to get it fixed (and documented) is to open a GitHub issue in this repository. That way, others can benefit from your struggles.

### Before opening an issue

1. Check this documentation quickly to see if the problem is already covered.
2. Check existing Issues (both **Open** and **Closed**) for something similar.
3. If you still don't see it -> open a new issue.

### Cluster access issues
If you suddenly can no longer access a specific partition, or experience any other cluster maintenance-related issues, you can contact the cluster support at `support@hpc.ku.dk`. Note that the cluster support will only help with general cluster issues, and not with specific software problems, etc.

## Contacts

| Who | Role | Contact |
|---|---|---|
| Marcel Fabian | Cluster administrator | via GitHub issue or email |
| HPC support | General cluster issues (access, maintenance) | support@hpc.ku.dk |
| GitHub Issues | Bug reports, documentation requests | [Open an issue](../../issues) |
| Mailing list | Cluster announcements (maintenance, outages) | [dcsc-ku-announce](https://mailman.nbi.ku.dk/mailman/listinfo/dcsc-ku-announce) |
