# PowerShell Cheat-Sheet
PowerShell is a task automation and configuration management program from Microsoft, consisting of a command-line shell and the associated scripting language.

## Install PowerShell
PowerShell was made open-source and cross-platform with PowerShell Core, and can be installed on multiple operating systems.

### Windows
1. Download MSI Package from the [Official PowerShell Docs](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2)
2. Set up PowerShell Profile in [Windows Terminal](windows/windows-terminal.md).
```json
"commandline": "pwsh.exe -nologo",
"name": "Powershell",
"source": "Windows.Terminal.PowershellCore"
```

### Linux (Ubuntu)
```sh
# Update the list of packages
sudo apt-get update
# Install pre-requisite packages.
sudo apt-get install -y wget apt-transport-https software-properties-common
# Download the Microsoft repository GPG keys
wget -q https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb
# Register the Microsoft repository GPG keys
sudo dpkg -i packages-microsoft-prod.deb
# Update the list of packages after we added packages.microsoft.com
sudo apt-get update
# Install PowerShell
sudo apt-get install -y powershell
# Start PowerShell
pwsh
```

## Profile
Set up a PowerShell Profile by opening the profile script :
```powershell
code $PROFILE
```

## (Optional) Set up starship Prompt
You can customize the look and feel of PowerShell with the Starship Prompt ([[starship]]).
-----------------------------------------------------------------------------

### Powershell Commands

1. System Information

```powershell
# Get system information
Get-ComputerInfo

# Get OS version
[System.Environment]::OSVersion

# Get the hostname of the computer
$env:COMPUTERNAME

# Get system architecture (x86 or x64)
[System.Environment]::Is64BitOperatingSystem

# Get CPU information
Get-WmiObject Win32_Processor

# Get memory information
Get-WmiObject Win32_ComputerSystem

# Get system uptime
(Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime
```

2. Files and Directories

```powershell
# Get the current directory
Get-Location

# Change the current directory
Set-Location C:\Path\To\Directory

# List all files and directories in the current location
Get-ChildItem

# List files with specific extension (e.g., .txt)
Get-ChildItem *.txt

# Get detailed file information
Get-Item "C:\path\to\file.txt"

# Copy a file
Copy-Item "C:\path\to\source.txt" "C:\path\to\destination.txt"

# Move a file
Move-Item "C:\path\to\source.txt" "C:\path\to\destination.txt"

# Rename a file
Rename-Item "C:\path\to\oldname.txt" "newname.txt"

# Remove a file
Remove-Item "C:\path\to\file.txt"

# Create a new directory
New-Item -ItemType Directory -Path "C:\path\to\new_directory"

# Remove a directory
Remove-Item "C:\path\to\directory" -Recurse
```

3. Environment Variables

```powershell
# List all environment variables
Get-ChildItem Env:
ls:Env

# Access a specific environment variable
$env:PATH

# Set an environment variable (for the session)
$env:MY_VAR = "value"

# Remove an environment variable (for the session)
Remove-Item Env:MY_VAR
```

4. Variables and Objects

```powershell
# Declare a variable
$myVar = "Hello, PowerShell!"

# Get the value of a variable
$myVar

# Create an array
$myArray = @("Apple", "Banana", "Cherry")

# Access an array element
$myArray[1]  # Banana

# Create a hashtable (dictionary)
$myHashtable = @{ Key1 = "Value1"; Key2 = "Value2" }

# Access hashtable values
$myHashtable["Key1"]

# Get the type of a variable
$myVar.GetType()

# Clear the variable
Remove-Variable myVar
```

5. Processes and Services

```powershell
# List all running processes
Get-Process

# Get a specific process
Get-Process -Name "chrome"

# Start a process
Start-Process "notepad.exe"

# Stop a process by name
Stop-Process -Name "notepad"

# Get a list of services
Get-Service

# Start a service
Start-Service -Name "wuauserv"

# Stop a service
Stop-Service -Name "wuauserv"

# Get detailed information about a service
Get-Service -Name "wuauserv" | Select-Object *
```

6. Networking

