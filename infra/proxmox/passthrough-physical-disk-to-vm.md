- Attach Pass Through Disk 
- Identify Disk 

Before adding a physical disk to host make note of vendor, serial  so that you'll know which disk to share in /dev/disk/by-id/ 

```bash
ls -l /dev/disk/by-id | grep Z1F41BLC
```

- List disk by-id with lsblk 

The `lsblk` is pre-installed, you can print and map the serial and WWN identifiers of attached disks using the following two commands:

```bash
 lsblk -o +MODEL,SERIAL,WWN
 ls -l /dev/disk/by-id/
```


- Update Configuration 
- Hot-Plug/Add physical device as new virtual SCSI disk 

```bash
qm set 592 -scsi2 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC

update VM 592: -scsi2 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC
```

- Hot-Unplug/Remove virtual disk 

```bash
qm unlink 592 --idlist scsi2

update VM 592: -delete scsi2
```

- Check Configuration File 

```bash
grep Z1F41BLC /etc/pve/qemu-server/592.conf

scsi2: /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC,size=2930266584K
```

- Stop and Restart KVM Virtual Machine 
