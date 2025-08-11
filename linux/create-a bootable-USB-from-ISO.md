### How to Create Bootable USB from ISO File on Linux

This assumes you already have an ISO file that you want to move to an external thumb drive; type USB storage volume.
1. Find the USB Device Name

First, connect the USB device and find the name under which it is presented on your computer. You can do this easily using the lsblk command.

```bash
lsblk
```

Find the USB device name.

As you can see, it is mounted as `sdb` in our case and can thus be accessed with its full path as `/dev/sdb.` However, if you have multiple USB sticks already connected to your system, the drive you’d like to target might be `/dev/sdc,` `/dev/sdd,` etc.
2. Unmount and Format the USB Device

After confirming your target drive, you need to unmount it before formatting it.

```bash
sudo umount /dev/sdb*
```

Next, we need to format the unmounted drive. Let's do this with the following command:

```bash
sudo mkfs.vfat -I /dev/sdb
```

3. Create a Bootable USB Using the dd Command

Were ready to copy the ISO file to the USB drive using the dd command. I'd recommend navigating to the directory where you downloaded the ISO. Let's say you put it in your user’s “Downloads” directory.

`cd ~/Downloads`

Since were already in the right directory, we can use the following command to write ISO to USB and create a bootable USB stick:

```bash
sudo dd bs=4M if=filename.iso of=/dev/sdb status=progress
```

Where “filename.iso” is, of course, replaced by the actual name of your ISO file.

    bs: Sets the default block size.
    if: Stands for “input file.” It is used to specify the location of the ISO file.
    of: Stands for “output file.” It sets where to write the ISO file. In our case, it is “/dev/sdb.”
