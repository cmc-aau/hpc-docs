# Other commands / FAQ
Below are some nice to know commands with example output and some common problems. This will continously be populated as people ask for certain things. **Your question here!**

## I'm not allowed to submit jobs
```
salloc: error: Job submit/allocate failed: Invalid account or account/partition combination specified
```

This error means you have not been associated with any usage account yet, so you must contact an administrator to add your user to the correct account.

## How can I get a more detailed overview of the job queue and their requested ressources
```
squeue -o "%.18i %Q  %.8j %.8u %.2t %.10M %.10L %.6C %m %R"
             JOBID PRIORITY      NAME     USER ST       TIME  TIME_LEFT   CPUS MIN_MEMORY NODELIST(REASON)
                 9 4294901751      test user01@b PD       0:00 14-00:00:00     40 300G (Resources)
                10 4294901750  minimap2 user02@b PD       0:00    1:00:00      4 40G (Nodes required for job are DOWN, DRAINED or reserved for jobs in higher priority partitions)
                11 4294901749      test user01@b PD       0:00 14-00:00:00      4 10G (Priority)
                 8 4294901752      test user01@b  R      20:35 13-23:39:25     40 0 bio-oscloud04
```

## Show busy/free cores for the entire cluster
Example output (A=allocated, I=idle, O=other, T=total):
```
$ sinfo -o "%C"
CPUS(A/I/O/T)
10/374/0/38
```

## Generate a usage report
```
$ sreport cluster UserUtilizationByAccount -t Hours  Users=$USER
--------------------------------------------------------------------------------
Cluster/User/Account Utilization 2023-11-20T00:00:00 - 2023-11-20T23:59:59 (86400 secs)
Usage reported in CPU Hours
--------------------------------------------------------------------------------
  Cluster     Login     Proper Name         Account     Used   Energy 
--------- --------- --------------- --------------- -------- -------- 
 biocloud abc@bio.+             ksa compute-account        6        0 

```

## Show reserved compute nodes
```
$ sinfo -T
RESV_NAME      STATE           START_TIME             END_TIME     DURATION  NODELIST
group1       ACTIVE  2021-08-31T10:23:01  2021-12-15T09:23:01  106-00:00:00  node[01,02]
```

## Show details about a particular job
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
   Partition=general AllocNode:Sid=bio-ospikachu02:340145
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

## Show details about the whole cluster configuration
```
$ scontrol show config
```

## SLURM environment variables
SLURM jobs will have a variety of environment variables set within job allocations, which might become handy programmatically. Below is an overview of relevant ones. For all of them refer to the SLURM documentation for [https://slurm.schedmd.com/sbatch.html#SECTION_INPUT-ENVIRONMENT-VARIABLES](input environment variables) and [https://slurm.schedmd.com/sbatch.html#SECTION_OUTPUT-ENVIRONMENT-VARIABLES](output environment variables). Some may not be present for your particular job, to list only those currently available within a job run for example `env | grep -iE 'SLURM|SBATCH'`.

| Variable(s) | Description |
| :--- | :--- |
| `SLURM_ARRAY_TASK_COUNT` | Total number of tasks in a job array |
| `SLURM_ARRAY_TASK_ID` | Job array ID (index) number |
| `SLURM_ARRAY_TASK_MAX` | Job array's maximum ID (index) number |
| `SLURM_ARRAY_TASK_MIN` | Job array's minimum ID (index) number |
| `SLURM_ARRAY_TASK_STEP` | Job array's index step size |
| `SLURM_ARRAY_JOB_ID` | Job array's master job ID number |
| `SLURM_CLUSTER_NAME` | Name of the cluster on which the job is executing |
| `SLURM_CPUS_ON_NODE` | Number of CPUS on the allocated node |
| `SLURM_CPUS_PER_TASK` | Number of cpus requested per task. Only set if the  |--cpus-per-task option is specified.
| `SLURM_JOB_ACCOUNT` | Account name associated of the job allocation |
| `SLURM_JOBID`, `SLURM_JOB_ID` | The ID of the job allocation |
| `SLURM_JOB_CPUS_PER_NODE` | Count of processors available to the job on this  |node.
| `SLURM_JOB_DEPENDENCY` | Set to value of the --dependency option |
| `SLURM_JOB_NAME` | Name of the job |
| `SLURM_NODELIST`, `SLURM_JOB_NODELIST` | List of nodes allocated to the job |
| `SLURM_NNODES`, `SLURM_JOB_NUM_NODES` | Total number of different nodes in the  |job's resource allocation
| `SLURM_MEM_PER_NODE` | Takes the value of --mem if this option was specified. |
| `SLURM_MEM_PER_CPU` | Takes the value of --mem-per-cpu if this option was  |specified.
| `SLURM_NTASKS`, `SLURM_NPROCS` | Same as -n or --ntasks if either of these  |options was specified.
| `SLURM_NTASKS_PER_NODE` | Number of tasks requested per node. Only set if the  |--ntasks-per-node option is specified.
| `SLURM_NTASKS_PER_SOCKET` | Number of tasks requested per socket. Only set if  |the --ntasks-per-socket option is specified.
| `SLURM_SUBMIT_DIR` | The directory from which sbatch was invoked |
| `SLURM_SUBMIT_HOST` | The hostname of the computer from which sbatch was  |invoked
| `SLURM_TASK_PID` | The process ID of the task being started |
| `SLURMD_NODENAME` | Name of the node running the job script |
| `SLURM_JOB_GPUS` | GPU IDs allocated to the job (if any). |
