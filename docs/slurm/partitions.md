# Hardware partitions
Before submitting a job you must carefully choose the correct hardware partition for it. To ensure hardware utilization is maximized the compute nodes are divided into separate partitions depending on their specs and for different use-cases. When submitting a job and choosing a hardware partition for it you should **always** consider:

 - The required memory **per CPU** (e.g. 10 CPUs and 20GB memory is 2GB per CPU). Only this ratio matters, the total memory requirement does **NOT** matter!
 - The expected CPU efficiency (i.e. do you expect to keep all allocated CPUs busy at 100% at all times?)
 - Whether you need to read/write lots of temporary files (thousands) and benefit from extra local scratch space ([more details here](../storage.md#local-scratch-space))
 - Whether you need to use special hardware like a GPU
 
 If in doubt just use the `default-op` for most things. The overall efficiency of the entire cluster depends entirely on how users submit jobs, and whether they actually use what they request. So please take great care in this regard - it impacts the productivity of all your collegues! The below flow diagram may help choosing the right partition for your needs.

## Choosing the correct partition
![test](img/biocloud_partition.drawio.svg)

!!! warning "Always detect available CPUs dynamically in scripts and workflows, never hard-code it!"
    It's very important to note that all partitions have a **max memory per CPU** configured, which may result in the scheduler allocating more CPU's for the job than requested until this ratio is satisfied. This is to ensure that no CPU's end up idle when a compute node is fully allocated in terms of memory, when it could have been at work finishing jobs faster instead. Therefore **NEVER** hardcode the number of threads/cores to use for individual software tools, instead make it a habit to detect it dynamically using fx `$(nproc)` in scripts and workflows.

## The `default-op` partition
The `default-op` partition is the default partition and is overprovisoned with a factor of 3, meaning each CPU can run up to 3 jobs at once. This is ideal for low to medium (<75%) efficiency, low to medium memory, and interactive jobs, as well as I/O jobs, that don't fully utilize the allocated CPU's 100% at all times. It's **highly recommended** to use this partition for most CPU intensive jobs unless you need lots of memory, or have taken extra care [optimizing the CPU efficiency](efficiency.md) of the job(s), in which case you can use the `general` or `high-mem` partitions instead and finish the jobs faster. Due to over-subscription it's actually possible to allocate a total of 1632 CPUs for jobs on this partition, however with less memory. The job time limit for this partition is 7 days.

**Max memory per CPU: 1.5GB**

| Hostname | vCPUs | Memory | Scratch space |
| :--- | :---: | :---: | :---: |
| `bio-oscloud01` | 96 | 0.5 TB | - |
| `bio-oscloud02` | 192 | 1 TB | - |
| `axomamma` | 256 | 1 TB | 3.5 TB |

## The `general` partition
The `general` partition is for high efficiency jobs only, since the CPU's are not shared among multiple jobs, but dedicated to each individual job. Therefore you must ensure that they are also fully utilized at all times, preferably 75-100%, otherwise please use the `default-op` partition instead if the memory (per CPU) is sufficient. The job time limit for this partition is 14 days.

**Max memory per CPU: 5GB**

| Hostname | vCPUs | Memory | Scratch space |
| :--- | :---: | :---: | :---: |
| `bio-oscloud[03-05]` | 192 | 1 TB | - |
| `bio-oscloud06` | 192 | 1 TB | 18TB |

## The `high-mem` partition
The `high-mem` partition is only for high efficiency jobs that also require lots of memory. Please do not submit anything here that doesn't require at least 5GB per CPU, otherwise please use the `general` partition. Like the `general` partition, the CPU's are dedicated to each individual job. Therefore you must ensure that they are also fully utilized at all times, preferably 75-100%, otherwise please use the `default-op` partition instead if the memory (per CPU) is sufficient. The job time limit for this partition is 28 days.

**Max memory per CPU: 10GB**

| Hostname | vCPUs | Memory | Scratch space |
| :--- | :---: | :---: | :---: |
| `bio-oscloud07` | 240 | 2 TB | - |
| `bio-oscloud08` | 192 | 2 TB | - |

## The `gpu` partition
This partition is ONLY for jobs that require a GPU. Please do not submit jobs to this partition if you don't need a GPU. There is otherwise no max memory per CPU or over-provisioning configured for this partition. The job time limit for this partition is 14 days.

| Hostname | vCPUs | Memory | Scratch space | GPU |
| :--- | :---: | :---: | :---: | :---: |
| `bio-oscloud09` | 64 | 256 GB | 3 TB | 2x nvidia A10 |
