# Job control and usage accounting
Below are some nice to know commands for controlling and checking up on running jobs, current and past.

## Get job status info
Use [`squeue`](https://slurm.schedmd.com/squeue.html), for example:
```
$ squeue
JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   24 general interact ksa@bio.  R       2:15      1 bio-oscloud04
```

??? "Job state codes (ST)"
      | Status	Code | Explaination |
      | --- | --- |
      | COMPLETED | CD | The job has completed successfully. |
      | COMPLETING | CG | The job is finishing but some processes are still active. |
      | FAILED | F | The job terminated with a non-zero exit code and failed to execute. |
      | PENDING | PD | The job is waiting for resource allocation. It will eventually run. |
      | PREEMPTED | PR | The job was terminated because of preemption by another job. |
      | RUNNING | R | The job currently is allocated to a node and is running. |
      | SUSPENDED | S | A running job has been stopped with its cores released to other jobs. |
      | STOPPED | ST | A running job has been stopped with its cores retained. |

      A complete list can be found in SLURM's [documentation](https://slurm.schedmd.com/squeue.html#lbAG)

??? "Job reason codes (REASON )"
      | Reason Code | Explaination |
      | --- | --- |
      | Priority | One or more higher priority jobs is in queue for running. Your job will eventually run. |
      | Dependency | This job is waiting for a dependent job to complete and will run afterwards. |
      | Resources | The job is waiting for resources to become available and will eventually run. |
      | InvalidAccount | The job’s account is invalid. Cancel the job and rerun with correct account. |
      | InvaldQoS | The job’s QoS is invalid. Cancel the job and rerun with correct account. |
      | QOSGrpCpuLimit | All CPUs assigned to your job’s specified QoS are in use; job will run eventually. |
      | QOSGrpMaxJobsLimit | Maximum number of jobs for your job’s QoS have been met; job will run eventually. |
      | QOSGrpNodeLimit | All nodes assigned to your job’s specified QoS are in use; job will run eventually. |
      | PartitionCpuLimit | All CPUs assigned to your job’s specified partition are in use; job will run eventually. |
      | PartitionMaxJobsLimit | Maximum number of jobs for your job’s partition have been met; job will run eventually. |
      | PartitionNodeLimit | All nodes assigned to your job’s specified partition are in use; job will run eventually. |
      | AssociationCpuLimit | All CPUs assigned to your job’s specified association are in use; job will run eventually. |
      | AssociationMaxJobsLimit | Maximum number of jobs for your job’s association have been met; job will run eventually. |
      | AssociationNodeLimit | All nodes assigned to your job’s specified association are in use; job will run eventually. |

      A complete list can be found in SLURM's [documentation](https://slurm.schedmd.com/squeue.html#lbAF)

## Prevent pending job from starting
Pending jobs can be marked in a "hold" state to prevent them from starting
```
scontrol hold <job_id>
```

To release a queued job from the ‘hold’ state :
```
scontrol release <job_id>
```

To cancel and rerun (requeue) a particular job:
```
scontrol requeue <job_id>
```

## Cancel a job
With `sbatch` you won't be able to just hit CTRL+c to stop what's running like you're used to in a terminal. Instead you must use `scancel`. Get the job ID from `squeue -u $(whoami)`, then use [`scancel`](https://slurm.schedmd.com/scancel.html) to cancel a running job, for example:
```
$ scancel 24
```

If the particular job doesn't stop and doesn't respond, consider using [`skill`](https://slurm.schedmd.com/skill.html) instead.

## Pause or resume a job
Use [`scontrol`](https://slurm.schedmd.com/scontrol.html) to control your own jobs, for example suspend a running job:
```
$ scontrol suspend 24
```

Resume again with
```
$ scontrol resume 24
```

## Modifying job attributes
Only a few job attributes can be changed after a job is submitted. These attributes include:

 - wall clock limit
 - job name
 - job dependency

For example:
```
$ scontrol update JobId=$JobID timelimit=<new timelimit>
```



## Job status information
Use [`sstat`](https://slurm.schedmd.com/sstat.html) to show the status and live usage accounting information of **running** jobs. For batch scripts you need to add `.batch` to the job ID, for example:
```
$ sstat 24.batch
```

This will print EVERY metric, so it's nice to select only a few most relevant ones, for example:

```
sstat --jobs24.batch --format=jobid,avecpu,maxrss,ntasks
```

??? "Useful format variables"
      | Variable | Description |
      | --- | --- |
      | avecpu | Average CPU time of all tasks in job. |
      | averss | Average resident set size of all tasks. |
      | avevmsize | Average virtual memory of all tasks in a job. |
      | jobid | The id of the Job. |
      | maxrss | Maximum number of bytes read by all tasks in the job. |
      | maxvsize | Maximum number of bytes written by all tasks in the job. |
      | ntasks | Number of tasks in a job. |
      
      For all variables see the [SLURM documentation](https://slurm.schedmd.com/sstat.html#SECTION_Job-Status-Fields)

## Job usage accounting
To view the status of **past** jobs and their usage accounting information use [`sacct`](https://slurm.schedmd.com/sacct.html). `sacct` will return **everything** accounted for by default which is very inconvenient to view in a terminal window, so below are only the most essential columns shown:
```
$ sacct -o jobid,jobname,start,end,NNodes,NCPUS,ReqMem,CPUTime,AveRSS,MaxRSS --user=$USER --units=G -j 138
JobID           JobName               Start                 End   NNodes      NCPUS     ReqMem    CPUTime     AveRSS     MaxRSS 
------------ ---------- ------------------- ------------------- -------- ---------- ---------- ---------- ---------- ---------- 
138          interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16        20G   00:02:56                       
138.interac+ interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56          0          0 
138.extern       extern 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56      0.00G      0.00G 

```

There are a huge number of other options to show, see [SLURM docs](https://slurm.schedmd.com/sacct.html#SECTION_Job-Accounting-Fields). If you really want to see everything use `sacct --long > file.txt` and dump it into a file or else it's too much for the terminal.
