# Requesting ressources and job submission
To request ressources through SLURM you need to be familiar with the following 3 commands depending on your needs. They all share the same options to define ressource constraints (number of CPUs, memory, GPU, etc), time limits, email for job status notifications etc (overview of the most [essential settings](#most-essential-options) is shown further down), but their use-case and how they work differ:

 - `salloc` (interactive): Request ressources and allocate them to a new shell session. Ressources remain allocated until the terminal session is exited (CTRL+d or `exit` or close window).
 - `sbatch` (non-interactive): Hand over a job to SLURM for later execution in the background whenever ressources become available. Ressources are freed for new jobs immediately once the script/command exits.
 - `srun`: Mostly only used to run parallel jobs within the context of a SLURM job where ressources are already allocated. `srun` will likely not be necessary for now.

## `salloc`
`salloc` is ideal for testing and development purposes where you need ressources for only a few hours or a work day, or if you need to experiment in the terminal with multiple commands without having to request ressources every single time with `srun`, for example:

```
$ salloc --ntasks 1 --cpus-per-task 10 --mem 10G
salloc: Granted job allocation 37
salloc: Waiting for resource configuration
salloc: Nodes bio-oscloud04 are ready for job

$ module load minimap2

$ minimap2 -t 10 database.fastq input.fastq > out.file
[M::mm_idx_gen::5.591*1.18] collected minimizers
[M::mm_idx_gen::6.954*1.34] sorted minimizers
[M::main::6.970*1.34] loaded/built the index for 120408 target sequence(s)
```

???+ Important
      When using `salloc` it's important to keep in mind that the allocated ressources remain reserved only for you until you `exit` the shell session. So don't leave it hanging for too long if you know you are not going to use it actively, otherwise other users might have needed the ressources.

## `sbatch` (recommended)
The `sbatch` is in many cases the best way to use SLURM. It's different in the way that the ressources are requested. It's done by `#SBATCH` comment-style lines in a shell script, and the script is then submitted to SLURM using an `sbatch script.sh` command. This is ideal for submitting large jobs that will run for many hours or days, but of course also for testing/development work. A full-scale example SLURM batch script could look like this:

**minimap2test.sh**
```bash
#!/usr/bin/bash -l
#SBATCH --job-name=minimap2test
#SBATCH --output=job_%j_%x.txt
#SBATCH --ntasks-per-node=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=10
#SBATCH --mem=10G
#SBATCH --nodes=1
#SBATCH --time=60:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=abc@bio.aau.dk

# load software modules or environments
module load minimap2

# run one or more commands as part a full pipeline script,
# or call scripts from elsewhere. Make sure you use the same
# number of max threads as requested 
minimap2 -t 10 database.fastq input.fastq > out.file
```

???+ Important
      The `bash -l` in the top "shebang" line is required for the compute nodes to be able to load conda environments correctly.

Submit the batch script to the SLURM job queue using `sbatch minimap2test.sh`, and it will then start once the requested amount of ressources are available (also taking into account your past usage and priorities of other jobs etc, all 3 job submission commands do that). If you set the `--mail-user` and `--mail-type` arguments you should get a notification email once the job starts and finishes with additional details like how many ressources you have actually used compared to what you have requested. This is essential information for future jobs to avoid overbooking. As the job is handled by slurm in the background by the SLURM daemons on the individual compute nodes you won't see any output to the terminal, it will instead be written to the file defined by `--output`. To follow along use `tail -f /user_data/abc/slurmjobs/job_123.txt`. The line `--output=job_%j_%x.txt` above would result in an output file named job_xx_minimap2test.txt in the current working directory.


## Most essential options
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
| --gres               | List of "generic consumable ressources" to use, for example a GPU. |
| --partition          | The SLURM partition to which the job is submitted. Default is to use the `biocloud-cpu` partition. |
| --time               | Defines the maximum time limit for job execution. It can be expressed in minutes, hours, or days. [Details here](https://slurm.schedmd.com/sbatch.html#OPT_time)          |
| --mail-type          | Configures email notifications for job events such as "BEGIN", "END", "FAIL", or "ALL". [Details here](https://slurm.schedmd.com/sbatch.html#OPT_mail-type)                       |
| --mail-user          | Specifies the email address where job notifications are sent.                                                |

Most options are self-explanatory. But for our setup and common use-cases you almost always want to set `--nodes` to 1, meaning your job will only run on a single compute node at a time. For multithreaded applications (most are nowadays) you mostly only need to set `ntasks` to `1` because threads are spawned from a single process (=task in SLURM parlor), and then increase `--cpus-per-task` instead.

Jobs that will spawn many parallel processes, fx when using GNU `parallel` or `xargs`, will require you to increase `ntasks` instead and set `--cpus-per-task` to `1`, as many independent processes will be launched that likely don't use any multithreading (depending on your exact command(s)/script). If they also use multithreading you must also increase `--cpus-per-task`, and the total number of CPU allocated by slurm will thus be `cpus-per-task * ntasks`.

??? "Jobs that span multiple compute nodes"
      If needed the BioCloud is properly set up with the `OpenMPI` and `PMIx` message interfaces for distributed work across multiple compute nodes, but it requires you to [tailor your scripts and commands](https://curc.readthedocs.io/en/latest/programming/parallel-programming-fundamentals.html) specifically for distributed work and is a topic for another time. You can run "brute-force parallel" jobs, however, using for example [GNU parallel](https://curc.readthedocs.io/en/latest/software/GNUParallel.html) and distribute them across nodes, but this is only for experienced users and they must figure that out for themselves for now.

If you need to use one or more GPUs you need to specify `--partition=biocloud-gpu` and set `--gres=gpu:x`, where `x` refers to the number of GPUs you need. Please don't do CPU work on the `biocloud-gpu` partition unless you also need a GPU.

## How many ressources should I request for my job(s)?
Exactly how many ressources your job(s) need(s) is something you have to experiment with and learn over time based on past experience. It's important to do a bit of experimentation before submitting large jobs to obtain a qualified guess since the utilization of all the allocated ressources across the cluster is ultimately based on people's own assessments alone. Below are some tips regarding CPU and memory.

### CPUs/threads
In general the number of CPUs that you book only affects how long the job will take to finish. Since most tools don't use 100% of each and every allocated thread throughout the duration (due to for example I/O delays, internal thread communication, single-threaded job steps etc), our partitions are set with an **oversubscription factor of 1.5** (not yet, just preparing docs for it) to optimize ressource utilization. This means that SLURM will in total allocate more CPUs than there are physical cores or hyper-threads on each compute node. For example, SLURM will allocate up to 288 CPUs on a compute node with 192 threads across all SLURM jobs on the node. The number of threads is not a hard limit like the physical amount of memory is, on the other hand, and SLURM will never exceed the maximum physical memory of each compute node. Instead jobs are killed if they exceed the allocated amount of memory for the job, or not be allowed to start in the first place.

### Memory
Requesting a sensible maximum amount of memory is important to avoid crashing jobs. It's generally best to **allocate more memory** than what you need, so that the job doesn't crash and the spent ressources don't go to waste and could have been used for something else. To obtain a qualified guess you can start the job based on an initial expectation, and then set a job time limit of maybe 5-10 minutes just to see if it might crash due to exceeding the allocated ressources, and if not you will see the maximum memory usage for the job in the email notification report. Then adjust accordingly and submit again with 10-15% extra than what was used at maximum. Different steps of a workflow will in many cases unavoidably need more memory than others, and so it might be a good idea to either split the job into multiple jobs, or use workflow tools that support cluster execution, for example [snakemake](https://snakemake.readthedocs.io/en/stable/executing/cluster.html).

Our compute nodes have plenty of memory, but some tools require lots of memory. If you know that your job is going to use a lot of memory, say 1TB, you might as well also request more CPUs, since your job will likely allocate a full compute node alone, and you can then finish the job faster. This of course depends on which compute node your job is allocated to, so you might want to request ressources on individual compute nodes specifically using the `nodelist` option, refer to the [hardware overview](../index.md).
