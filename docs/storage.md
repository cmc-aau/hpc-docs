# Storage
All nodes are connected to a Ceph network storage cluster through a few different mount points for different purposes, listed below. Data is stored with 3x replication to protect against data loss or corruption due to hardware and disk failures. The data is **NOT** backed up anywhere, however, which means it's **NOT** protected against human errors. If you delete data, it's gone and cannot be restored! Some mount points are therefore mounted read-only, but your own data is your responsibility alone. So be careful, also when sharing data with other users when doing shared projects etc.

| Mount point | Permissions | Contents |
| --- | --- | --- |
| `/home` | Read/write on your own home folder only, the rest is read-only (by default) | All data for individual projects and generally most things should go here |
| `/projects` | Read/write on everything (by default) | All data for shared projects should go here |
| `/databases` | Mounted read-only | Various databases required for bioinformatic tools |
| `/raw_data` | Mounted read-only | Raw data (mostly from DNA sequencing) that should never be touched nor deleted |
| `/incoming` | Read/write (by default) | Incoming raw data that should be moved and organized in `/raw_data` as soon as possible |
| `/nanopore` | Read/write (by default) | Incoming raw data from nanopore sequencing that should be moved and organized in `/raw_data` as soon as possible. This mount **will be removed** in the future, please use `/incoming` instead. |
| `/user_data` | Read-only | This is an old location for user data. This mount **will be removed** in the future, please use your home folder instead. |
| | | |

## Moving large amounts of data around
If you need to move large amounts of data (or numerous files at once regardless of size) it will happen in an instant regardless of the size if you make sure you are doing it on the same filesystem/mount, for example from `/projects/xxx` to `/projects/yyy`. However, if you need to move things across mount points, for example from `/home/xxx` to `/projects/yyy` please ask an administrator to do it for you, since moving between mount points will happen over the network and cause unneccessary traffic and load on the storage cluster, let alone be very slow. An administrator can move anything instantly anywhere on the storage cluster, but if you need to transfer external files there is no way around it, it will be limited by the network speed.

## Best practices with Ceph storage
**Whereever possible always try to write fewer but larger files instead of many small ones**. The Ceph storage cluster uses a file metadata cache server that holds an entry for every single open file, so the cache memory usage is directly proportional to the number of open files. If there are too many open files (across **ALL** connected clients/nodes at once) the metadata server will ask clients to release some pressure on the cache and if they fail to do so in time they will simply be [evicted](https://docs.ceph.com/en/reef/cephfs/eviction/) without notice (meaning the Ceph cluster will force an unmount). This will happen on any node that tips the iceberg and is thus not a result of the exact storage actions of individual clients but rather that of all nodes connected to the Ceph cluster across the whole university at any one time. See [Ceph best usage practices](https://docs.ceph.com/en/reef/cephfs/app-best-practices/) for additional details.

## Shared folders
If you need to enable other users to access a file/folder you need to set the group ownership of the folder to the `bio_server_users@bio.aau.dk` group and set the [setGID](https://www.geeksforgeeks.org/setuid-setgid-and-sticky-bits-in-linux-file-permissions/) bit on folders (to ensure child files/folders will inherit the ownership of a parent folder). Example below.

**Never** use `chmod 777`. It's a major security weakness that can allow anyone (even users outside of the university authentication system, human or not) to do anything they want with a file/folder and potentially place hidden payload there. Folders will be scanned regularly for insecure permissions and corrected without notice. 

```
folder="some_folder/"

# create the folder
mkdir -p $folder

# set  group ownership
chown -R :bio_server_users@bio.aau.dk "$folder"

# If there are already files with weak permissions, correct them with for example:
find "$folder" -type d -exec chmod 775 {} \;
find "$folder" -type f -exec chmod 664 {} \;

# set the setGID sticky bit to ensure new files and folders inherit group ownership
chmod 2775 "$folder"
```

This can only be done on folders that you own, so ask other users or an administrator if you need to change ownership on an existing folder that you don't own.
