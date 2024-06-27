# Requesting resources and job submission
Once you are [logged in](../access.md) to one of the login nodes through SSH, there are several ways to request resources and run jobs at different complexity levels through SLURM, but here are the most essential ways for interactive (foreground) and non-interactive (background) use. Generally you need to be acquainted with 3 SLURM job submission commands depending on your needs. These are [`srun`](https://slurm.schedmd.com/archive/slurm-23.02.6/srun.html), [`salloc`](https://slurm.schedmd.com/archive/slurm-23.02.6/salloc.html), and [`sbatch`](https://slurm.schedmd.com/archive/slurm-23.02.6/sbatch.html). They all share the exact same [options](#most-essential-options) to define trackable resource constraints ("TRES" in SLURM parlor, fx number of CPUs, memory, GPU, etc), time limits, email for job status notifications, and many other things, but are made for different use-cases, which will be described below.

## Interactive jobs
An interactive shell is useful for testing and development purposes where you need resources only for a short time, or to experiment with scripts and workflows on minimal example data before submitting larger jobs using [`sbatch`](#non-interactive-jobs) that will run for much longer in the background instead. It's also the only way to run [graphical apps](#graphical-apps-gui).

???+ Important
      When using an interactive shell it's important to keep in mind that the allocated resources remain reserved only for you until you `exit` the shell session. So don't leave it hanging idle for too long if you know you are not going to actively use it, otherwise other users might have needed the resources in the meantime. For the same reasons, it's **not allowed** to use `salloc` or `srun` within an emulated terminal with `screen` or `tmux`, because resources will remain reserved even though nothing is running after commands/scripts have finished. It's much better to use [`sbatch`](#non-interactive-jobs) instead. As a last resort if you really insist on an interactive session you can append for example `; exit` to the last command you execute to ensure that the job allocation is automatically terminated when the command exits (regardless of exit status). Or just use `sbatch`!! :)

### Using the `salloc` command
To immediately request and allocate resources (once available) and start an **interactive shell** session directly on the allocated compute node(s) through SLURM, just type [`salloc`](https://slurm.schedmd.com/archive/slurm-23.02.6/salloc.html):

```
$ salloc
```

Here SLURM will find a compute node with the default amount of resources available (which is 1CPU, 512MB memory, and a 1-hour time limit at the time of writing) and start the session on the allocated compute node(s) within the requested resource constraints. If you need more resources you need to explicitly ask for it, for example:

```
$ salloc --cpus-per-task 2 --mem 4G --time 0-3:00:00
```

Resources will then remain allocated until the shell is exited with `CTRL+d`, typing `exit`, or closing the window. If it takes more than a few seconds to allocate resources, your job might be queued due to a variety of reasons. If so check the [`REASON` codes](jobcontrol.md#get-job-status-info) for the job with `squeue` from another session.

### Using the `srun` command
If you just need to run a single command/script in the foreground it's better to use [`srun`](https://slurm.schedmd.com/archive/slurm-23.02.6/srun.html), which will run things directly on a compute node instead of first starting an interactive shell. As opposed to `salloc` the job is terminated **immediately** once the command/script finishes. Any required software modules or conda environments must be loaded first before issuing the command, for example:

```
$ module load minimap2
$ srun --cpus-per-task 8 --mem 16G --time 1-00:00:00 minimap2 <options>
```

The terminal will be blocked for the entire duration, hence for larger jobs it's ideal to submit a job through [`sbatch`](#non-interactive-jobs) instead, which will run in the background.

[`srun`](https://slurm.schedmd.com/archive/slurm-23.02.6/srun.html) is also used if multiple tasks (separate processes) must be run within the same resource allocation (job) already obtained through [`salloc`](https://slurm.schedmd.com/archive/slurm-23.02.6/salloc.html) or [`sbatch`](#non-interactive-jobs), see [example](#multi-node-multi-task-example) below. SLURM tasks can then span multiple compute nodes at once to distribute highly parallel work at any scale.

### Graphical apps (GUI)
In order to run graphical programs simply append the [`--x11` option](https://slurm.schedmd.com/archive/slurm-23.02.6/srun.html#OPT_x11) to `salloc` or `srun` and run the program. The graphical app will then show up in a window on your own computer, while running inside a SLURM job on the cluster:
```
$ srun --cpus-per-task 2 --mem 4G --time 0-3:00:00 --x11 /path/to/gui/app
```

It's important to mention that in order for this to work properly, you must first ensure that you have connected to the particular login node using either the `ssh -X` option or that you have set the `ForwardX11 yes` option in your SSH config file, [see example here](../../access/#ssh-config-file).

???- "Connectivity and interactive jobs"
      Keep in mind that with interactive jobs briefly losing connection to the login-node can result in the job being killed. This is to avoid that resources would otherwise remain blocked due to unresponsive shell sessions. If you still see the job in the `squeue` overview, however, use [`sattach`](https://slurm.schedmd.com/archive/slurm-23.02.6/sattach.html) to reattach to a running interactive job, just remember to append `.interactive` to the job ID, fx `38.interactive`.

## Non-interactive jobs
The most convenient and **highly recommended** way to run most things is by submitting jobs to the job queue for execution in the background in the form of SLURM batch scripts through the `sbatch` command. Resource requirements are instead defined by `#SBATCH` comment-style directives at the top of a shell script, and the script is then submitted to SLURM using a simple `sbatch script.sh` command. This is ideal for submitting large jobs that will run for many hours or days, but of course also for testing/development work. A SLURM batch script should always contain (in order):

 - Any number of `#SBATCH` lines with options defining resource constraints and other [options](#most-essential-options) for the subsequent SLURM task(s) to be run.
 - A list of commands to load required software modules or conda environments that are required for *all tasks*. This can also be done separately within any external scripts being run.
 - The main body of the script/workflow, or call to an external script or program to run within the resource allocation (=job ID)

Submit the batch script to the SLURM job queue using `sbatch script.sh`, and it will then start once the requested amount of resources are available (also taking into account your past usage and priorities of other jobs etc, all 3 job submission commands do that). If you set the `--mail-user` and `--mail-type` arguments you should get a notification email once the job starts and finishes with additional details like how many resources you have actually used compared to what you have requested. This is essential information for future jobs to avoid overbooking and maximize resource utilization of the cluster.

You can also simply add `#SBATCH` lines to any shell script you already have, and also run the script with arguments, so for example instead of `bash script.sh -i input -o output ...` you can simply run `sbatch script.sh -i input -o output ...`.

???- "Non-interactive job output (`stdout`/`stderr` streams)"
      As the job is handled by SLURM in the background by the SLURM daemons on the individual compute nodes you won't see any output to the terminal. It will instead be written to the file(s) defined by `--output` and/or `--error`. To follow along in real time use for example `tail -f job_123.out`.

### Single-node, single-task example
A full-scale example SLURM `sbatch` script for a single task could look like this:

```bash
#!/usr/bin/bash -l
#SBATCH --job-name=minimap2test
#SBATCH --output=job_%j_%x.out
#SBATCH --partition=default-op
#SBATCH --cpus-per-task=10
#SBATCH --mem=10G
#SBATCH --time=2-00:00:00
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=abc@bio.aau.dk

# Exit on first error and if any variables are unset
set -eu

# load software modules or environments
module load minimap2

# Get number of CPUs from the SLURM allocation (SLURM will sometimes 
# give you more than what you have requested to optimize efficiency).
# This only works with single-node jobs
max_threads="$(nproc)"

# run one or more commands as part a full pipeline script or call scripts from elsewhere
minimap2 -t "$max_threads" database.fastq input.fastq > out.file
```

### Multi-node, multi-task example
An example SLURM `sbatch` script for parallel (independent) execution across multiple nodes could look like this:

```bash
#!/usr/bin/bash -l
#SBATCH --job-name=minimap2test
#SBATCH --output=job_%j_%x.out
#SBATCH --nodes=5
#SBATCH --ntasks=5
#SBATCH --ntasks-per-node=1
#SBATCH --partition=default-op
#SBATCH --cpus-per-task=60
#SBATCH --mem-per-cpu=3G
#SBATCH --time=2-00:00:00
#SBATCH --mail-type=END,FAIL
#SBATCH --mail-user=abc@bio.aau.dk

# Exit on first error and if any variables are unset
set -eu

# load software modules or environments
module load minimap2

# Must use srun when doing distributed work across multiple nodes
srun --ntasks 1 minimap2 -t 60 database.fastq input1.fastq > out.file1
srun --ntasks 1 minimap2 -t 60 database.fastq input2.fastq > out.file2
srun --ntasks 1 minimap2 -t 60 database.fastq input3.fastq > out.file3
srun --ntasks 1 minimap2 -t 60 database.fastq input4.fastq > out.file4
srun --ntasks 1 minimap2 -t 60 database.fastq input5.fastq > out.file5
```

For more examples of parallel jobs and array jobs, for now see for example [this page](https://kb.swarthmore.edu/display/ACADTECH/Running+an+array+or+batch+job+on+Strelka).

???+ Important
      The `bash -l` in the top "shebang" line is required for the compute nodes to be able to load software modules and conda environments correctly.

??? "Jobs that span multiple compute nodes"
      If needed the BioCloud is properly set up with the `OpenMPI` and `PMIx` message interfaces for distributed work across multiple compute nodes, but it requires you to [tailor your scripts and commands](https://curc.readthedocs.io/en/latest/programming/parallel-programming-fundamentals.html) specifically for distributed work and is a topic for another time. You can run "brute-force parallel" jobs, however, using for example [GNU parallel](https://curc.readthedocs.io/en/latest/software/GNUParallel.html) and distribute them across nodes, but this is only for experienced users and they must figure that out for themselves for now.

## Requesting one or more GPUs
If you need to use one or more GPUs you need to specify `--partition=gpu` and set `--gres=gpu:x`, where `x` refers to the number of GPUs you need. Please don't do CPU work on the `gpu` partition unless you also need a GPU. It's also worth considering using `--cpus-per-gpu` and `--mem-per-gpu`. Additional details [here](https://slurm.schedmd.com/archive/slurm-23.02.6/gres.html).

## Most essential options
There are plenty of options with the SLURM job submission commands, but below are the most important ones for our current setup and common use-cases. If you need anything else you can start with the [SLURM cheatsheet](https://slurm.schedmd.com/archive/slurm-23.02.6/pdfs/summary.pdf), or else refer to the SLURM documentation for the individual commands [`srun`](https://slurm.schedmd.com/archive/slurm-23.02.6/srun.html), [`salloc`](https://slurm.schedmd.com/archive/slurm-23.02.6/salloc.html), and [`sbatch`](https://slurm.schedmd.com/archive/slurm-23.02.6/sbatch.html).

| Option | Default value(s) | Description |
| --- | --- | --- |
| `--job-name`                                 | The name of the script | A user-defined name for the job or task. This name helps identify the job in logs and accounting records. |
| `--begin`                                    | Now | Specifies a start time for the job to begin execution. Jobs won't start before this time. [Details here](https://slurm.schedmd.com/archive/slurm-23.02.6/sbatch.html#OPT_begin). |
| `--output`, `--error`                        | `slurm-<jobid>.out` | Redirect the job's standard output/error (`stdout`/`stderr`) to a file, ideally on network storage. All directories in the path must exist before the job can start. By default `stderr` and `stdout` are merged into a file `slurm-%j.out` in the current workdir, where `%j` is the job allocation number. See filename patterns [here](https://slurm.schedmd.com/archive/slurm-23.02.6/sbatch.html#SECTION_%3CB%3Efilename-pattern%3C/B%3E). |
| `--ntasks-per-node`                          | `1` | Specifies the number of tasks to be launched per allocated compute node. |
| `--ntasks`                                   | `1` | Indicates the total number of tasks or processes that the job should execute. |
| `--cpus-per-task`                            | `1` | Sets the number of CPU cores allocated per task. Required for parallel and multithreaded applications. |
| `--mem`, `--mem-per-cpu`, or `--mem-per-gpu` | `512MB` (per node) | Specifies the memory limit per node, or per allocated CPU/GPU. These are mutually exclusive. |
| `--nodes`                                    |  `1` | Indicates the total number of compute nodes to be allocated for the job. |
| `--nodelist`                                 |  | Specifies a comma-separated list of specific compute nodes to be allocated for the job. |
| `--exclusive`                                |  | Flag. If set will request exclusive access to a full compute node, meaning no other jobs will be allowed to run on the node. In this case you might as well also use all available memory by setting `--mem=0`, unless there are suspended jobs on the particular node. [Details here](https://slurm.schedmd.com/archive/slurm-23.02.6/sbatch.html#OPT_exclusive). |
| `--gres`                                     |  | List of "generic consumable resources" to use, for example a GPU. [Details here](https://slurm.schedmd.com/archive/slurm-23.02.6/sbatch.html#OPT_gres). |
| `--partition`                                | `general` | The SLURM partition to which the job is submitted. Default is to use the `general` partition. |
| `--chdir`                                    |  | Set the working directory of the batch script before it's executed. Setting this using environment variables is not supported. |
| `--time`                                     | `0-01:00:00` | Defines the maximum time limit for job execution before it will be killed automatically. Format `DD-HH:MM:SS`. Maximum allowed value is that of the partition used. [Details here](https://slurm.schedmd.com/archive/slurm-23.02.6/sbatch.html#OPT_time) |
| `--mail-type`                                | `NONE` | Configures email notifications for certain job events. One or more comma-separated values of: `NONE`, `ALL`, `BEGIN`, `END`, `FAIL`, `REQUEUE`, `ARRAY_TASKS`. [Details here](https://slurm.schedmd.com/archive/slurm-23.02.6/sbatch.html#OPT_mail-type) |
| `--mail-user`                                | Local user | Specifies the email address where job notifications are sent. |
| `--x11`                                      | `all` | Enable forwarding of graphical applications from the job to your computer. It's required that you have either connected using the `ssh -X` option or you have set the `ForwardX11 yes` option in your [SSH config file](../../access/#ssh-config-file). For `salloc` or `srun` only. [Details here](https://slurm.schedmd.com/archive/slurm-23.02.6/srun.html#OPT_x11). |

Most options are self-explanatory. But for our setup and common use-cases you almost always want to set `--nodes` to 1, meaning your job will only run on a single compute node at a time. For multithreaded applications (most are nowadays) you mostly only need to set `ntasks` to `1` because threads are spawned from a single process (=task in SLURM parlor), and thus increase `--cpus-per-task` instead.

## How many resources should I request for my job(s)?
Exactly how many resources your job(s) need(s) is something you have to experiment with and learn over time based on past experience. It's important to do a bit of experimentation before submitting large jobs to obtain a qualified guess since the utilization of all the allocated resources across the cluster is ultimately based on people's own assessments alone. Below are some tips regarding CPU and memory, but in the end please always **only request what you need**, and no more. This is essential to optimize the resource utilization and efficiency of the entire cluster.

### CPUs/threads
In general the number of CPUs that you book only affects how long the job will take to finish and how many jobs can run concurrently. The only thing to really consider is how many CPUs you want to use for the particular job out of your max limit (see [Usage accounting](https://cmc-aau.github.io/biocloud-docs/slurm/accounting/#show-qos-info-and-limitations) for how to see the current limits). If you use all CPUs for one job, you can't start more jobs until the first one has finished, the choice is yours. But regardless, it's very important to ensure that your jobs actually fully utilize the allocated number of CPUs, so don't start a job with `20` allocated CPUs if you only set max threads for a certain tool to `10`, for example. It also depends very much on the specific software tools you use for the individual steps in a workflow and how they are implemented, so you are not always in full control of the utilization. Furthermore, if you run a workflow with many different steps each using different tools, they will likely not use resources in the same way, and some may not even support multithreading at all (like R, depending on the packages used) and thus only run in a single single-threaded process, for example. In this case it might be a good idea to either split the job into multiple jobs if they run for a long time, or use workflow tools that support cluster execution, for example [Snakemake](../guides/snakemake/intro.md) where you can define separate resource requirements for individual steps. This is also the case for memory usage.

#### Overprovisioning
Sometimes there is just no way around it, and **if you don't expect your job(s) to be very efficient, please submit to the overprovisioned `default-op` partition**, which is also the default. Overprovisioning simply means that SLURM will allocate more CPU's than available on each machine, so that more than one job will run on each CPU, ensuring that each physical CPU is actually utilized 100% and thus more people are happy!

#### Use `nproc` everywhere
The number of CPUs is not a hard limit like the physical amount of memory is, on the other hand, and SLURM will never exceed the maximum physical memory of each compute node. Instead jobs are killed if they exceed the allocated amount of memory for the job (only if no other jobs need the memory), or not be allowed to start in the first place. With CPUs the processes you run simply won't be able to detect any more CPUs than those allocated, hence it's handy to just use `nproc` within scripts to detect the number of available CPUs instead of manually setting a value for each tool. Furthermore, if you request more memory per CPU that the max allowed for the particular partition (refer to the [hardware overview](../index.md#slurm-partitions)), SLURM will automatically allocate more CPU's for the job, and hence, again, it's a good idea to detect the number of CPU's dynamically using `nproc` everywhere.

### Memory
Requesting a sensible maximum amount of memory is important to avoid crashing jobs. It's generally best to **allocate more memory** than what you need, so that the job doesn't crash and the spent resources don't go to waste and could have been used for something else anyways. To obtain a qualified guess you can start the job based on an initial expectation, and then set a job time limit of maybe 5-10 minutes just to see if it might crash due to exceeding the allocated memory, and if not you will see the maximum memory usage for the job in the email notification report (or use `seff <jobid>`). Then adjust accordingly and submit again with a little extra than what was used at maximum. Different steps of a workflow will in many cases, unavoidably, need more memory than others, so again, if they run for a long time, split it into multiple jobs or use [Snakemake](../guides/snakemake/intro.md).

Our compute nodes have plenty of memory, but some tools require lots of memory. If you know that your job is going to use a lot of memory (per CPU that is), you should likely submit the job to the `high-mem` partition. In order to fully utilize each compute node a general rule of thumb is to:

???+ tip "Rule of thumb for optimal memory usage"
      Request a **maximum** of **5GB per CPU** (1TB / 192t) if submitting jobs to the `general` partition. If you need more than that submit to the `high-mem` partition instead. If you request more memory per CPU than allowed for a particular partition, SLURM will automatically allocate more CPUs to scale accordingly, [details here](https://slurm.schedmd.com/slurm.conf.html#OPT_MaxMemPerCPU). It is therefore ideal to detect the number of CPUs available dynamically in your workflows using for example `nproc`.

If you know that you are almost going to fully saturate the memory on a compute node (depending on partition), you might as well also request more CPUs up to the total of a single compute node, since your job will likely allocate a full compute node alone, and CPU's can end up idle, while you could have finished the job faster. If needed you can also submit directly to the individual compute nodes specifically using the `nodelist` option (and potentially also `--exclusive`), refer to the [hardware overview](../index.md) for hostnames and compute node specs.

Also keep in mind that the effective amount of memory available to SLURM jobs is less than what the physical machines have available because they are virtual machines running on a hypervisor OS that also needs some memory. A 1 TB machine roughly has 950 GB available and the 2 TB ones have 1.9 TB. See `sinfo -N -l` for details of each node.
