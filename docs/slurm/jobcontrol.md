# Job control and useful commands
Below are some nice to know commands for controlling and checking up on jobs, current and past.

## Get job status info
Use [`squeue`](https://slurm.schedmd.com/archive/slurm-23.02.6/squeue.html), for example:
```
$ squeue
JOBID         NAME       USER       TIME    TIME_LEFT CPU MIN_ME ST PARTITION NODELIST(REASON)
 2380         dRep ab12cd@bio 1-01:36:22  12-22:23:38  80   300G  R   general bio-oscloud02
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

      A complete list can be found in SLURM's [documentation](https://slurm.schedmd.com/archive/slurm-23.02.6/squeue.html#lbAG)

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

      A complete list can be found in SLURM's [documentation](https://slurm.schedmd.com/archive/slurm-23.02.6/squeue.html#lbAF)

The columns to show can be customized using the `--format` option, but can also be set with the environment variable `SQUEUE_FORMAT` to avoid typing it every time. You can always override this to suit your needs in your `.bashrc` file. The default format is currently:

```
SQUEUE_FORMAT="%.6i %.12j %.10u %.10M %.12L %.3C %.6m %.2t %.9P %R"
```

See a full list [here](https://slurm.schedmd.com/archive/slurm-23.02.6/squeue.html#OPT_format).

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
With `sbatch` you won't be able to just hit CTRL+c to stop what's running like you're used to in a terminal. Instead you must use `scancel`. Get the job ID from `squeue -u $(whoami)`, then use [`scancel`](https://slurm.schedmd.com/archive/slurm-23.02.6/scancel.html) to cancel a running job, for example:
```
$ scancel 24
```

If the particular job doesn't stop and doesn't respond, consider using [`skill`](https://slurm.schedmd.com/archive/slurm-23.02.6/skill.html) instead.

## Pause or resume a job
Use [`scontrol`](https://slurm.schedmd.com/archive/slurm-23.02.6/scontrol.html) to control your own jobs, for example suspend a running job:
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
Use [`sstat`](https://slurm.schedmd.com/archive/slurm-23.02.6/sstat.html) to show the status and live usage accounting information of **running** jobs. For batch scripts you need to add `.batch` to the job ID, for example:
```
$ sstat 24.batch
```

This will print EVERY metric, so it's nice to select only a few most relevant ones, for example:

```
$ sstat --jobs24.batch --format=jobid,avecpu,maxrss,ntasks
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
      
      For all variables see the [SLURM documentation](https://slurm.schedmd.com/archive/slurm-23.02.6/sstat.html#SECTION_Job-Status-Fields)

## Job efficiency metrics
To view the efficiency of individual jobs use `seff`, for example:

```
$ seff 2357
Job ID: 2357
Cluster: biocloud
User/Group: <redacted>
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 96
CPU Utilized: 60-11:06:29
CPU Efficiency: 45.76% of 132-02:48:00 core-walltime
Job Wall-clock time: 1-09:01:45
Memory Utilized: 383.42 GB
Memory Efficiency: 45.11% of 850.00 GB
```

This will also be shown in notification emails.
