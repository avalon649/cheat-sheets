
### Create VM
```bash
Command Syntax

qm create <vmid> [OPTIONS]

qm create 117 --memory 2048 --name test --net0 virtio,bridge=vmbr0
```

### Destroy VM
```bash
Command Syntax 

qm destroy <vmid> [OPTIONS]

qm destroy vmid
```

### Attach hardisk to VM
```bash
Command Syntax

qm set <vmid> [OPTIONS]

qm set 200 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-200-disk-1
```