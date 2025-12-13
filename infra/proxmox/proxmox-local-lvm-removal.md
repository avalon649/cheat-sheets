### Proxmox local:lvm removal

* Delete local:lvm from Web Gui

`datacenter - storage`

* On Proxmox Terminal

```bash
lvremove /dev/pve/data
```

```bash
lvresize -l +100%FREE /dev/pve/root
```

```bash
resize2fs /dev/mapper/pve-root
```