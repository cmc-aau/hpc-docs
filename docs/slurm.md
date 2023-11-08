# Requesting ressources through SLURM

## Introduction to SLURM
SLURM (Simple Linux Utility for Resource Management) is a highly flexible and powerful job scheduler for managing and scheduling computational workloads on high-performance computing (HPC) clusters. SLURM is designed to efficiently allocate resources and manage job execution on clusters of any size, from a single server to tens of thousands. SLURM manages resources on an HPC cluster by dividing them into partitions. Users submit jobs to these partitions from a login-node, and then the SLURM controller schedules and allocates resources to those jobs based on available resources and user-defined constraints. SLURM also stores detailed usage information of all users' jobs in a usage accounting database, which allows enforcement of fair-share policies and priorities for job scheduling for each partition. The BioCloud servers are currently divided into two partitions with the same usage policies (currently no limit FIFO, first-in-first-out): the `biocloud-cpu` for CPU intensive jobs and the `biocloud-gpu` for jobs that benefit from a GPU.

**Overview figure here**

### SLURM Components
- **Controller nodes**: Controls everything and collects usage, job state information, ressource allocation on each compute node etc
- **Compute nodes**: The raw compute nodes or slaves in the cluster. They do nothing but run users' jobs and report back to the controller(s)
- **Login nodes**: User accessible frontend where jobs are submitted through the controller nodes and distributed to the compute nodes.
- **Partitions**: Logical groupings of compute nodes with for example similar characteristics (e.g. GPU, memory, etc), or to separate different accounting policies

## Getting an overview
To start with it's always nice to get an overview of the cluster, it's partitions, and how many ressources that are currently allocated. This is achieved with the `sinfo` command, example output:

```
$ sinfo
PARTITION     AVAIL  TIMELIMIT  NODES  STATE NODELIST
biocloud-cpu*    up 14-00:00:0      1   idle bio-oscloud04
```

To get an overview of running jobs use `squeue`, example output:
```
# everything
$ squeue
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   24 biocloud- interact ksa@bio.  R       2:15      1 bio-oscloud04

# specific user (usually yourself)
$ squeue -u $(whoami)
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   35 biocloud- interact ksa@bio.  R       8:28      1 bio-oscloud04
```

