### How to create an iscsi target on linux

1. Install Client Tools:

```bash
sudo apt update && sudo apt install -y open-iscsi
```

2. Get Ubuntu's Client IQN:

```bash
sudo cat /etc/iscsi/initiatorname.iscsi
```
`example initiatorname.iscsi`

```text
## DO NOT EDIT OR REMOVE THIS FILE!
## If you remove this file, the iSCSI daemon will not start.
## If you change the InitiatorName, existing access control lists
## may reject this initiator.  The InitiatorName must be unique
## for each iSCSI initiator.  Do NOT duplicate iSCSI InitiatorNames.
InitiatorName=iqn.2004-10.com.ubuntu:01:c10474e6447
```

3. Start The iSCSI Daemon:

```bash
sudo systemctl start iscsid
```

4. Configure the Ubuntu iSCSI Initiator:

* Update the Initiator Configuration:

   Make sure the IQN matches the one allowed on the TrueNAS server.

* Discover iSCSI Targets:

   Use the IP of your TrueNAS server:

```bash
sudo iscsiadm --mode discovery --type sendtargets --portal <TrueNAS_IP>
```

5. Log in to the Target:

```bash
   sudo iscsiadm --mode node --targetname <Target_IQN_From_TrueNAS> --portal <TrueNAS_IP> --login
   ```

6. Check the Connected Block Devices:

```bash
lsblk
```

7. Create a Filesystem and Mount:

```bash
   sudo mkfs.ext4 /dev/sdX  # Replace 'sdX' with your actual device
   sudo mkdir /mnt/iscsi
   sudo mount /dev/sdX /mnt/iscsi
```

8. Enable Automatic Login on Boot:

```bash
   sudo iscsiadm --mode node --targetname <Target_IQN_From_TrueNAS> --portal <TrueNAS_IP> --op update --name node.startup --value automatic
   ```