```powershell
# Display network adapters and their configuration
Get-NetAdapter

# Get IP configuration details
Get-NetIPAddress

# Test network connectivity (ping)
Test-Connection www.google.com

# Display active network connections
Get-NetTCPConnection

# Get DNS settings
Get-DnsClientServerAddress

# Display routing table
Get-NetRoute
```

7. System Configuration

```powershell
# Set system timezone
Set-TimeZone -Name "Pacific Standard Time"

# Set date and time
Set-Date "2025-03-08 12:00:00"

# Get PowerShell version
$PSVersionTable.PSVersion

# Set the execution policy (for running scripts)
Set-ExecutionPolicy RemoteSigned

# View the execution policy
Get-ExecutionPolicy

# View or set Windows updates
Get-WmiObject -Class Win32_QuickFixEngineering
```

8. User Management

```powershell
# List all users on the system
Get-LocalUser

# Create a new user
New-LocalUser "username" -Password (ConvertTo-SecureString "password" -AsPlainText -Force)

# Add a user to a group
Add-LocalGroupMember -Group "Administrators" -Member "username"

# Remove a user from a group
Remove-LocalGroupMember -Group "Administrators" -Member "username"

# Delete a user
Remove-LocalUser "username"
```

9. File Permissions

```powershell
# Get the ACL (Access Control List) for a file or folder
Get-Acl "C:\path\to\file_or_folder"

# Set permissions on a file or folder
Set-Acl "C:\path\to\file_or_folder" "C:\path\to\ACLFile.txt"

# Add permissions to a file/folder (e.g., FullControl for user)
$acl = Get-Acl "C:\path\to\file_or_folder"
$permission = "DOMAIN\User", "FullControl", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($permission)
$acl.AddAccessRule($accessRule)
Set-Acl "C:\path\to\file_or_folder" $acl
```

10. Scheduled Tasks

```powershell
# Get all scheduled tasks
Get-ScheduledTask

# Create a new scheduled task
New-ScheduledTask -Action (New-ScheduledTaskAction -Execute "C:\path\to\script.ps1") -Trigger (New-ScheduledTaskTrigger -At 9am -Daily) -TaskName "MyTask"

# Register the scheduled task
Register-ScheduledTask -TaskName "MyTask" -InputObject $task

# Run a scheduled task immediately
Start-ScheduledTask -TaskName "MyTask"

# Remove a scheduled task
Unregister-ScheduledTask -TaskName "MyTask" -Confirm:$false
```

11. Piping and Output

```powershell
# Pipe output to another command
Get-Process | Where-Object { $_.CPU -gt 100 }

# Pipe to a file (e.g., to save output)
Get-Process | Out-File "C:\path\to\file.txt"

# Pipe to the clipboard
Get-Process | clip

# Format output as table
Get-Process | Format-Table -Property Name, CPU

# Format output as list
Get-Process | Format-List

# Display progress bar (for long-running tasks)
Write-Progress -PercentComplete 50 -Status "Working" -Activity "Task in progress..."
```

12. Scripting and Control Flow

```powershell
# If statement
if ($a -gt 10) { Write-Output "Greater than 10" } else { Write-Output "Less than or equal to 10" }

# Loop (For)
for ($i = 0; $i -lt 5; $i++) { Write-Output "Looping $i" }

# While loop
$i = 0
while ($i -lt 5) { Write-Output "Looping $i"; $i++ }

# Foreach loop
$items = 1..5
foreach ($item in $items) { Write-Output "Item: $item" }

# Try-Catch for error handling
try {
    $result = 1 / 0
} catch {
    Write-Error "Error: $_"
}
```

13. Remote Management

```powershell
# Enter a remote session
Enter-PSSession -ComputerName "RemotePC"

# Run a command remotely
Invoke-Command -ComputerName "RemotePC" -ScriptBlock { Get-Process }

# Exit a remote session
Exit-PSSession

# Copy a file to a remote machine
Copy-Item "C:\path\to\file.txt" -Destination "C:\path\on\remote\machine"

# Get remote session details
Get-PSSession
```