To get live information about the whole cluster, the ressource utilization of individual nodes, number of SLURM jobs running etc, visit the [Grafana dashboard](http://bio-ospikachu04.srv.aau.dk:3000/).

## Requesting ressources and job submission
To request ressources through SLURM you need to be familiar with the following 3 commands depending on your needs. They all share the same options to define ressource constraints (number of CPUs, memory, GPU, etc), time limits, email for job status notifications etc (overview of the most [essential settings](#most-commonly-used-settings) is shown further down), but their use-case and how they work differ:

 - `srun` (interactive): Request ressources and run a command/script in the foreground once ressources become available. Ressources are freed for new jobs immediately once the command exits.
 - `salloc` (interactive): Request ressources and allocate them to a new shell session. Ressources remain allocated until the terminal session is exited (CTRL+d or `exit` or close window).
 - `sbatch` (non-interactive): Hand over a job to SLURM for later execution in the background whenever ressources become available. Ressources are freed for new jobs immediately once the script/command exits.

### `srun` example
`srun` is ideal for running quick commands that don't take too long, for example when testing parts of a script on toy-data or similar, because it runs in the foreground (closing window or a momentary disconnect will stop the command), for example:

```
# first load required software module or conda environment etc
# BEFORE the srun command. If the command is a script it can also
# be done in the script instead
$ module load minimap2

# request 10 cpus and 10GB total memory for running
# a simple minimap2 command (1 process/task)
$ srun --ntasks 1 --cpus-per-task 10 --mem 10G minimap2 -t 10 database.fastq input.fastq > out.file
salloc: Granted job allocation 36
salloc: Waiting for resource configuration
salloc: Nodes bio-oscloud04 are ready for job
[M::mm_idx_gen::5.591*1.18] collected minimizers
[M::mm_idx_gen::6.954*1.34] sorted minimizers
[M::main::6.970*1.34] loaded/built the index for 120408 target sequence(s)
...
```

### `salloc` example
`salloc` is ideal for testing and development purposes where you need ressources for only a few hours or a work day, or if you need to experiment in the terminal with multiple commands without having to request ressources every single time with `srun`, for example:

```
# request ressources first, 10 CPUs and 10GB total memory
$ salloc --ntasks 1 --cpus-per-task 10 --mem 10G
salloc: Granted job allocation 37
salloc: Waiting for resource configuration
salloc: Nodes bio-oscloud04 are ready for job

# Now everything you run will be run on the allocated node

# load required software module or conda environment etc
module load minimap2

# run a simple minimap2 command (1 process/task)
$ minimap2 -t 10 database.fastq input.fastq > out.file
[M::mm_idx_gen::5.591*1.18] collected minimizers
[M::mm_idx_gen::6.954*1.34] sorted minimizers
[M::main::6.970*1.34] loaded/built the index for 120408 target sequence(s)
...
```

When using `salloc` it's important to keep in mind that the allocated ressources remain reserved only for you until you `exit` the shell session. So don't leave it hanging for too long if you know you are not going to use it actively, otherwise other users might have needed the ressources.

### `sbatch` example
The `sbatch` is in many cases the best way to use SLURM. It's different in the way that the ressources are requested. It's done by `#SBATCH` comment-style lines in a shell script, and the script is then submitted to SLURM using an `sbatch script.sh` command. This is ideal for submitting large jobs that will run for many hours or days, but of course also for testing/development work. A full-scale example SLURM batch script could look like this:

**minimap2test.sh**
```bash
#!/usr/bin/env bash

# set a max_threads variable to ensure the same amount of threads/CPUs is 
# both allocated and used throughout the script
max_threads=10

#SBATCH --job-name=minimap2test
#SBATCH --output=/user_data/abc/slurmjobs/job_%j.txt
#SBATCH --ntasks-per-node=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=${max_threads}
#SBATCH --mem=10G
#SBATCH --nodes=1
#SBATCH --time=60:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=abc@bio.aau.dk

# load software modules or environments
module load minimap2

# run one or more commands as part a full pipeline script,
# or call scripts from elsewhere
minimap2 -t ${max_threads} database.fastq input.fastq > out.file
```

Submit the batch script to the SLURM job queue using `sbatch minimap2test.sh`, and it will then start once the requested amount of ressources are available (also taking into account your past usage and priorities of other jobs etc, all 3 job submission commands do that). If you set the `--mail-user` and `--mail-type` arguments you should get a notification email once the job starts and finishes with additional details like how many ressources you have actually used compared to what you have requested. This is essential information for future jobs to avoid overbooking. As the job is handled by slurm in the background by the SLURM daemons on the individual compute nodes you won't see any output to the terminal, it will instead be written to the file defined by `--output`. To follow along use `tail -f /user_data/abc/slurmjobs/job_123.txt`.

### Most essential settings
There are plenty of options with the SLURM job submission commands, but below are the most important ones for our current setup and common use-cases. If you need anything else you can start with the [SLURM cheatsheet](https://slurm.schedmd.com/pdfs/summary.pdf), or else refer to the SLURM documentation for the individual commands [`srun`](https://slurm.schedmd.com/srun.html), [`salloc`](https://slurm.schedmd.com/salloc.html), and [`sbatch`](https://slurm.schedmd.com/sbatch.html).

| Option               | Description                                                                                                  |
| -------------------  | ------------------------------------------------------------------------------------------------------------  |
| --job-name           | A user-defined name for the job or task. This name helps identify the job in logs and accounting records.    |
| --begin              | Specifies a start time for the job to begin execution. Jobs won't start before this time.                  |
| --output             | Defines the name of the output file for the job's standard output (stdout) ideally on shared storage, see filename patterns [here](https://slurm.schedmd.com/sbatch.html#SECTION_%3CB%3Efilename-pattern%3C/B%3E).                                |
| --ntasks-per-node    | Specifies the number of tasks to be launched per allocated compute node.                                     |
| --ntasks             | Indicates the total number of tasks or processes that the job should execute.                               |
| --cpus-per-task      | Sets the number of CPU cores allocated per task. Required for parallel and multithreaded applications.         |
| --mem                | Specifies the memory limit per node or per task for the job.             |
| --nodes              | Indicates the total number of compute nodes to be allocated for the job.                                    |
| --nodelist           | Specifies a comma-separated list of specific compute nodes to be allocated for the job.                     |
| --gres               | List of generic consumable ressources to use. This is only needed if you need to use a GPU. |
| --partition          | The SLURM partition to which the job is submitted. Default is to use the `biocloud-cpu` partition. |
| --time               | Defines the maximum time limit for job execution. It can be expressed in minutes, hours, or days.          |
| --mail-type          | Configures email notifications for job events such as "BEGIN," "END," "FAIL," etc.                         |
| --mail-user          | Specifies the email address where job notifications are sent.                                                |

Most options are self-explanatory. But for our setup and common use-cases you almost always want to set `--nodes` to 1, meaning your job will only run on a single compute node at a time. For multithreaded applications you mostly only need to set `ntasks` to `1` because threads are spawned from a single process/task, and then increase `--cpus-per-task` instead. For jobs that will run many things in parallel, fx when using GNU `parallel` or `xargs`, you must instead increase `ntasks` accordingly and set `--cpus-per-task` to `1`, because multiple and independent processes will be launched that likely don't use any multithreading.

Should it be needed the BioCloud is properly set up with the `OpenMPI` and `PMIx` message interfaces for distributed work across multiple compute nodes, but it requires you to [tailor your scripts and commands](https://curc.readthedocs.io/en/latest/programming/parallel-programming-fundamentals.html) specifically for distributed work and is a topic for another time. You can run "brute-force parallel" jobs, however, using for example [GNU parallel](https://curc.readthedocs.io/en/latest/software/GNUParallel.html) and distribute them across nodes, but this is only for experienced users and they must figure that out for themselves for now.

If you need to use one or more GPUs you need to specify `--partition=biocloud-gpu` and set `--gres=gpu:x`, where `x` refers to the number of GPUs you need. Please don't do CPU work on the `biocloud-gpu` partition unless you also need a GPU.

## Job control, status and usage accounting
Below are some nice to know commands for controlling and checking up on running jobs, current and past.

### Get job status info
Use [`squeue`](https://slurm.schedmd.com/squeue.html), for example:
```
$ squeue
```

Overview of status codes available [here](https://curc.readthedocs.io/en/latest/running-jobs/squeue-status-codes.html).

### Cancel a job
Get the job ID from `squeue -u $(whoami)`, then use [`scancel`](https://slurm.schedmd.com/scancel.html), for example:
```
$ scancel 24
```

### Pause or resume a job
Use [`scontrol`](https://slurm.schedmd.com/scontrol.html) to control your own jobs, for example suspend a running job:
```
$ scontrol suspend 24
```

Resume again with
```
$ scontrol resume 24
```

### Adjust allocated ressources
It's also possible to adjust allocated ressources to free them up for others to use without having to stop anything, for example:
```
$ scontrol update JobId=24 NumNodes=1 NumTasks=1 CPUsPerTask=1
```

### Job status information
Use [`sstat`](https://slurm.schedmd.com/sstat.html) to show the status and exit codes of your current and past jobs.
```
$ sstat
```

With additional details:
```
sstat --jobs=your_job-id --format=jobid,cputime,maxrss,ntasks
```

Useful format variables:

| Variable | Description |
| --- | --- |
| avecpu | Average CPU time of all tasks in job. |
| averss | Average resident set size of all tasks. |
| avevmsize | Average virtual memory of all tasks in a job. |
| jobid | The id of the Job. |
| maxrss | Maximum number of bytes read by all tasks in the job. |
| maxvsize | Maximum number of bytes written by all tasks in the job. |
| ntasks | Number of tasks in a job. |

### Job usage accounting
To see usage accounting information about jobs use [`sacct`](https://slurm.schedmd.com/sacct.html):
```
$ sacct
```

## How many ressources should I request for my job(s)?
Exactly how many ressources your job(s) need(s) is something you have to experiment with and learn over time based on past experience. It's important to do a bit of experimentation before submitting large jobs to obtain a qualified guess since the utilization of all the allocated ressources across the cluster are ultimately based on people's own assessments alone. Below are some tips regarding CPU and memory.

### CPUs/threads
In general the number of CPUs that you book only affects how long the job will take to finish. Since most tools don't use 100% of each and every allocated thread throughout the duration (due to for example I/O delays, internal thread communication, single-threaded job steps etc), our partitions are set with an **oversubscription factor of 1.5** (not yet, just preparing docs for it) to optimize ressource utilization. This means that SLURM will in total allocate more CPUs than there are physical cores or hyper-threads on each compute node. For example, SLURM will allocate up to 288 CPUs on a compute node with 192 threads across all SLURM jobs on the node. The number of threads is not a hard limit like the physical amount of memory is, on the other hand, and SLURM will never exceed the maximum physical memory of each compute node. Instead jobs are killed if they exceed the allocated amount of memory for the job or not be allowed to start in the first place.

### Memory
Requesting a sensible maximum amount of memory is important to avoid crashing jobs. It's generally best to **allocate more memory** than what you need, so that the job doesn't crash and the spent ressources don't go to waste and could have been used for something else. To obtain a qualified guess you can start the job based on an initial expectation, and then set a job time limit of maybe 5-10 minutes just to see if it might crash due to exceeding the allocated ressources, and if not you will see the maximum memory usage in the email notification report. Then adjust accordingly and submit again with 10-15% extra than what was used at maximum. Certain steps of a workflow will obviously use more than others, so either request a maximum across all steps, split the job into multiple jobs, or use workflow tools that support cluster execution, for example [snakemake](https://snakemake.readthedocs.io/en/stable/executing/cluster.html).

Our compute nodes have plenty of memory, but some tools require lots of memory. If you know that your job is going to use a lot of memory, say 1TB, you might as well also request more CPUs, since your job will likely allocate a full compute node alone and you can then finish the job faster. This of course depends on which compute node your job is allocated to, so you might want to request ressources on individual compute nodes specifically using the `nodelist` option, refer to the [hardware overview](index.md).

## FAQ
**How can I get a more detailed overview of the job queue and their requested ressources**
```
squeue -o "%.18i %Q  %.8j %.8u %.2t %.10M %.10L %.6C %m %R"
             JOBID PRIORITY      NAME     USER ST       TIME  TIME_LEFT   CPUS MIN_MEMORY NODELIST(REASON)
                 9 4294901751      test jm12em@b PD       0:00 14-00:00:00     40 300G (Resources)
                10 4294901750  minimap2 ksa@bio. PD       0:00    1:00:00      4 40G (Nodes required for job are DOWN, DRAINED or reserved for jobs in higher priority partitions)
                11 4294901749      test jm12em@b PD       0:00 14-00:00:00      4 10G (Priority)
                 8 4294901752      test jm12em@b  R      20:35 13-23:39:25     40 0 bio-oscloud04
```

**Show details about a particular job**
```
$ scontrol show job 24
JobId=24 JobName=interactive
   UserId=ksa@bio.aau.dk(101632) GroupId=ksa@bio.aau.dk(101632) MCS_label=N/A
   Priority=4294901738 Nice=0 Account=compute-account QOS=normal
   JobState=RUNNING Reason=None Dependency=(null)
   Requeue=1 Restarts=0 BatchFlag=0 Reboot=0 ExitCode=0:0
   RunTime=00:02:00 TimeLimit=14-00:00:00 TimeMin=N/A
   SubmitTime=2023-11-01T11:20:01 EligibleTime=2023-11-01T11:20:01
   AccrueTime=Unknown
   StartTime=2023-11-01T11:20:01 EndTime=2023-11-15T11:20:01 Deadline=N/A
   SuspendTime=None SecsPreSuspend=0 LastSchedEval=2023-11-01T11:20:01 Scheduler=Main
   Partition=biocloud-cpu AllocNode:Sid=bio-ospikachu02:340145
   ReqNodeList=(null) ExcNodeList=(null)
   NodeList=bio-ospikachu05
   BatchHost=bio-ospikachu05
   NumNodes=1 NumCPUs=1 NumTasks=1 CPUs/Task=1 ReqB:S:C:T=0:0:*:*
   ReqTRES=cpu=1,mem=10G,node=1,billing=1
   AllocTRES=cpu=1,mem=10G,node=1,billing=1
   Socks/Node=* NtasksPerN:B:S:C=0:0:*:1 CoreSpec=*
   MinCPUsNode=1 MinMemoryNode=10G MinTmpDiskNode=0
   Features=(null) DelayBoot=00:00:00
   OverSubscribe=OK Contiguous=0 Licenses=(null) Network=(null)
   Command=/bin/bash
   WorkDir=/user_data/ksa/projects/slurmtest
   Power=
```

**Show details about the whole cluster configuration**
```
$ scontrol show config
```