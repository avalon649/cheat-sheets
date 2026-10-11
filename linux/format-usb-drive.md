# How to Format a USB Drive on Linux (Garuda / Arch)

> ⚠️ **Formatting erases everything on the drive.** Back up anything you need first,
> and double-check the device name — picking the wrong one will wipe an internal disk.

---

## 1. Identify the USB drive

Plug the drive in, then list block devices:

```sh
lsblk -o NAME,SIZE,TYPE,TRAN,FSTYPE,LABEL,MOUNTPOINTS,MODEL
```

Look for the entry with `TRAN` = `usb` and a size matching your stick
(e.g. `sda`, `sdb`). On this machine the internal disks are `nvme0n1` and
`nvme1n1` — **never** use those.

To be extra sure, watch the kernel log while plugging the drive in:

```sh
sudo dmesg -w
```

Press `Ctrl+C` to stop once you see the new device (e.g. `sdb: sdb1`).

In the steps below, replace **`/dev/sdX`** with your drive (e.g. `/dev/sdb`)
and **`/dev/sdX1`** with its first partition (e.g. `/dev/sdb1`).

---

## 2. Unmount the drive

KDE may auto-mount the drive. Unmount every partition on it:

```sh
sudo umount /dev/sdX?* 2>/dev/null
```

Confirm nothing from it is mounted (the `MOUNTPOINTS` column should be empty):

```sh
lsblk /dev/sdX
```

---

## 3. Wipe old signatures

Removes old partition tables and filesystem signatures so you start clean:

```sh
sudo wipefs -a /dev/sdX
```

---

## 4. Create a new partition table and partition

Use **MBR (`msdos`)** for maximum compatibility (TVs, car stereos, older devices):

```sh
sudo parted -s /dev/sdX mklabel msdos mkpart primary 1MiB 100%
```

Or use **GPT** if it's only for modern computers:

```sh
sudo parted -s /dev/sdX mklabel gpt mkpart primary 1MiB 100%
```

Make the kernel re-read the new layout:

```sh
sudo partprobe /dev/sdX
```

---

## 5. Create the filesystem

Pick **one** depending on where you'll use the drive:

| Filesystem | Best for | Max file size |
|---|---|---|
| FAT32 | TVs, consoles, car stereos, anything old | 4 GB |
| exFAT | Windows / macOS / Linux, big files | Effectively unlimited |
| ext4 | Linux only | 16 TB |
| NTFS | Mainly Windows | Effectively unlimited |

### FAT32

```sh
sudo mkfs.vfat -F 32 -n MYUSB /dev/sdX1
```

(FAT32 labels: max 11 characters, uppercase recommended.)

### exFAT

```sh
sudo mkfs.exfat -n MYUSB /dev/sdX1
```

### ext4

```sh
sudo mkfs.ext4 -L MYUSB /dev/sdX1
```

After mounting an ext4 drive, give your user ownership so you can write to it
without `sudo` (see step 6 for mounting):

```sh
sudo chown "$USER":"$USER" /mnt/usb
```

### NTFS

NTFS tools aren't installed by default — install them first:

```sh
sudo pacman -S --needed ntfs-3g
```

Then format (`-f` = quick format):

```sh
sudo mkfs.ntfs -f -L MYUSB /dev/sdX1
```

---

## 6. Verify and mount

Check the new filesystem and label:

```sh
lsblk -o NAME,SIZE,FSTYPE,LABEL,MOUNTPOINTS /dev/sdX
```

### Option A — Mount as your user (recommended, same as KDE's Dolphin)

```sh
udisksctl mount -b /dev/sdX1
```

It will print where it was mounted (usually `/run/media/<user>/MYUSB`).

### Option B — Mount manually

```sh
sudo mkdir -p /mnt/usb
```

```sh
sudo mount /dev/sdX1 /mnt/usb
```

---

## 7. Copy files to it

```sh
cp /path/to/file /run/media/$USER/MYUSB/
```

(Use `/mnt/usb/` instead if you mounted manually — prefix with `sudo` for FAT32/exFAT/NTFS mounted that way.)

---

## 8. Safely eject

Flush pending writes first — important for large files:

```sh
sync
```

### If mounted with udisksctl

```sh
udisksctl unmount -b /dev/sdX1
```

```sh
udisksctl power-off -b /dev/sdX
```

### If mounted manually

```sh
sudo umount /mnt/usb
```

You can now unplug the drive.

---

## Quick reference — FAT32 in one go

```sh
DEV=/dev/sdX   # <-- CHANGE THIS after checking lsblk!
sudo umount ${DEV}?* 2>/dev/null
sudo wipefs -a $DEV
sudo parted -s $DEV mklabel msdos mkpart primary fat32 1MiB 100%
sudo partprobe $DEV
sudo mkfs.vfat -F 32 -n MYUSB ${DEV}1
lsblk -o NAME,SIZE,FSTYPE,LABEL $DEV
```

---

## Troubleshooting

**"Device or resource busy"** — something still has it mounted or open:

```sh
lsblk /dev/sdX
```

```sh
sudo fuser -vm /dev/sdX1
```

**Drive is read-only / write-protected** — check for a physical lock switch on
the stick or SD adapter, then check the kernel's read-only flag:

```sh
sudo blockdev --getro /dev/sdX
```

(`1` = read-only; a hardware switch or a failing drive is the usual cause.)

**Partition doesn't appear after `parted`** — unplug and replug the drive, or:

```sh
sudo partprobe /dev/sdX
```

**GUI alternative** — KDE Partition Manager does all of the above graphically:

```sh
sudo pacman -S --needed partitionmanager
```
