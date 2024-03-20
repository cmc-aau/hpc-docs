# Hardware overview
Welcome to the documentation website for the BioCloud HPC at the section of biotechnology at Aalborg University. Here you will (hopefully) find everything you need to use the cluster for large scale bioinformatic workflows and other things that require more compute resources and storage.

The current setup consists of data center servers with dual socket AMD Epyc CPU's of various sizes (from 2022) with a total of **1848 vCPUs** and **12 TB RAM**, all connected to a **2.3 PB+ storage** cluster ([Ceph](https://ceph.com/)). Most servers/VMs in the cluster are set up with the [SLURM workload manager](https://slurm.schedmd.com/archive/slurm-23.02.6/overview.html) to effectively manage the compute resources for more than a hundred users.

## Cloud compute
Optimized for heavy parallel CPU workloads that may require lots of memory and storage. One machine also has 2x NVIDIA A10 GPU's for smaller jobs that benefit from GPU acceleration. A few servers have local scratch storage for faster I/O and to avoid overburdening the [ceph storage cluster](storage.md) when lots (millions) of small files need to be written. The servers are located in an enterprise data center at AAU with 25 gbps network, and they are only accessible remotely from the local AAU network or VPN. The `axomamma.srv.aau.dk` server is the only bare-metal machine, and it's not managed by Slurm as it's reserved for GUI apps like CLC and Arb. The rest are identical virtual machines (VM's) in a 1:1 physical to virtual CPU configuration. All servers run Ubuntu 22.04 LTS.

### SLURM partitions
The compute nodes are divided into separate partitions depending on specs to ensure utilization is maximized. You mainly need to choose between partitions depending on how much memory (per CPU) your jobs need, expected job efficiency (i.e. do you expect to keep all CPUs busy at 100%?), and whether you need a GPU. The `default-op` partition is the default partition and is overprovisoned with a factor of 3, meaning each CPU can run up to 3 jobs at once. This is ideal for low efficiency- and interactive jobs, as well as I/O jobs, that wouldn't otherwise fully utilize the CPU's available 100%.

| Partition | Hostname | Threads/Memory | Max. mem per CPU | GPU |
| :--- | :---: | :---: | :---: | :---: |
| `default-op` | `bio-oscloud[01-02]` |  96-192t / 0.5-1.0 TB | 1.5 GB | - |
| `general` | `bio-oscloud[03-06]` |  192t / 1 TB | 5 GB | - |
| `high-mem` | `bio-oscloud[07-08]` | 192-240t / 2 TB | 10 GB | - |
| `gpu` | `bio-oscloud09` | 64t / 256 GB | - | 2x  nvidia A10 |

### Standalone servers
| Hostname | Threads/Memory | Scratch | GPU |
| :--- | :---: | :---: | :---: |
| `axomamma.srv.aau.dk` | 256t / 1 TB | 3.5 TB HDD | - |

## Workstations
These workstations are physically accessible around the labs, but are also available over the network. They have powerful GPU's for enhanced neural network performance and are mainly used for Nanopore basecalling. They are not connected to the storage cluster nor the university Active Directory for user authentication.

| Server name | Threads/Memory | GPU| CUDA cores total |
| :--- | :---: | :---: | :---: | 
| WS1       |  40t / 128 GB | 2x Nvidia RTX2080 Super | 6.144 |
| WS2       |  32t / 96 GB | 1x Nvidia RTX4090 | 16.384 |
| WS5       |  24t / 65 GB | 2x Nvidia RTX4090 | 32.768 |
| GridION   |  16t / 64 GB | 1x Nvidia Quadro GV100 | 5.120 |
