# Hardware overview
Welcome to the documentation website for the BioCloud HPC at the section of biotechnology at Aalborg University. Here you will (hopefully) find everything you need to use the cluster for large scale bioinformatic workflows and other things that require more compute ressources and storage.

The current setup consists of data center servers with dual socket AMD Epyc CPU's of various sizes (from 2022) with a total of **1752 vCPUs** and **11.2 TB RAM**, all connected to a **2.3 PB+ storage** cluster ([Ceph](https://ceph.com/)). Most servers/VMs in the cluster are set up with the [SLURM workload manager](https://slurm.schedmd.com/overview.html) to effectively manage the compute ressources for more than a hundred users.

## Cloud compute
Optimized for heavy parallel CPU workloads that may require lots of memory and storage, but one server also has 2x NVIDIA A10 GPU's for smaller jobs that benefit from GPU acceleration. A few servers have local scratch storage for faster I/O. The servers are located in an enterprise data center at AAU with 25 gbps network, and they are only accessible remotely from the local AAU network or VPN. The `axomamma.srv.aau.dk` server is the only bare-metal machine, it's not managed by Slurm as it's reserved for GUI apps like CLC and Arb and for students. The rest are virtual machines (VM's) in a 1:1 physical to virtual CPU configuration, and are set up identically. All VM's run Ubuntu 22.04 LTS, while `axomamma.srv.aau.dk` runs Ubuntu 20.04 LTS. 

| Node type | Threads/Memory | Scratch | GPU | Slurm partition |
| --- | --- | --- | --- | --- |
| General compute nodes (x5) |  192t / 1 TB | - | - | biocloud-cpu |
| Fat node (x1) | 192t / 2 TB | - | - | biocloud-cpu |
| Extra fat node (x1) | 256t / 2 TB | 15 TB NVMe | - | biocloud-cpu |
| GPU node (x1) | 64t / 256 GB | 3 TB NVMe | 2x  nvidia A10 | biocloud-gpu |
| axomamma.srv.aau.dk | 256t / 1 TB | 3.5 TB HDD | - | - |

## Workstations
These workstations are physically accessible around the labs, but are also available over the network. They have powerful GPU's for enhanced neural network performance and are mainly used for Nanopore basecalling. They are not connected to the storage cluster nor the university Active Directory for user authentication.

| Server name | IP   | Threads/Memory | GPU| CUDA cores total |
| ---         | ---  | --- | --- |--- | 
| WS1         | 172.25.230.247 | 40t / 128 GB | 2x Nvidia RTX2080 Super | 6.144 |
| WS2         | 172.25.230.249 | 32t / 96 GB | 1x Nvidia RTX4090 | 16.384 |
| WS5         | 172.25.230.248  | 24t / 65 GB | 2x Nvidia RTX4090 | 32.768 |
| P24-A100    | 172.25.230.250  | 160t / 512 GB | 4x Nvidia A100 | 27.648 | 
| GridION     | 172.25.230.240 | 16t / 64 GB | 1x Nvidia Quadro GV100 | 5.120 |
