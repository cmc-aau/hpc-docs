# WIP - Submitting jobs (SLURM)

## Introduction to SLURM

SLURM (Simple Linux Utility for Resource Management) is a highly flexible and powerful job scheduler for managing and scheduling computational workloads on high-performance computing (HPC) clusters. SLURM is designed to efficiently allocate resources and manage job execution on clusters of any size from a single server to tens of thousands.

## How SLURM Works

SLURM manages resources on an HPC cluster by dividing them into partitions. Users submit jobs to these partitions, and SLURM schedules and allocates resources to those jobs based on available resources and user-defined constraints. SLURM also has advanced usage accounting that enforces fairness and priorities for job scheduling.

### SLURM Components
- **Controller nodes**: Controls everything and collects usage, job state information, ressource allocation on each compute node etc
- **Compute nodes**: The raw compute nodes or slaves in the cluster. They do nothing but run users' jobs and report to the controller(s)
- **Login nodes**: User accessible frontend where jobs are submitted through the controller nodes and distributed to the compute nodes.
- **Partitions**: Logical groupings of compute nodes with for example similar characteristics (e.g. GPU, memory, etc), or to separate different accounting policies

## Submitting a slurm job
Mainly 3 commands depending on the needs, they all share the same options to define ressource constraints, job status notifications etc.
 - `srun`: Immediately request ressources and run a command/script in the foreground
 - `salloc`: Immediately request ressources and allocate them to the current shell session
 - `sbatch`: Hand over a job to slurm for later background execution when ressources are available.

### srun examples

### salloc examples

### sbatch examples

Example sbatch script
```bash
#!/usr/bin/env bash

# Can be useful to set a variable to ensure the same amount of threads/cpus is
# requested for the slurm job and used by all script commands downstream
max_threads=10

# Adjust job ressource allocation here
#SBATCH --job-name=minimap2test
#SBATCH --output=/user_data/ksa/slurmjobs/log%j.txt
#SBATCH --ntasks-per-node=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=${max_threads}
#SBATCH --mem=40G
#SBATCH --nodes=1
#SBATCH --time=60:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=abcd@bio.aau.dk

# Load required software modules or conda environments
module load Minimap2/2.15-foss-2018a

# Either run an external script or snakemake pipeline from somewhere else...
bash /path/to/shell/script.sh -i inputfilepath -o outputdir -s somesetting -t ${max_threads}

# ...or include everything here, for example:

# Maps some DNA to a database using minimap2 and writes out to nowhere (null-device)
input="/user_data/ksa/projects/slurmtest/inputdata/midasok2023sepbarcode63.fastq.gz"
database="/databases/midas/MiDAS5.1_20230726/output/FLASVs.fa"

minimap2 \
  -ax map-ont \
  -t "$max_threads" \
  --secondary=no \
  "$database" \
  -K20M "$input" > /dev/null
```

Submit it to slurm
```bash
sbatch my_job_script.sh
```

## Some commonly used settings
`--job-name=minimap2test`
`--output=/user_data/ksa/slurmjobs/log%j.txt`
`--ntasks-per-node=1`
`--ntasks=1`
`--cpus-per-task=${max_threads}`
`--mem=40G`
`--nodes=1`
`--time=60:00`
`--mail-type=ALL`
`--mail-user=abcd@bio.aau.dk`

[Cheat sheet here](https://slurm.schedmd.com/pdfs/summary.pdf).

### Cancelling or adjusting allocated ressources for a running job
```
# get job ID
squeue

# cancel job
scancel 24

# pause job
scontrol suspend 24

# resume job
scontrol resume 24

# adjust allocated ressources
scontrol update JobId=24 NumNodes=1 NumTasks=1 CPUsPerTask=1
```

### Checking up on the status of your jobs
```
$ squeue
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   24 biocloud- interact ksa@bio.  R       2:15      1 bio-ospikachu05
```

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

or `sstat -j 24`

```
$ sacct
JobID           JobName  Partition    Account  AllocCPUS      State ExitCode 
------------ ---------- ---------- ---------- ---------- ---------- -------- 
17           minimap2t+ biocloud-+ compute-a+          2  COMPLETED      0:0 
17.batch          batch            compute-a+          2  COMPLETED      0:0 
17.extern        extern            compute-a+          2  COMPLETED      0:0 
18           memorytest biocloud-+ compute-a+          0     FAILED      1:0 
19           memorytest biocloud-+ compute-a+          1  COMPLETED      0:0 
19.batch          batch            compute-a+          1  COMPLETED      0:0 
19.extern        extern            compute-a+          1  COMPLETED      0:0 
20           memorytest biocloud-+ compute-a+          1  COMPLETED      0:0 
20.batch          batch            compute-a+          1  COMPLETED      0:0 
20.extern        extern            compute-a+          1  COMPLETED      0:0 
21           memorytest biocloud-+ compute-a+          1     FAILED      1:0 
21.batch          batch            compute-a+          1     FAILED      1:0 
21.extern        extern            compute-a+          1  COMPLETED      0:0 
22           memoryfil+ biocloud-+ compute-a+          1 OUT_OF_ME+    0:125 
22.batch          batch            compute-a+          1 OUT_OF_ME+    0:125 
22.extern        extern            compute-a+          1  COMPLETED      0:0 
23           interacti+ biocloud-+ compute-a+          0     FAILED      1:0 
24           interacti+ biocloud-+ compute-a+          1     FAILED      2:0 
24.extern        extern            compute-a+          1  COMPLETED      0:0
```

when logging in, you have xxx ressources, dont run anything
use screen or tmux

how to test/development with few ressources

 - list of partitions (general, GPU)
 - sbatch scripts and examples, cheat sheets

https://hpc-aub-users-guide.readthedocs.io/en/latest/octopus/jobs.html

In order to use a GPU for the deep learning job (or other jobs that require GPU usage), the following flag must be specified in the job script:

`#SBATCH --gres=gpu`

R SLURM