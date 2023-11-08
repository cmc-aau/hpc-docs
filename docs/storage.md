# Storage
List directories and where stuff is/should be.

## Best practices
Always try to write fewer but larger files instead of many small ones. Also see [ceph best practices](https://docs.ceph.com/en/reef/cephfs/app-best-practices/).

## Shared folders
**Never** use `chmod 777`. Folders will be scanned regularly for insecure permissions and corrected without notice. Instead add the folder to the `bio_server_users@bio.aau.dk` group and set the setGID bit, for example:
```
folder="some_folder/"

# create the folder
mkdir -p $folder

# set  group ownership
sudo chown -R :bio_server_users@bio.aau.dk "$folder"

# If there's already content within, correct permissions with
find "$folder" -type d -exec chmod 775 {} \;
find "$folder" -type f -exec chmod 664 {} \;

# set the setGID sticky bit to ensure new files and folders inherit group ownership
sudo chmod 2775 "$folder"
```
