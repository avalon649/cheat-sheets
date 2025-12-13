### 1. To prepare the disks for LVM creation, first wipe them.

```bash
wipefs -a /dev/sdb /dev/sdc 
```

### 2. Create physical volumes on the disks using pvcreate. Physical volumes are the basic blocks of LVM storage.

```bash
pvcreate /dev/sdb /dev/sdc 
```
 
### 3. Next create a volume group with all the disks. Volume group will combine all physical volumes to give one central storage structure.

```bash
vgcreate vgrp /dev/sdb /dev/sdc
#vgrp is the name of the vol group 
```

### 4. Now we are ready to create the logical volume on the volume group. I am creating one with 18.00 TB space.

```bash
lvcreate -L 18.0T -n lvol vgrp
#lvol is the name for the logical volume 
```

### 5. On the logical volume create a File System for usage. I am creating an ext4 file system on it.

```bash
mkfs.ext4 /dev/vgrp/lvol 
```

### 6. File system is set up, so we can mount it under a directory.

#Create a mount point

```bash
mkdir -p /mnt/lvol
```
# Mount the volume

```bash
mount /dev/vgrp/lvol /mnt/lvol
```

# Add it to /etc/fstab for automount:

```bash
echo "/dev/vgrp/lvol /mnt/lvol ext4 defaults 0 2" >> /etc/fstab 
```