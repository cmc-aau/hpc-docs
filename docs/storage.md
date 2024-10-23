# Storage
All nodes are connected to a Ceph network storage cluster through a few different mount points for different purposes, listed below. Data is stored with 3x replication to protect against data loss or corruption due to hardware and disk failures. The data is **NOT backed up** anywhere, however, which means it's not protected against human errors. If you delete data, it's gone and cannot be restored! Some mount points are therefore mounted read-only, but your own data is your responsibility alone. So be careful, also when sharing data with other users when doing shared projects etc.

## Network storage mount points

| Mount point | Permissions | Contents |
| --- | --- | --- |
| `/home` | Read/write on your own home folder only | All data for individual projects and generally most things should go here |
| `/projects` | Read/write | All data for shared projects should go here |
| `/databases` | Read-only | Various databases required for bioinformatic tools |
| `/raw_data` | Read-only | Raw data (mostly from DNA sequencing) that should never be touched nor deleted |
| `/incoming` | Read/write | Incoming raw data that should be moved and organized in `/raw_data` as soon as possible |
| `/nanopore` | Read/write | Incoming raw data from nanopore sequencing that should be moved and organized in `/raw_data` as soon as possible. This mount **will be removed** in the near future, please use `/incoming` instead. |

## Local scratch space
For jobs requiring heavy I/O where large amounts of data or thousands of temporary files need to be written, it's important to avoid using the network storage (if possible) and instead write to a local harddrive instead, which both avoids overloading the network and the storage cluster itself, but is also much faster. When deciding whether you need local scratch space, both the size of the data as well the number of files are important, however the ladder is by far the most important because it has the biggest impact on the performance of the storage cluster overall, [more info below](#avoid-small-files-at-all-costs).

Scratch space is not available on all compute nodes (listed under [hardware partition](slurm/partitions.md)), so if you need scratch space you must ensure that you submit jobs to those specific nodes using the `--nodelist` option.

Scratch space is always mounted at `/scratch`. All data owned by your user anywhere under `/scratch` on a particular compute node is **automatically deleted** after the last SLURM job run by your user has finished. Therefore ensure to move files elsewhere as the last step of the job, otherwise it will be lost. When using scratch space please create a separate folder there to work in, preferably just named by your username, so that other users' jobs that may also use the `/scratch` folder are not impacted by your work.

## Storage policy
Our network storage is **NOT** a long-term storage solution. **EVERYTHING IS TEMPORARY, SO CLEAN UP AFTER YOURSELF IMMEDIATELY AFTER EACH JOB!!** You will likely forget about it very quickly after you've done your work, so make it a habit to cleanup shortly after every job. Once studies are published, delete all the data and ensure that for example raw DNA sequencing data is uploaded to public archives and the corresponding code to produce results in the studies is available somewhere, preferably in a GitHub repository.

## Max file size
1TB currently, subject to change, [details here](https://docs.ceph.com/en/latest/cephfs/administration/#maximum-file-sizes-and-performance).

## Moving large amounts of data around
If you need to move large amounts of data (or numerous files at once regardless of size) it will happen in an instant regardless of the size if you make sure you are doing it on the same filesystem/mount, for example from `/projects/xxx` to `/projects/yyy`. However, if you need to move things across mount points, for example from `/home/xxx` to `/projects/yyy` please ask an administrator to do it for you, since moving between mount points will happen over the network and cause unneccessary traffic and load on the storage cluster, let alone be very slow. An administrator can move anything instantly anywhere on the storage cluster, but if you need to transfer external files there is no way around it, it will be limited by the network speed.

## Best practices with Ceph storage
### Avoid small files at all costs
**Whereever possible always try to write fewer but larger files instead of many small ones**. The Ceph storage cluster uses a file metadata cache server that holds an entry for every single open file, so the cache memory usage is directly proportional to the number of open files (meaning being accessed in any way). If there are too many open files (across **ALL** connected clients/nodes at once) the metadata server will ask clients to release some pressure on the cache and if they fail to do so in time they will simply be [evicted](https://docs.ceph.com/en/reef/cephfs/eviction/) without notice (meaning the Ceph cluster will force an unmount). This will happen on any node that tips the iceberg and is thus not a result of the exact storage actions of individual clients but rather that of all nodes connected to the Ceph cluster across the whole university at any one time. See [Ceph best usage practices](https://docs.ceph.com/en/reef/cephfs/app-best-practices/) for additional details.

### Avoid using `ls -l`
When listing directories, it's common to use `ls -l` to list things vertically, however this will also request various other information like permissions, file size, owner, group, access time etc. This will burden the metadata servers, especially if used in loops in scripts on many files, so if you don't need all this extra information and just want to list the contents vertically instead of horizontally, just use `ls -1` instead and make that a habit. Likewise, don't use `stat` on many files if not neccessary.

## Shared folders
If you need to give other users write access to a file/folder that you own, you need to set the group ownership of the folder to the `bio_server_users@bio.aau.dk` group and set the [setGID](https://www.geeksforgeeks.org/setuid-setgid-and-sticky-bits-in-linux-file-permissions/) bit on folders (to ensure child files/folders will inherit the ownership of a parent folder), see the example below. This will give **everyone** with access to the BioCloud servers full control of the files. If you only want a specific group of people to have write access, there is only one way to do that, which is to contact the university IT services to create an email address group for the specific users, and then follow the same steps below, but instead use the new email of that group.

**Never** use `chmod 777`. It's a major security weakness that can allow anyone (even users outside of the university authentication system, human or not) to do anything they want with a file/folder and potentially place hidden payload there. Folders will be scanned regularly for insecure permissions and corrected without notice. Only `owner` and `group` should have the execute bit set, never `other`. See [permissions in Linux](https://www.geeksforgeeks.org/permissions-in-linux/) to learn about file and folder permissions on Linux systems.

```
folder="some_folder/"

# create the folder if it doesn't exist already
mkdir -p "$folder"

# set group ownership
chown -R :bio_server_users@bio.aau.dk "$folder"

# If there are already files with weak permissions, correct them with:
chmod -R o-x "$folder"

# set the setGID sticky bit to ensure new files and folders inherit group ownership
chmod 2775 "$folder"
```
