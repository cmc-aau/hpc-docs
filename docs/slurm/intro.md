# Introduction to SLURM
SLURM (Simple Linux Utility for Resource Management) is a highly flexible and powerful job scheduler for managing and scheduling computational workloads on high-performance computing (HPC) clusters. SLURM is designed to efficiently allocate resources and manage job execution on clusters of any size, from a single server to tens of thousands. SLURM manages resources on an HPC cluster by dividing them into partitions. Users submit jobs to these partitions from a login-node, and then the SLURM controller schedules and allocates resources to those jobs based on available resources and user-defined constraints. SLURM also stores detailed usage information of all users' jobs in a usage accounting database, which allows enforcement of fair-share policies and priorities for job scheduling for each partition.

The BioCloud compute nodes are currently divided into two partitions with the same usage policies (currently no limit FIFO, first-in-first-out): the `biocloud-cpu` partition for CPU intensive jobs and the `biocloud-gpu` partition for jobs that benefit from a GPU. 

## BioCloud SLURM cluster overview
![SLURM overview](img/slurm-overview-inverted.png)

## Getting started
Start with obtaining shell access to either of the 3 login nodes `bio-oscloud[01-03].srv.aau.dk` as described in [Getting access](../access.md). To start with it's always nice to get an overview of the cluster, it's partitions, and how many ressources that are currently allocated. This is achieved with the `sinfo` command, example output:

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
               103 biocloud- 001-SG-0 jm12em@b  R    2:38:03      1 bio-oscloud04
               109 biocloud- sourdoug ft25sh@b  R    1:06:50      1 bio-oscloud04

# specific user (usually yourself)
$ squeue -u $(whoami)
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
               113 biocloud- minimap2 ksa@bio.  R       0:04      1 bio-oscloud04
```

Or get a more detailed overview of current ressource allocations and which jobs are running on all compute nodes with the wrapper script from [slurm_tools](https://github.com/OleHolmNielsen/Slurm_tools) `pestat`:
```
$ pestat
Hostname            Partition     Node Num_CPU  CPUload  Memsize  Freemem  Joblist
                                 State Use/Tot  (15min)     (MB)     (MB)  JobID User ...
bio-oscloud04   biocloud-cpu*     mix  160 192    8.11*   957078   815942  110 ft25sh@bio.aau.dk 112 ksa@bio.aau.dk
```

See `pestat -h` for more options.

## Live monitoring
For live monitoring of the whole cluster, the ressource utilization of individual nodes, number of SLURM jobs running etc, visit the [Grafana dashboard](http://bio-ospikachu04.srv.aau.dk:3000/) (only available on the internal AAU network).
