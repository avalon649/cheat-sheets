# Fix a USB Drive Showing as Multiple Partitions on Windows

  > **Warning:** This process wipes all data on the drive. Back up anything important
  before proceeding.

  ---

  ## Steps

  ### 1. Open an Elevated Shell
  Press **Win + X** and select **PowerShell (Admin)** or **Command Prompt (Admin)**, then
  click **Yes** on the UAC prompt.

  ### 2. Launch DiskPart
  ```cmd
  diskpart

  3. Identify Your USB Drive

  list disk
  Note the disk number that corresponds to your USB drive.

  4. Select the Drive

  select disk [NUMBER]

  ▎ Caution: Double-check the disk number. Selecting the wrong disk will destroy data on
  ▎ that drive.

  5. Wipe All Partitions

  clean

  ▎ If you get an error, close any open File Explorer windows pointing to the drive and
  ▎ run clean again.

  6. Create a New Primary Partition

  create partition primary

  7. Format the Drive

  format fs=ntfs quick

  ▎ Swap ntfs for fat32 if you need cross-platform compatibility (e.g., Linux, macOS, game
  ▎  consoles).

  8. Mark the Partition Active

  active

  9. Assign a Drive Letter

  assign

  10. Verify the Drive

  list disk
  Confirm the drive status shows as Healthy.

  11. Exit DiskPart

  exit

  ---
  Visual Confirmation

  Type Disk Management into the Windows search bar, open it, and select your USB drive to
  visually confirm the partition layout looks correct.
  ```