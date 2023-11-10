# Introduction to SLURM
SLURM (Simple Linux Utility for Resource Management) is a highly flexible and powerful job scheduler for managing and scheduling computational workloads on high-performance computing (HPC) clusters. SLURM is designed to efficiently allocate resources and manage job execution on clusters of any size, from a single server to tens of thousands. SLURM manages resources on an HPC cluster by dividing them into partitions. Users submit jobs to these partitions from a login-node, and then the SLURM controller schedules and allocates resources to those jobs based on available resources and user-defined constraints. SLURM also stores detailed usage information of all users' jobs in a usage accounting database, which allows enforcement of fair-share policies and priorities for job scheduling for each partition. The BioCloud servers are currently divided into two partitions with the same usage policies (currently no limit FIFO, first-in-first-out): the `biocloud-cpu` for CPU intensive jobs and the `biocloud-gpu` for jobs that benefit from a GPU.

**Overview figure here**

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
