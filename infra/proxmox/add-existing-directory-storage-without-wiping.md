### Add existing directory storage to proxmox without wiping it first

So first, I started with getting idea of what the current disk setup looks like with `lsblk.`

```
> lsblk
NAME               MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                  8:0    0 931.5G  0 disk
├─sda1               8:1    0  1007K  0 part
├─sda2               8:2    0   512M  0 part
└─sda3               8:3    0   931G  0 part
  ├─pve-swap       253:0    0     8G  0 lvm  [SWAP]
  ├─pve-root       253:1    0    96G  0 lvm  /
  ├─pve-data_tmeta 253:2    0   8.1G  0 lvm
  │ └─pve-data     253:4    0 794.8G  0 lvm
  └─pve-data_tdata 253:3    0 794.8G  0 lvm
    └─pve-data     253:4    0 794.8G  0 lvm
sdb                  8:16   0 465.8G  0 disk
└─sdb1               8:17   0 465.8G  0 part
nvme0n1            259:0    0 476.9G  0 disk
└─nvme0n1p1        259:1    0 476.9G  0 part
```

Then, I got more details, taking particular note of the UUID of each disk.

```
root@proxmox:~# lsblk -fs
NAME             FSTYPE      FSVER    LABEL UUID                                   FSAVAIL FSUSE% MOUNTPOINT
sda1
└─sda
sda2             vfat        FAT32          8EB3-321A
└─sda
sdb1             ext4        1.0            4ebde4e5-f56e-4bbb-bf16-a803efd5ae60    282.8G    33% 
└─sdb
pve-swap         swap        1              53bd881c-6778-496e-939f-76f716311840                  [SWAP]
└─sda3           LVM2_member LVM2 001       0s1SbD-EreR-DwIQ-WcdO-ywLD-e5DF-3rAtrM
  └─sda
pve-root         ext4        1.0            1c91f72d-1721-49d3-bbf7-76681f5aec4c     86.4G     3% /
└─sda3           LVM2_member LVM2 001       0s1SbD-EreR-DwIQ-WcdO-ywLD-e5DF-3rAtrM
  └─sda
pve-data
├─pve-data_tmeta
│ └─sda3         LVM2_member LVM2 001       0s1SbD-EreR-DwIQ-WcdO-ywLD-e5DF-3rAtrM
│   └─sda
└─pve-data_tdata
  └─sda3         LVM2_member LVM2 001       0s1SbD-EreR-DwIQ-WcdO-ywLD-e5DF-3rAtrM
    └─sda
nvme0n1p1        ext4        1.0            4106806e-0ec1-49a4-85e8-dc2337a04086    127.9G    68% 
└─nvme0n1
```


Now that I've got the UUID I need, I created a mount file in ```/etc/systemd/system```

```
Mount up!

root@proxmox:~# cat /etc/systemd/system/mnt-pve-ssd.mount
[Install]
WantedBy=multi-user.target

[Mount]
Options=defaults
Type=ext4
What=/dev/disk/by-uuid/4ebde4e5-f56e-4bbb-bf16-a803efd5ae60
Where=/mnt/pve/ssd

[Unit]
Description=Mount storage 'ssd' under /mnt/pve
```

After creating the configs for both mnt-pve-ssd.mount and mnt-pve-m2.mount, I checked to see if they were enabled.

```
root@proxmox:~# systemctl list-unit-files -t mount

UNIT FILE                     STATE     VENDOR PRESET
-.mount                       generated -
dev-hugepages.mount           static    -
dev-mqueue.mount              static    -
mnt-pve-m2.mount              disabled  disabled
mnt-pve-ssd.mount             disabled  disabled
proc-fs-nfsd.mount            static    -
proc-sys-fs-binfmt_misc.mount disabled  disabled
run-rpc_pipefs.mount          static    -
sys-fs-fuse-connections.mount static    -
sys-kernel-config.mount       static    -
sys-kernel-debug.mount        static    -
sys-kernel-tracing.mount      static    -

12 unit files listed.
```

Not yet! So I enabled both of my disks

```
root@proxmox:~# systemctl enable mnt-pve-m2.mount
Created symlink /etc/systemd/system/multi-user.target.wants/mnt-pve-m2.mount → /etc/systemd/system/mnt-pve-m2.mount.
root@proxmox:~# systemctl enable mnt-pve-ssd.mount
Created symlink /etc/systemd/system/multi-user.target.wants/mnt-pve-ssd.mount → /etc/systemd/system/mnt-pve-ssd.mount.
```

Then I checked to see they were enabled.

```
root@proxmox:~# systemctl list-unit-files -t mount

UNIT FILE                     STATE     VENDOR PRESET
-.mount                       generated -
dev-hugepages.mount           static    -
dev-mqueue.mount              static    -
mnt-pve-m2.mount              enabled   enabled
mnt-pve-ssd.mount             enabled   enabled
proc-fs-nfsd.mount            static    -
proc-sys-fs-binfmt_misc.mount disabled  disabled
run-rpc_pipefs.mount          static    -
sys-fs-fuse-connections.mount static    -
sys-kernel-config.mount       static    -
sys-kernel-debug.mount        static    -
sys-kernel-tracing.mount      static    -

12 unit files listed.
```

Lookin good. I rebooted, then checked out /etc/mtab and there they were.

```
/dev/nvme0n1p1 /mnt/pve/m2 ext4 rw,relatime 0 0
/dev/sdb1 /mnt/pve/ssd ext4 rw,relatime 0 0
```

Then I checked again with lsblk and there were my mounted disk, nice!

```
root@proxmox:~# lsblk
NAME               MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                  8:0    0 931.5G  0 disk
├─sda1               8:1    0  1007K  0 part
├─sda2               8:2    0   512M  0 part
└─sda3               8:3    0   931G  0 part
  ├─pve-swap       253:0    0     8G  0 lvm  [SWAP]
  ├─pve-root       253:1    0    96G  0 lvm  /
  ├─pve-data_tmeta 253:2    0   8.1G  0 lvm
  │ └─pve-data     253:4    0 794.8G  0 lvm
  └─pve-data_tdata 253:3    0 794.8G  0 lvm
    └─pve-data     253:4    0 794.8G  0 lvm
sdb                  8:16   0 465.8G  0 disk
└─sdb1               8:17   0 465.8G  0 part /mnt/pve/ssd
nvme0n1            259:0    0 476.9G  0 disk
└─nvme0n1p1        259:1    0 476.9G  0 part /mnt/pve/m2
```

One big caveat I wanted to mention -- if you back up your /etc/pve directory, make sure the UUID in your backup files match your current setup. Took me 3 days of battling Proxmox to realize that.