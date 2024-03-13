# Quarterly maintenance
There are scheduled quarterly maintenance days throughout the year on tuesdays in weeks 7, 20, 38, 49, where ITS, Netic, and the CLAAUDIA team will perform security upgrades on the OpenStack platform. The OpenStack hypervisors which host the virtual machines will be rebooted, and so will all VM's in the BioCloud. There have already been made maintenance reservations in the SLURM scheduler that ensure no jobs will be able to run these days (and you will therefore see the reason code `(ReqNodeNotAvail, Reserved for maintenance)` if the time limit extends into this reservation), but jobs can still be submitted to the queue. Stay tuned on the Teams channel.

|  | 2024 | 2025 | 2026 | 2027 | 2028 |
| :--- | :---: | :---: | :---: | :---: | :---: |
| Q1 (week 7) | 19-03-2024 | 11-02-2025 | 10-02-2026 | 09-02-2027 | 08-02-2028 | 
| Q2 (week 20) | 14-05-2024 | 13-05-2025 | 12-05-2026 | 11-05-2027 | 09-05-2028 | 
| Q3 (week 38) | 17-09-2024 | 16-09-2025 | 15-09-2026 | 14-09-2027 | 12-09-2028 | 
| Q4 (week 49) | 03-12-2024 | 02-12-2025 | 01-12-2026 | 30-11-2027 | 28-11-2028 | 
