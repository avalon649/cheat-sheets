### Increase disk size

```bash
Command Syntax

qm resize <vmid> <disk> <size> [OPTIONS]

qm resize 100 virtio0 +5G
```

### Convert Virtual-Box Image to qcow2

```bash
Command Syntax 

qemu-img [standard options] command [command options]

qemu-img convert -f vdi -O qcow2 ubuntu.vdi ubuntu.qcow2
```

### Import an external disk 

```bash
Command Syntax

qm importdisk <vmid> <source> <storage> [OPTIONS]

qm importdisk 105 disk.vmdk local-lvm
```

### Pysical disk passthrough

```bash
ls -l /dev/disk/by-id

qm set  592  -scsi2 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC

update VM 592: -scsi2 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC
```
