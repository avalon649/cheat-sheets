
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