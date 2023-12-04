# Usage accounting
All users belong to an account (usually their PI) and all usage is tracked per user, but limitations can be set by administrators at a few different levels: at the cluster, partition, account, user, or QOS level. User associations with accounts rarely change, so in order to be able to temporarily request additional ressources or obtain higher priority for certain projects, users can submit to different SLURM "Quality of Service"s (QOS). By default all users submit jobs to the `normal` QOS. To submit to a different QOS to obtain a higher priority first discuss with your supervisor/PI, and then contact an administrator to get permission.

## Show QOS info and limitations
```
$ sacctmgr show qos format=name,priority,grptres,maxtres,maxtrespu,maxjobspu,maxtrespa,maxjobspa
      Name   Priority       GrpTRES       MaxTRES     MaxTRESPU MaxJobsPU     MaxTRESPA MaxJobsPA 
---------- ---------- ------------- ------------- ------------- --------- ------------- --------- 
    normal          0                                   cpu=576                                   
  highprio          1                                  cpu=1024                                   
```

## Undergraduate students
Undergraduate students (upto but NOT including master projects) share ressources within the `students` account and only their combined usage is limited. View current limitations with for example:

```
$ sacctmgr list account students -s | head -n 3
```

## Job usage summary
To view the status of **past** jobs and their usage accounting information use [`sacct`](https://slurm.schedmd.com/sacct.html). `sacct` will return **everything** accounted for by default which is very inconvenient to view in a terminal window, so below are only the most essential columns shown:
```
$ sacct -o jobid,jobname,start,end,NNodes,NCPUS,ReqMem,CPUTime,AveRSS,MaxRSS --user=$USER --units=G -j 138
JobID           JobName               Start                 End   NNodes      NCPUS     ReqMem    CPUTime     AveRSS     MaxRSS 
------------ ---------- ------------------- ------------------- -------- ---------- ---------- ---------- ---------- ---------- 
138          interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16        20G   00:02:56                       
138.interac+ interacti+ 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56          0          0 
138.extern       extern 2023-11-21T10:43:48 2023-11-21T10:43:59        1         16              00:02:56      0.00G      0.00G 

```

There are a huge number of other options to show, see [SLURM docs](https://slurm.schedmd.com/sacct.html#SECTION_Job-Accounting-Fields). If you really want to see everything use `sacct --long > file.txt` and dump it into a file or else it's too much for the terminal.

## Usage reports
Account usage:
```
$ sreport cluster AccountUtilizationByUser
```

User usage, listed by account
```
$ sreport cluster UserUtilizationByAccount
```

Top usage by user the last week, in CPU hours, not CPU minutes (default)
```
$ sreport user top start=now-1week -t hour
```
