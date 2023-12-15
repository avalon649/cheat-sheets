
```
How to fix a USB drive showing up as two drives (fragmented into multiple partitions) on Windows:

    Hold the Windows key and press X, select PowerShell (Admin), select Yes to the pop-up. You can also use Command Prompt.
    In the Powershell interface type diskpart to enter the disk partition tool.
    Type list disk to see all disks listed.
    Select the USB drive by typing select disk [NUMBER]. Be careful to select the correct drive.
    Type clean. An error will occur if you have the drive folder open, close the window and repeat the command if this happens.
    Type create partition primary.
    Type format fs=ntfs quick to format the drive (you can also choose to set fs=fat32).
    Type active.
    Type assign.
    Type list disk and confirm your drive looks healthy.
    Type exit to exit.

You can get visual confirmation by typing ‘disk manager’ into the windows search bar in the bottom left. Select the USB to view.
```