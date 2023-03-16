
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
### Clone VM
```bash
Command Syntax

qm clone <vmid> <newid> [OPTIONS]

qm clone 100 101 --format vmdk -name 100_vmware_export
```
### Start/Stop VM

```bash
Command Syntax 

qm start/stop <vmid> [OPTIONS]

qm start/stop 100
```