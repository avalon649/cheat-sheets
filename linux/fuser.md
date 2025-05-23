### Fuser Basic Troubleshooting Commands

#### Checks Which Processes is Using The Mount Point

```bash
fuser -m /mnt/nfs_mount_point
```

#### To Forcefully Kill The Processes 

```bash
fuser -km /mnt/nfs_mount_point
```
