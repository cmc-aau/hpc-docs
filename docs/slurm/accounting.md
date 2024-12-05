# Usage accounting and priority
All users belong to an account (usually their PI) where all usage is tracked on a per-user basis, but limitations and priorities can be set at a few different levels: at the cluster, partition, account, user, or QOS level. User associations with accounts rarely change, so in order to be able to temporarily request additional resources or obtain higher priority for certain projects, users can submit to different SLURM "Quality of Service"s (QOS). By default, all users can only submit jobs to the `normal` QOS with equal resource limits and base priority for everyone. Periodically users may submit to the `highprio` QOS instead, which has extra resources and higher priority (and therefore the usage is also billed 2x), however this must first be discussed among the owners of the hardware (PI's), and then you must contact an administrator to grant your user permission to submit jobs to it.

## Job priority
When a job is submitted a priority value is calculated based on several factors, where a higher number indicates a higher priority in the queue. This does not impact running jobs, and the effect of prioritization is only noticable when the cluster is operating near peak capacity, or when the hardware partition to which the job has been submitted is nearly fully allocated. Otherwise jobs will usually start immediately as long as there are resources available and you haven't reached the maximum CPU's per user limit, if any.

Different weights are given to different priority factors, where the most significant ones are the account fair-share factor (which is simply the number of CPU's each account (PI) has contributed with to the cluster), the user's recent resource usage (higher usage results in a lower priority), and the QOS, as described above. All factors are normalized to a value between 0-1, then weighted by an adjustable scalar, which may be adjusted occasionally depending on the overall cluster usage. Users can also be nice to other users and reduce the priority of their own jobs by setting a "nice" value using `--nice` when submitting for example less time-critical jobs. Job priorities are then calculated according to the following formula:

```
Job_priority =
	(PriorityWeightQOS) * (QOS_factor)
	(PriorityWeightAssoc) * (assoc_factor) +
	(PriorityWeightAge) * (age_factor) +
	(PriorityWeightFairshare) * (fair-share_factor) +
	(PriorityWeightJobSize) * (job_size_factor) -
	- nice_factor
```

To obtain up-to-date weights use `sprio -w`:
```
$ sprio -w
          JOBID PARTITION   PRIORITY       SITE        AGE  FAIRSHARE    JOBSIZE        QOS
        Weights                               1          5         10          3         10
```

The priority of pending jobs can be obtained using `sprio`, but is also shown when running `squeue`.

The fair-share factor is calculated according to the [fair-tree algorithm](https://slurm.schedmd.com/archive/slurm-23.02.6/fair_tree.html) and has a usage decay half-life of 2 weeks, but is completely reset at the first day of each month. To see your current fair-share factor run `sshare -U`.

???+ tip "The fair-share factor and CPU efficiency"
      The value of the fair-share factor is calculated based on CPU usage in units of **allocation seconds** and not CPU seconds, which is normally the unit used for CPU usage reported by the `sreport` and `sacct` commands. Therefore, this also means that the CPU efficiency of past jobs directly impacts how much actual work can be performed by the allocated CPUs for each user within each account before their fair share of resources is consumed for the period.

The age factor will max out to `1.0` when 3 days of queue time has been accrued for any job.

For more details about job prioritization see the [SLURM documentation](https://slurm.schedmd.com/archive/slurm-23.02.6/priority_multifactor.html) and this [presentation](https://slurm.schedmd.com/SLUG19/Priority_and_Fair_Trees.pdf).

## QOS info and limitations
See all available QOS and their limitations:
```
$ sacctmgr show qos format=name,priority,grptres,mintres,maxtres,maxtrespu,maxjobspu,maxtrespa,maxjobspa
      Name   Priority       GrpTRES       MinTRES       MaxTRES     MaxTRESPU MaxJobsPU     MaxTRESPA MaxJobsPA 
---------- ---------- ------------- ------------- ------------- ------------- --------- ------------- --------- 
    normal          1                cpu=1,mem=512M                    cpu=192                                   
  highprio          10                cpu=1,mem=512M                   cpu=512                                   
                                 
```

See all account associations for your user and the QOS's you are allowed to use:
```
$ sacctmgr list association user=$USER format=account%10s,user%20s,qos%20s
   Account                 User                  QOS 
---------- -------------------- -------------------- 
      root       ksa@bio.aau.dk      highprio,normal
```

### Undergraduate students
Undergraduate students (up to but NOT including master projects) share resources within the `students` account and only their combined usage is limited. View current limitations with for example:

```
$ sacctmgr list account students -s format="account,priority,MaxCPUs,grpcpus,qos" | head -n 3
   Account   Priority  MaxCPUs  GrpCPUs                  QOS 
---------- ---------- -------- -------- -------------------- 
  students                          192               normal
```

## Job resource usage summary
### Running jobs
Use [`sstat`](https://slurm.schedmd.com/archive/slurm-23.02.6/sstat.html) to show the status and live usage accounting information of only **running** jobs. For batch scripts you need to add `.batch` to the job ID, for example:
```
$ sstat <job_id>.batch
```

This will print EVERY metric, so it's nice to select only a few most relevant ones, for example:

```
$ sstat --jobs <job_id>.batch --format=jobid,avecpu,maxrss,ntasks
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

### Past jobs
To view the status of **past** jobs and their usage accounting information use [`sacct`](https://slurm.schedmd.com/archive/slurm-23.02.6/sacct.html). `sacct` will return **everything** accounted for by default which is very inconvenient to view in a terminal window, so the below command will show the most essential information:
```
$ sacct -o jobid,jobname,start,end,NNodes,NCPUS,ReqMem,CPUTime,AveRSS,MaxRSS --user=$USER --units=G -j 138
JobID           JobName               Start                 End   NNodes      NCPUS     ReqMem    CPUTime     AveRSS     MaxRSS 
------------ ---------- ------------------- ------------------- -------- ---------- ---------- ---------- ---------- ---------- 
138          interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16        20G   00:02:56                       
138.interac+ interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56          0          0 
138.extern       extern 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56      0.00G      0.00G 

```

There are a huge number of other options to show, see [SLURM docs](https://slurm.schedmd.com/archive/slurm-23.02.6/sacct.html#SECTION_Job-Accounting-Fields). If you really want to see everything use `sacct --long > file.txt` and dump it into a file or else it's too much for the terminal.

## Reservation usage
Show reservation usage in CPU hours and percent of the reservation total used
```
$ sreport reservation utilization -t hourper
--------------------------------------------------------------------------------
Reservation Utilization 2024-11-04T00:00:00 - 2024-11-04T23:59:59
Usage reported in TRES Hours/Percentage of Total
--------------------------------------------------------------------------------
  Cluster      Name               Start                 End      TRES Name                     Allocated                          Idle 
--------- --------- ------------------- ------------------- -------------- ----------------------------- ----------------------------- 
 biocloud  amplicon 2024-11-04T08:00:00 2024-11-05T15:18:55            cpu                  1154(19.20%)                  4858(80.80%) 
```

## Job efficiency summary
### Individual jobs
To view the efficiencies of individual jobs use `seff`, for example:

```
$ seff 2357
Job ID: 2357
Cluster: biocloud
User/Group: <username>
State: COMPLETED (exit code 0)
Nodes: 1
Cores per node: 96
CPU Utilized: 60-11:06:29
CPU Efficiency: 45.76% of 132-02:48:00 core-walltime
Job Wall-clock time: 1-09:01:45
Memory Utilized: 383.42 GB
Memory Efficiency: 45.11% of 850.00 GB
```

This information will also be shown in notification emails when jobs finish.

### Multiple jobs
Perhaps a more useful way to use `sacct` is through the [reportseff](https://github.com/troycomi/reportseff) tool (pre-installed), which can be used to calculate the CPU, memory, and time efficiencies of past jobs, so that you can optimize future jobs and ensure resources are utilized to the max (**2TM**). For example:
```
$ reportseff -u $(whoami) --format partition,jobid,state,jobname,alloccpus,elapsed,cputime,CPUEff,MemEff,TimeEff -S r,pd,s --since d=4
  Partition     JobID        State                       JobName                    AllocCPUS     Elapsed        CPUTime      CPUEff   MemEff   TimeEff 
  shared    306282     COMPLETED                     midasok                        2         00:13:07       00:26:14      4.1%    15.4%     10.9%  
   general      306290     COMPLETED           smk-map2db-sample=barcode46             16         00:01:38       00:26:08     90.6%    18.7%     2.7%   
   general      306291     COMPLETED           smk-map2db-sample=barcode47             16         00:02:14       00:35:44     92.8%    18.4%     3.7%   
   general      306292     COMPLETED           smk-map2db-sample=barcode58             16         00:02:32       00:40:32     80.3%    19.0%     4.2%   
   general      306293     COMPLETED           smk-map2db-sample=barcode34             16         00:02:16       00:36:16     78.1%    18.7%     3.8%   
   general      306294     COMPLETED           smk-map2db-sample=barcode22             16         00:02:38       00:42:08     81.1%    19.0%     4.4%   
   general      306295     COMPLETED           smk-map2db-sample=barcode35             16         00:02:26       00:38:56     79.0%    19.0%     4.1%   
   general      306296     COMPLETED           smk-map2db-sample=barcode82             16         00:01:39       00:26:24     68.9%    17.8%     2.8%   
   general      306297     COMPLETED           smk-map2db-sample=barcode70             16         00:02:04       00:33:04     76.0%    19.5%     3.4%   
   general      306298     COMPLETED           smk-map2db-sample=barcode94             16         00:01:44       00:27:44     70.1%    17.9%     2.9%   
   general      306331     COMPLETED           smk-map2db-sample=barcode59             16         00:04:00       01:04:00     87.2%    19.6%     6.7%   
   general      306332     COMPLETED           smk-map2db-sample=barcode83             16         00:02:13       00:35:28     76.3%    19.0%     3.7%   
   general      306333     COMPLETED           smk-map2db-sample=barcode11             16         00:02:49       00:45:04     81.6%    19.4%     4.7%   
   general      306334     COMPLETED           smk-map2db-sample=barcode23             16         00:03:33       00:56:48     61.6%    19.0%     5.9%   
   general      306598     COMPLETED      smk-mapping_overview-sample=barcode46         1         00:00:09       00:00:09     77.8%     0.0%     1.5%   
   general      306601     COMPLETED      smk-mapping_overview-sample=barcode82         1         00:00:09       00:00:09     77.8%     0.0%     1.5%   
   general      306625     COMPLETED      smk-mapping_overview-sample=barcode94         1         00:00:07       00:00:07     71.4%     0.0%     1.2%   
   general      306628     COMPLETED      smk-concatenate_fastq-sample=barcode71        1         00:00:07       00:00:07     71.4%     0.0%     1.2%   
   general      306629     COMPLETED      smk-concatenate_fastq-sample=barcode70        1         00:00:07       00:00:07     71.4%     0.0%     1.2%   
   general      306630     COMPLETED      smk-concatenate_fastq-sample=barcode34        1         00:00:07       00:00:07     57.1%     0.0%     1.2%   
```

## Usage reports
`sreport` can be used to summarize usage in many different ways, below are some examples.

### Account usage by user
```
$ sreport cluster AccountUtilizationByUser format=account%8,login%23,used%10 -t hourper start="now-1week"
--------------------------------------------------------------------------------
Cluster/Account/User Utilization 2024-03-13T12:00:00 - 2024-03-19T23:59:59 (561600 secs)
Usage reported in CPU Hours/Percentage of Total
--------------------------------------------------------------------------------
 Account                   Login               Used 
-------- ----------------------- ------------------ 
    root                             154251(63.71%) 
    root              <username>      37176(15.35%) 
     jln                                3988(1.65%) 
     jln              <username>        2010(0.83%) 
     jln              <username>           0(0.00%) 
     jln              <username>        1978(0.82%) 
     kln                                   7(0.00%) 
     kln              <username>           7(0.00%) 
      ma                               13460(5.56%) 
      ma              <username>        6152(2.54%) 
      ma              <username>        2504(1.03%) 
      ma              <username>        3710(1.53%) 
      ma              <username>         963(0.40%) 
      ma              <username>         131(0.05%) 
      md                              52921(21.86%) 
      md              <username>       18690(7.72%) 
      md              <username>       22443(9.27%) 
      md              <username>       11788(4.87%) 
     mms                               17036(7.04%) 
     mms              <username>         600(0.25%) 
     mms              <username>       16436(6.79%) 
     phn                                6799(2.81%) 
     phn              <username>         114(0.05%) 
     phn              <username>          31(0.01%) 
     phn              <username>        6654(2.75%) 
     phn              <username>           0(0.00%) 
     sss                               22081(9.12%) 
     sss              <username>       22081(9.12%) 
students                                 638(0.26%) 
students              <username>         638(0.26%) 
```

### User usage by account
```
$ sreport cluster UserUtilizationByAccount format=login%23,account%8,used%10 -t hourper start="now-1week"
--------------------------------------------------------------------------------
Cluster/User/Account Utilization 2024-03-13T12:00:00 - 2024-03-19T23:59:59 (561600 secs)
Usage reported in CPU Hours/Percentage of Total
--------------------------------------------------------------------------------
                  Login  Account               Used 
----------------------- -------- ------------------ 
             <username>     root      37176(15.35%) 
             <username>       md       22443(9.27%) 
             <username>      sss       22081(9.12%) 
             <username>       md       18690(7.72%) 
             <username>      mms       16436(6.79%) 
             <username>       md       11788(4.87%) 
             <username>      phn        6654(2.75%) 
             <username>       ma        6152(2.54%) 
             <username>       ma        3710(1.53%) 
             <username>       ma        2504(1.03%) 
             <username>      jln        2010(0.83%) 
             <username>      jln        1978(0.82%) 
             <username>       ma         963(0.40%) 
             <username> students         638(0.26%) 
             <username>      mms         600(0.25%) 
             <username> compute+         145(0.06%) 
             <username>       ma         131(0.05%) 
             <username>      phn         114(0.05%) 
             <username>      phn          31(0.01%) 
             <username>      kln           7(0.00%) 
             <username>      phn           0(0.00%) 
             <username>      jln           0(0.00%) 
```

### Top users
```
$ sreport user top topcount=20 format=login%21,account%8,used%10 start="now-1week" -t hourper
--------------------------------------------------------------------------------
Top 20 Users 2024-03-13T11:00:00 - 2024-03-19T23:59:59 (565200 secs)
Usage reported in CPU Hours/Percentage of Total
--------------------------------------------------------------------------------
                Login  Account               Used 
--------------------- -------- ------------------ 
          <username>      root      37176(15.26%) 
          <username>        md       22698(9.32%) 
          <username>       sss       22082(9.06%) 
          <username>        md       18850(7.74%) 
          <username>       mms       16436(6.75%) 
          <username>        md       12038(4.94%) 
          <username>       phn        6654(2.73%) 
          <username>        ma        6167(2.53%) 
          <username>        ma        3780(1.55%) 
          <username>        ma        2504(1.03%) 
          <username>       jln        2383(0.98%) 
          <username>       jln        1978(0.81%) 
          <username>        ma         963(0.40%) 
          <username>  students         648(0.27%) 
          <username>       mms         600(0.25%) 
          <username>        kt         146(0.06%) 
          <username>        ma         133(0.05%) 
          <username>       phn         114(0.05%) 
          <username>       kln          98(0.04%) 
          <username>       phn          31(0.01%) 
```
