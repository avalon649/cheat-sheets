# Proxmox Cheat-Sheets
## Resize Disk
### Increase disk size
Increase disk size in the GUI or with the following command
```
qm resize 100 virtio0 +5G
```

### Decrease disk size
> Before decreasing disk sizes in Proxmox, you should take a backup!
1. Convert qcow2 to raw
```
qemu-img convert vm-100.qcow2 vm-100.raw
```
2. Shrink the disk
```
qemu-img resize -f raw vm-100.raw 10G
```
3. Convert back to qcow2#
```
qemu-img convert -p -O qcow2 vm-100.raw vm-100.qcow2
```
### Import an external disk 

```
qm importdisk 105 disk.vmdk local-lvm
```
### Pysical disk passthrough

```
ls -l /dev/disk/by-id

qm set  592  -scsi2 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC

update VM 592: -scsi2 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC
```

### Create VM

```bash
qm create 117 --memory 2048 --name test --net0 virtio,bridge=vmbr0
```

### Destroy VM

```bash
qm destroy vmid
```

### Attach hardisk to VM

```bash
qm set 200 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-200-disk-1
```