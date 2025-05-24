### A comprehensive list of Linux commands used to create new partitions

🧰 1. fdisk (for MBR disks)
📌 Use case: Basic partitioning on traditional MBR disks (/dev/sdX)

```bash
sudo fdisk /dev/sdX
```

📋 Steps:

    Press n to create a new partition

    Press p for primary partition

    Enter partition number (e.g., 1)

    Specify start and end sectors (or just press Enter to use defaults)

    Press w to write changes

🧰 2. parted (for GPT/MBR disks)
📌 Use case: Advanced scripting and GPT support

```bash
sudo parted /dev/sdX
```

📋 Example (non-interactive):

```bash
sudo parted /dev/sdX --script \
  mklabel gpt \
  mkpart primary ext4 1MiB 512MiB \
  mkpart primary ext4 512MiB 100%
```

🧰 3. gdisk (GPT version of fdisk)
📌 Use case: Interactive GPT partitioning

```bash
sudo gdisk /dev/sdX
```

📋 Steps:

    n to create a new partition

    Choose default partition number and sectors

    w to write the table and exit

🧰 4. cfdisk (ncurses UI)
📌 Use case: User-friendly terminal UI for MBR/GPT

```bash
sudo cfdisk /dev/sdX
```

📋 Steps:

    Choose partition label (MBR or GPT)

    Use arrow keys to select “New”, specify size

    Set type, boot flag if needed

    Write changes, then quit

🧰 5. sfdisk (scriptable fdisk)
📌 Use case: Scripting MBR partition layouts

```bash
echo ',500M,L' | sudo sfdisk /dev/sdX
```

📋 Example using a full layout:

```bash
cat <<EOF | sudo sfdisk /dev/sdX
label: dos
label-id: 0x12345678
device: /dev/sdX
unit: sectors

/dev/sdX1 : start=2048, size=1048576, type=83
/dev/sdX2 : start=1050624, size=2097152, type=83
EOF
```

🧰 6. sgdisk (scriptable GPT tool)
📌 Use case: Scripted GPT partitioning (for /dev/nvme0n1 or /dev/sdX)

```bash
sudo sgdisk -o /dev/sdX                          # Create new GPT
sudo sgdisk -n 1:0:+512M -t 1:8300 /dev/sdX       # Add partition
sudo sgdisk -n 2:0:+5G -t 2:8300 /dev/sdX
```

🧰 7. nvme CLI (for NVMe drives, low-level management)
📌 Use case: Format or secure erase, not traditional partitioning

Install:

```bash
sudo apt install nvme-cli
```

List NVMe:

```bash
sudo nvme list
```

Format:

```bash
sudo nvme format /dev/nvme0n1
```

    ⚠️ Does not create partitions, only erases or reinitializes the disk.

Use standard tools like parted, gdisk, etc., for partitioning NVMe devices (/dev/nvme0n1).
🧰 8. gparted (GUI tool)
📌 Use case: Visual partition management

Install:

```bash
sudo apt install gparted
```

Launch:

```bash
sudo gparted
```

Use GUI to:

    Create new partition tables (GPT/MBR)

    Resize/move/create/delete partitions

🧰 9. mkfs.* (Not for partitioning, but used after partitioning)

After you create a partition, format it:

```bash
sudo mkfs.ext4 /dev/sdX1
```

Other types:

```bash
    mkfs.vfat – for FAT32

    mkfs.ntfs – for NTFS

    mkfs.xfs, mkfs.btrfs, etc.
```    