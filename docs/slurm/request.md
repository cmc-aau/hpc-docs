# Requesting ressources and job submission
There are several ways to request ressources and run jobs at different complexity levels through SLURM, but here are the most essential ways for interactive (foreground) and non-interactive (background) use. Generally you need to be acquainted with 3 SLURM job submission commands depending on your needs. These are [`srun`](https://slurm.schedmd.com/srun.html), [`salloc`](https://slurm.schedmd.com/salloc.html), and [`sbatch`](https://slurm.schedmd.com/sbatch.html). They all share the same options to define trackable ressource constraints ("TRES" in SLURM parlor, fx number of CPUs, memory, GPU, etc), time limits, email for job status notifications, and many other things, but their use-cases differ.

## Interactive jobs
An interactive shell is useful for testing and development purposes where you need ressources only for a short time, or to experiment with scripts and workflows on minimal example data before submitting larger jobs using [`sbatch`](#non-interactive-jobs) that will run for much longer in the background instead.

To immediately request and allocate ressources (once available) and start an interactive shell directly on the allocated compute node(s) through SLURM, use [`salloc`](https://slurm.schedmd.com/salloc.html), for example:
```
salloc --ntasks 1 --cpus-per-task 32 --mem 128G
```
Here SLURM will find a compute node with 32 CPUs and 128GB memory available and start an interactive shell on the allocated compute node(s) within the requested ressource constraints. Ressources will remain allocated until the shell is exited with `CTRL+d`, typing `exit`, or if closing the window.

???+ Important
      When using an interactive shell it's important to keep in mind that the allocated ressources remain allocated only for you until you `exit` the shell session. So don't leave it hanging idle for too long if you know you are not going to actively use it, otherwise other users might have needed the ressources in the meantime. For the same reasons, it's **not allowed** to use `salloc` or `srun` within an emulated terminal with `screen` or `tmux`, because ressources will remain allocated even though nothing is running after commands/scripts have finished. It's much better to use [`sbatch`](#non-interactive-jobs) instead.

To execute a command/script in the foreground through SLURM use [`srun`](https://slurm.schedmd.com/srun.html) to run a SLURM task directly on an allocated compute node instead of first starting an interactive shell. Any required software modules or conda environments must be loaded before issuing the command, for example:
```
module load minimap2
srun --ntasks 1 --cpus-per-task 32 --mem 128G /path/to/script/or/command
```
Ressources are then freed immediately for other jobs once the command/script exits. The terminal will be blocked for the entire duration, hence for larger jobs it's ideal to submit a job through [`sbatch`](#non-interactive-jobs) instead, which will run in the background.

???+ Note
      Keep in mind that with interactive jobs briefly losing connection to the login-node can result in the job being killed. This is to avoid that ressources would otherwise remain blocked due to unresponsive shell sessions. If you still see the job in the `squeue` overview, however, use [`sattach`](https://slurm.schedmd.com/sattach.html) to reattach to a running interactive job, just remember to append `.interactive` to the job ID, fx `38.interactive`.

[`srun`](https://slurm.schedmd.com/srun.html) is also used if multiple tasks (separate processes) must be run within the same ressource allocation (job) already obtained through [`salloc`](https://slurm.schedmd.com/salloc.html) or [`sbatch`](#non-interactive-jobs), see [example](#multi-node-multi-task-example) below.

## Non-interactive jobs
SLURM batch scripts is in many cases the preferred way to start jobs and is the recommended way to use SLURM. It's different in the way that the ressources are requested. It's done by `#SBATCH` comment-style directives in a shell script, and the script is then submitted to SLURM using an `sbatch script.sh` command. This is ideal for submitting large jobs that will run for many hours or days, but of course also for testing/development work. Ideally, a SLURM batch script should always contain (in order):

 - Any number of `#SBATCH` lines with options defining ressource constraints and other [options](#most-essential-options) for the subsequent SLURM task(s) being run
 - A list of commands to load required software modules or conda environments that are required for *all tasks*. This can also be done within any external scripts being run.
 - The main body of the script/workflow, or otherwise any number of `srun` calls to start external commands/scripts as asynchronous SLURM tasks within the same ressource allocation (=job ID)

Submit the batch script to the SLURM job queue using `sbatch script.sh`, and it will then start once the requested amount of ressources are available (also taking into account your past usage and priorities of other jobs etc, all 3 job submission commands do that). If you set the `--mail-user` and `--mail-type` arguments you should get a notification email once the job starts and finishes with additional details like how many ressources you have actually used compared to what you have requested. This is essential information for future jobs to avoid overbooking and maximize ressource utilization of the cluster.

???- "Non-interactive job output (`stdout`/`stderr` streams)"
      As the job is handled by SLURM in the background by the SLURM daemons on the individual compute nodes you won't see any output to the terminal. It will instead be written to the file(s) defined by `--output` and `--error`. To follow along use for example `tail -f job_123.out`.

### Single-node, single-task example
A full-scale example SLURM `sbatch` script for a single task could look like this:

```bash
#!/usr/bin/bash -l
#SBATCH --job-name=minimap2test
#SBATCH --output=job_%j_%x.out
#SBATCH --error=job_%j_%x.err
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=10
#SBATCH --mem=10G
#SBATCH --time=2-00:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=abc@bio.aau.dk

# load software modules or environments
module load minimap2

# run one or more commands as part a full pipeline script,
# or call scripts from elsewhere. Make sure you use the same
# number of max threads as requested 
minimap2 -t 10 database.fastq input.fastq > out.file
```

### Multi-node, multi-task example
An example SLURM `sbatch` script for parallel (independent) execution across multiple nodes could look like this:

```
#!/usr/bin/bash -l
#SBATCH --job-name=minimap2test
#SBATCH --output=job_%j_%x.out
#SBATCH --error=job_%j_%x.err
#SBATCH --nodes=5
#SBATCH --ntasks=5
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=192
#SBATCH --mem=500G
#SBATCH --time=2-00:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=abc@bio.aau.dk

# load software modules or environments
module load minimap2

# Must use srun when doing distributed work across multiple nodes
srun --ntasks 1 minimap2 -t 192 database.fastq input1.fastq > out.file1
srun --ntasks 1 minimap2 -t 192 database.fastq input2.fastq > out.file2
srun --ntasks 1 minimap2 -t 192 database.fastq input3.fastq > out.file3
srun --ntasks 1 minimap2 -t 192 database.fastq input4.fastq > out.file4
srun --ntasks 1 minimap2 -t 192 database.fastq input5.fastq > out.file5
```

For more examples of parallel jobs and array jobs see for example [this page](https://kb.swarthmore.edu/display/ACADTECH/Running+an+array+or+batch+job+on+Strelka) for now.

???+ Important
      The `bash -l` in the top "shebang" line is required for the compute nodes to be able to load software modules and conda environments correctly.

## Most essential options
There are plenty of options with the SLURM job submission commands, but below are the most important ones for our current setup and common use-cases. If you need anything else you can start with the [SLURM cheatsheet](https://slurm.schedmd.com/pdfs/summary.pdf), or else refer to the SLURM documentation for the individual commands [`srun`](https://slurm.schedmd.com/srun.html), [`salloc`](https://slurm.schedmd.com/salloc.html), and [`sbatch`](https://slurm.schedmd.com/sbatch.html).

| Option               | Description                                                                                                  |
| -------------------  | ------------------------------------------------------------------------------------------------------------  |
| `--job-name`           | A user-defined name for the job or task. This name helps identify the job in logs and accounting records.    |
| `--begin`              | Specifies a start time for the job to begin execution. Jobs won't start before this time.                  |
| `--output`, `--error`             | Redirect the job's standard output/error (`stdout`/`stderr`) to a file, ideally on network storage. All directories in the path must exist before the job can start. By default `stderr` and `stdout` are merged into a file `slurm-%j.out` in the current workdir, where `%j` is the job allocation number. See filename patterns [here](https://slurm.schedmd.com/sbatch.html#SECTION_%3CB%3Efilename-pattern%3C/B%3E).                                |
| `--ntasks-per-node`    | Specifies the number of tasks to be launched per allocated compute node.                                     |
| `--ntasks`             | Indicates the total number of tasks or processes that the job should execute.                               |
| `--cpus-per-task`      | Sets the number of CPU cores allocated per task. Required for parallel and multithreaded applications.         |
| `--mem`, `--mem-per-cpu`, or `--mem-per-gpu`                | Specifies the memory limit per node, or per allocated CPU/GPU. These are mutually exclusive.             |
| `--nodes`              | Indicates the total number of compute nodes to be allocated for the job.                                    |
| `--nodelist`           | Specifies a comma-separated list of specific compute nodes to be allocated for the job.                     |
| `--gres`               | List of "generic consumable ressources" to use, for example a GPU. |
| `--partition`          | The SLURM partition to which the job is submitted. Default is to use the `general` partition. |
| `--chdir` | Set the working directory of the batch script before it's executed. Environment variables are not supported. |
| `--time`               | Defines the maximum time limit for job execution before it will be killed automatically. Format `DD-HH:MM:SS`. Maximum allowed value is that of the partition used. [Details here](https://slurm.schedmd.com/sbatch.html#OPT_time)          |
| `--mail-type`          | Configures email notifications for certain job events. One or more comma-separated values of: `NONE`, `ALL`, `BEGIN`, `END`, `FAIL`, `REQUEUE`, `ARRAY_TASKS`. [Details here](https://slurm.schedmd.com/sbatch.html#OPT_mail-type)                       |
| `--mail-user`          | Specifies the email address where job notifications are sent.                                                |

Most options are self-explanatory. But for our setup and common use-cases you almost always want to set `--nodes` to 1, meaning your job will only run on a single compute node at a time. For multithreaded applications (most are nowadays) you mostly only need to set `ntasks` to `1` because threads are spawned from a single process (=task in SLURM parlor), and thus increase `--cpus-per-task` instead.

??? "Jobs that span multiple compute nodes"
      If needed the BioCloud is properly set up with the `OpenMPI` and `PMIx` message interfaces for distributed work across multiple compute nodes, but it requires you to [tailor your scripts and commands](https://curc.readthedocs.io/en/latest/programming/parallel-programming-fundamentals.html) specifically for distributed work and is a topic for another time. You can run "brute-force parallel" jobs, however, using for example [GNU parallel](https://curc.readthedocs.io/en/latest/software/GNUParallel.html) and distribute them across nodes, but this is only for experienced users and they must figure that out for themselves for now.

If you need to use one or more GPUs you need to specify `--partition=gpu` and set `--gres=gpu:x`, where `x` refers to the number of GPUs you need. Please don't do CPU work on the `gpu` partition unless you also need a GPU.

## How many ressources should I request for my job(s)?
Exactly how many ressources your job(s) need(s) is something you have to experiment with and learn over time based on past experience. It's important to do a bit of experimentation before submitting large jobs to obtain a qualified guess since the utilization of all the allocated ressources across the cluster is ultimately based on people's own assessments alone. Below are some tips regarding CPU and memory.

### CPUs/threads
In general the number of CPUs that you book only affects how long the job will take to finish. Since most tools don't use 100% of each and every allocated thread throughout the duration (due to for example I/O delays, internal thread communication, single-threaded job steps etc), our partitions are set with an **oversubscription factor of 1.5** (not yet, just preparing docs for it) to optimize ressource utilization. This means that SLURM will in total allocate more CPUs than there are physical cores or hyper-threads on each compute node. For example, SLURM will allocate up to 288 CPUs on a compute node with 192 threads across all SLURM jobs on the node. The number of threads is not a hard limit like the physical amount of memory is, on the other hand, and SLURM will never exceed the maximum physical memory of each compute node. Instead jobs are killed if they exceed the allocated amount of memory for the job, or not be allowed to start in the first place.

### Memory
Requesting a sensible maximum amount of memory is important to avoid crashing jobs. It's generally best to **allocate more memory** than what you need, so that the job doesn't crash and the spent ressources don't go to waste and could have been used for something else. To obtain a qualified guess you can start the job based on an initial expectation, and then set a job time limit of maybe 5-10 minutes just to see if it might crash due to exceeding the allocated ressources, and if not you will see the maximum memory usage for the job in the email notification report. Then adjust accordingly and submit again with 10-15% extra than what was used at maximum. Different steps of a workflow will in many cases unavoidably need more memory than others, and so it might be a good idea to either split the job into multiple jobs, or use workflow tools that support cluster execution, for example [snakemake](https://snakemake.readthedocs.io/en/stable/executing/cluster.html).

Our compute nodes have plenty of memory, but some tools require lots of memory. If you know that your job is going to use a lot of memory, say 1TB, you might as well also request more CPUs, since your job will likely allocate a full compute node alone, and you can then finish the job faster. This of course depends on which compute node your job is allocated to, so you might want to request ressources on individual compute nodes specifically using the `nodelist` option, refer to the [hardware overview](../index.md).
