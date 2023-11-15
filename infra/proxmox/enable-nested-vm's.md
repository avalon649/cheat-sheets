# How to: Enable nested hypervisor/virtualization on Proxmox VE (PVE)

`By default, PVE does not expose hardware-assisted virtulization extentions to VMs. Performance of hypervisor within VM will be degraded. If we configure VM’s CPU as “host” and enable hardware-assisted virtulization extensions, the performance of nested hypervisor will be optimal.`

1. Login to PVE terminal or via SSH or web gui -> Shell

     `Note: Microsoft Hyper-V nested hypervisor only works with Intel CPUs. (Refer to: Run Hyper-V in a Virtual Machine with Nested Virtualization)`

2. Usually it works for

    `AMD CPU or very recent Intel one
    Use kernel >= 3.10 (Is always the case in Proxmox VE 4.x)
    Enable nested support`

3. `To check if nested virtulizatoin is enabled, bring up terminal/SSH/Shell, execute following command`

```bash
cat /sys/module/kvm_intel/parameters/nested
```
If it returns “N”, that means it’s disabled

To enable

### Intel
```
echo "options kvm-intel nested=Y" > /etc/modprobe.d/kvm-intel.conf
```
### AMD
```
echo "options kvm-amd nested=1" > /etc/modprobe.d/kvm-amd.conf
```
Then

### Intel
```
modprobe -r kvm_intel
modprobe kvm_intel
```
### AMD
```
modprobe -r kvm_amd
modprobe kvm_amd
```
Note: If error returns, just reboot the PVE host

Check again

### Intel
```
cat /sys/module/kvm_intel/parameters/nested
```
### AMD
```
cat /sys/module/kvm_amd/parameters/nested
```
We should get “Y” which means nested virtulization is enabled

4 To enable nested virtulizatoin for guest VMs

Intel CPU:
```
– Set the CPU type for VMs to “host”
```

AMD CPU:
```
– Set the CPU type for VMs to “host”
– Add following flags to the configuration file
```
```
args: -cpu host,+svm
```

5 We can use following command to check/verify hardware virtualization support is enabled or not on Linux OSs
```
egrep '(vmx|svm)' --color=always /proc/cpuinfo
```