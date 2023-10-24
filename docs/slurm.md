# WIP - Submitting jobs (SLURM)

## Introduction to SLURM

SLURM (Simple Linux Utility for Resource Management) is a highly flexible and powerful job scheduler for managing and scheduling computational workloads on high-performance computing (HPC) clusters. SLURM is designed to efficiently allocate resources and manage job execution on clusters.

## How SLURM Works

SLURM manages resources on an HPC cluster by dividing them into partitions. Users submit jobs to these partitions, and SLURM schedules and allocates resources to those jobs based on available resources and user-defined constraints. SLURM also enforces fairness and priorities for job scheduling.

### SLURM Components
- **Nodes**: The compute servers in the cluster.
- **Partitions**: Logical groupings of nodes with similar characteristics (e.g., CPU architecture).
- **Jobs**: Computational tasks submitted by users.
- **Schedulers**: SLURM's decision-making component responsible for job allocation.

## Basic SLURM Commands

### Submitting a Job

```bash
sbatch my_job_script.sh
```

  - `sbatch` is a slurm command

### Checking up on the status of your jobs


when logging in, you have xxx ressources, dont run anything
use screen or tmux

how to test/development with few ressources

 - list of partitions (general, GPU)
 - sbatch scripts and examples, cheat sheets

https://hpc-aub-users-guide.readthedocs.io/en/latest/octopus/jobs.html

In order to use a GPU for the deep learning job (or other jobs that require GPU usage), the following flag must be specified in the job script:

`#SBATCH --gres=gpu`

R SLURM