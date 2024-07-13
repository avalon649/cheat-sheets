## Instructions

### Choose your Ubuntu Cloud Image

Download Ubuntu (replace with the url of the one you chose from above)
```bash
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
```
1. Create a new virtual machine

```bash
qm create 8000 --memory 2048 --core 2 --name ubuntu-cloud --net0 virtio,bridge=vmbr0
```

2. Import the downloaded Ubuntu disk to local-lvm storage

```bash
qm importdisk 8000 focal-server-cloudimg-amd64.img local
```

3. Attach the new disk to the vm as a scsi drive on the scsi controller

```bash
qm set 8000 --scsihw virtio-scsi-pci --scsi0 local:8000/vm-8000-disk-0.raw
```

4. Add cloud init drive

```bash
qm set 8000 --ide2 local:cloudinit
```

6. Make the cloud init drive bootable and restrict BIOS to boot from disk only

```bash
qm set 8000 --boot c --bootdisk scsi0
```

7. Add serial console

```bash
qm set 8000 --serial0 socket --vga serial0
```
8. Start the vm and install all reqiured packages

9. Create template.

```bash
qm template 8000
```

10. Clone template.

```bash
qm clone 8000 135 --name test-lab --full
```