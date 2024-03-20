# Job control and useful commands
Below are some nice to know commands for controlling and checking up on jobs, current and past.

## Get job status info
Use [`squeue`](https://slurm.schedmd.com/archive/slurm-23.02.6/squeue.html), for example:
```
$ squeue
JOBID         NAME       USER       TIME    TIME_LEFT CPU MIN_ME ST PARTITION NODELIST(REASON)
 2380         dRep ab12cd@bio 1-01:36:22  12-22:23:38  80   300G  R   general bio-oscloud02
```

To show only your own jobs use `squeue --me`. This is used quite often so `sq` has been made an alias of `squeue --me`.

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
SQUEUE_FORMAT="%.12i %.16j %.10u %.10M %.12L %.3C %.6m %.2t %.9P %R"
```

See a full list [here](https://slurm.schedmd.com/archive/slurm-23.02.6/squeue.html#OPT_format).

## Prevent pending job from starting
Pending jobs can be marked in a "hold" state to prevent them from starting
```
scontrol hold <job_id>
```

To release a queued job from the ‘hold’ or 'requeued held' states:
```
scontrol release <job_id>
```

To cancel and rerun (requeue) a particular job:
```
scontrol requeue <job_id>
```

## Cancel a job
With `sbatch` you won't be able to just hit CTRL+c to stop what's running like you're used to in a terminal. Instead you must use `scancel`. Get the job ID from `squeue --me`, then use [`scancel`](https://slurm.schedmd.com/archive/slurm-23.02.6/scancel.html) to cancel a running job, for example:
```
$ scancel <job_id>

# cancel ALL your jobs
$ scancel --me
```

If the particular job doesn't stop and doesn't respond, consider using [`skill`](https://slurm.schedmd.com/archive/slurm-23.02.6/skill.html) instead.

## Pause or resume a job
Use [`scontrol`](https://slurm.schedmd.com/archive/slurm-23.02.6/scontrol.html) to control your own jobs, for example suspend a running job:
```
$ scontrol suspend <job_id>
```

Resume again with
```
$ scontrol resume <job_id>
```

## Show details about a running or queued job
```
scontrol show jobid=<jobid>
```

## Modifying job attributes
Only a few job attributes can be changed after a job is submitted and **NOT** running yet. These attributes include:

 - wall clock limit
 - job name
 - job dependency
 - partition or QOS

For example:
```
$ scontrol update JobId=<jobid> timelimit=<new timelimit>
$ scontrol update JobId=<jobid> partition=high-mem
```

If the job is already running adjusting the time limit must be done by an administrator.
