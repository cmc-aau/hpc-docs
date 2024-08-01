# Introduction
Welcome to the documentation website for the BioCloud HPC at the section of biotechnology at Aalborg University. Here you will find everything you need to use the cluster for large scale bioinformatic workflows and other things that require more compute resources and storage than normal computers have. The current setup consists of data center servers with dual socket AMD Epyc CPU's of various sizes (from 2022) with a total of **1848 vCPUs** and **12 TB RAM**, all connected to a **2.3 PB+ storage** cluster ([Ceph](https://ceph.com/)).

The cluster is a shared resource for more than a hundred users, so to effectively manage the compute resources and allow many simultaneous users to run jobs concurrently in a completely isolated manner the cluster is set up with the [SLURM workload manager](https://slurm.schedmd.com/archive/slurm-23.02.6/overview.html).

To gain access you must first ask an administrator to allow you to login using your AAU email, and then you will be able to use the cluster resources through either the web portal, or by submitting SLURM batch jobs through a remote shell on a login node.

## Quarterly maintenance
There are pre-scheduled quarterly maintenance days throughout the year on tuesdays in weeks 7, 20, 38, 49, where security upgrades will be performed on all servers, which will be rebooted any time during the day. There will be made a maintenance reservation in SLURM that ensures no jobs will be able to run on these days (and you will therefore see the reason code `(ReqNodeNotAvail, Reserved for maintenance)` if the time limit extends into this reservation), but jobs can still be submitted to the queue. Always stay tuned on the Teams channel for news and announcements.

|  | 2024 | 2025 | 2026 | 2027 | 2028 |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Q1 (week 7) | 19-03-2024 | 11-02-2025 | 10-02-2026 | 09-02-2027 | 08-02-2028 | 
| Q2 (week 20) | 14-05-2024 | 13-05-2025 | 12-05-2026 | 11-05-2027 | 09-05-2028 | 
| Q3 (week 38) | 17-09-2024 | 16-09-2025 | 15-09-2026 | 14-09-2027 | 12-09-2028 | 
| Q4 (week 49) | 03-12-2024 | 02-12-2025 | 01-12-2026 | 30-11-2027 | 28-11-2028 | 
