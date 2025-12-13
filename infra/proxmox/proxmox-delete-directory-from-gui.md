### Delete Directoy From Gui

#### From Proxmox Terminal

```bash
systemctl disable mnt-pve-testdir.mount
umount /mnt/pve/testdir
```

```bash
nano /proc/mounts
```

```bash
rm /etc/systemd/system/mnt-pve-testdir.mount
```

```reboot```