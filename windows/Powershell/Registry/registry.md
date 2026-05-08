# PowerShell Registry Cheat Sheet

> Windows Registry management via the `Registry` PSDrive provider and `Microsoft.Win32.Registry` .NET class

---

## Table of Contents

1. [Registry Hives & Abbreviations](#1-registry-hives--abbreviations)
2. [Navigate the Registry](#2-navigate-the-registry)
3. [Read Keys & Values](#3-read-keys--values)
4. [Create Keys & Values](#4-create-keys--values)
5. [Modify Values](#5-modify-values)
6. [Delete Keys & Values](#6-delete-keys--values)
7. [Search the Registry](#7-search-the-registry)
8. [Copy & Move Keys](#8-copy--move-keys)
9. [Registry Value Types](#9-registry-value-types)
10. [Permissions & ACLs](#10-permissions--acls)
11. [Remote Registry](#11-remote-registry)
12. [Export & Import](#12-export--import)
13. [.NET Registry Class](#13-net-registry-class)
14. [Common Registry Locations](#14-common-registry-locations)
15. [reg.exe Equivalents](#15-regexe-equivalents)
16. [Quick-Reference Table](#16-quick-reference-table)

---

## 1. Registry Hives & Abbreviations

| Full Name                        | PS Drive Abbreviation | .NET / reg.exe Name          |
|----------------------------------|-----------------------|------------------------------|
| `HKEY_LOCAL_MACHINE`             | `HKLM:`               | `HKLM` / `HKEY_LOCAL_MACHINE`|
| `HKEY_CURRENT_USER`              | `HKCU:`               | `HKCU` / `HKEY_CURRENT_USER` |
| `HKEY_CLASSES_ROOT`              | `HKCR:`               | `HKCR` / `HKEY_CLASSES_ROOT` |
| `HKEY_USERS`                     | `HKU:`                | `HKU`  / `HKEY_USERS`        |
| `HKEY_CURRENT_CONFIG`            | `HKCC:`               | `HKCC` / `HKEY_CURRENT_CONFIG`|

> `HKCR:`, `HKU:`, and `HKCC:` are not mapped by default. Map them first:
```powershell
New-PSDrive -Name HKCR -PSProvider Registry -Root HKEY_CLASSES_ROOT  | Out-Null
New-PSDrive -Name HKU  -PSProvider Registry -Root HKEY_USERS         | Out-Null
New-PSDrive -Name HKCC -PSProvider Registry -Root HKEY_CURRENT_CONFIG | Out-Null
```

---

## 2. Navigate the Registry

```powershell
# List available registry drives
Get-PSDrive -PSProvider Registry

# Change to a registry hive (like cd in filesystem)
Set-Location HKLM:
Set-Location HKCU:\Software

# View current location
Get-Location

# List subkeys of a key
Get-ChildItem HKLM:\SOFTWARE
Get-ChildItem HKCU:\Software\Microsoft

# Recursive listing of subkeys
Get-ChildItem HKLM:\SOFTWARE -Recurse -ErrorAction SilentlyContinue

# Recursive with depth limit
Get-ChildItem HKLM:\SOFTWARE -Recurse -Depth 2 -ErrorAction SilentlyContinue

# Get a key object (the key itself, not its children)
Get-Item HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion

# Check if a key exists
Test-Path HKLM:\SOFTWARE\MyApp
Test-Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"

# Get key names only
(Get-ChildItem HKLM:\SOFTWARE).Name

# Count subkeys
(Get-ChildItem HKLM:\SOFTWARE).Count
```

---

## 3. Read Keys & Values

```powershell
# Get all values in a key
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion

# Get a specific value
Get-ItemPropertyValue HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion -Name ProgramFilesDir

# Read via Get-Item (access value by name)
(Get-Item HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion).GetValue("ProgramFilesDir")

# Read all values as a table
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion |
    Format-List *

# Exclude PS-internal properties (PSPath, PSParentPath, etc.)
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion |
    Select-Object * -ExcludeProperty PS*

# List value names only
(Get-Item HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion).Property

# List values with name, data, and type
$key = Get-Item "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"
$key.Property | ForEach-Object {
    [PSCustomObject]@{
        Name  = $_
        Value = $key.GetValue($_)
        Type  = $key.GetValueKind($_)
    }
} | Format-Table -AutoSize

# Read the default value of a key (unnamed value "")
(Get-Item "HKCR:\.txt").GetValue("")

# Read a DWORD as integer
[int](Get-ItemPropertyValue "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled")

# Read multi-string (REG_MULTI_SZ) value
(Get-Item "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager").GetValue("BootExecute")

# Read binary value as hex string
$bytes = (Get-Item "HKLM:\SOFTWARE\Example").GetValue("BinaryVal")
($bytes | ForEach-Object { $_.ToString("X2") }) -join " "
```

---

## 4. Create Keys & Values

```powershell
# Create a new registry key
New-Item -Path HKCU:\Software\MyApp

# Create nested keys in one command
New-Item -Path "HKCU:\Software\MyApp\Settings" -Force

# Create a key and immediately set a value
New-Item -Path HKCU:\Software\MyApp |
    New-ItemProperty -Name "Version" -Value "1.0" -PropertyType String

# Create a String value (REG_SZ)
New-ItemProperty -Path HKCU:\Software\MyApp -Name "InstallPath" -Value "C:\MyApp" -PropertyType String

# Create a DWORD value (REG_DWORD)
New-ItemProperty -Path HKCU:\Software\MyApp -Name "MaxRetries" -Value 3 -PropertyType DWord

# Create a QWORD value (REG_QWORD)
New-ItemProperty -Path HKCU:\Software\MyApp -Name "MaxSize" -Value 1073741824 -PropertyType QWord

# Create an ExpandString value (REG_EXPAND_SZ — supports %variables%)
New-ItemProperty -Path HKCU:\Software\MyApp -Name "LogPath" -Value "%TEMP%\MyApp.log" -PropertyType ExpandString

# Create a MultiString value (REG_MULTI_SZ)
New-ItemProperty -Path HKCU:\Software\MyApp -Name "Servers" -Value @("server1","server2","server3") -PropertyType MultiString

# Create a Binary value (REG_BINARY)
New-ItemProperty -Path HKCU:\Software\MyApp -Name "Data" -Value ([byte[]](0x01,0x02,0x03,0xFF)) -PropertyType Binary

# Create or overwrite (Force)
New-ItemProperty -Path HKCU:\Software\MyApp -Name "Version" -Value "2.0" -PropertyType String -Force

# Add to startup (run at login — current user)
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" `
    -Name "MyApp" -Value "C:\MyApp\myapp.exe" -PropertyType String -Force

# Add to startup (all users — requires elevation)
New-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" `
    -Name "MyApp" -Value "C:\MyApp\myapp.exe" -PropertyType String -Force
```

---

## 5. Modify Values

```powershell
# Set an existing value (overwrites without prompting)
Set-ItemProperty -Path HKCU:\Software\MyApp -Name "Version" -Value "2.1"

# Set a DWORD value
Set-ItemProperty -Path HKCU:\Software\MyApp -Name "MaxRetries" -Value 5

# Set multiple values at once using a hashtable
$props = @{
    Version    = "2.1"
    MaxRetries = 5
    LogPath    = "C:\Logs\MyApp.log"
}
$props.GetEnumerator() | ForEach-Object {
    Set-ItemProperty -Path HKCU:\Software\MyApp -Name $_.Key -Value $_.Value
}

# Rename a value
Rename-ItemProperty -Path HKCU:\Software\MyApp -Name "OldName" -NewName "NewName"

# Enable long paths (requires elevation)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" `
    -Name "LongPathsEnabled" -Value 1 -Type DWord

# Disable Windows telemetry (example — requires elevation)
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" `
    -Name "AllowTelemetry" -Value 0 -Type DWord -Force

# Toggle a DWORD between 0 and 1
$path = "HKLM:\SOFTWARE\MyApp"
$name = "FeatureEnabled"
$curr = (Get-ItemPropertyValue $path -Name $name -ErrorAction SilentlyContinue)
Set-ItemProperty -Path $path -Name $name -Value (if ($curr -eq 1) { 0 } else { 1 })

# Append a string to a multi-string value
$path = "HKLM:\SOFTWARE\MyApp"
$name = "Servers"
$curr = [string[]](Get-ItemPropertyValue $path -Name $name)
Set-ItemProperty -Path $path -Name $name -Value ($curr + "server4")

# Increment a DWORD counter
$val = (Get-ItemPropertyValue HKCU:\Software\MyApp -Name "RunCount" -ErrorAction SilentlyContinue) ?? 0
Set-ItemProperty -Path HKCU:\Software\MyApp -Name "RunCount" -Value ($val + 1)
```

---

## 6. Delete Keys & Values

```powershell
# Delete a single value
Remove-ItemProperty -Path HKCU:\Software\MyApp -Name "OldSetting"

# Delete multiple values
Remove-ItemProperty -Path HKCU:\Software\MyApp -Name "Val1","Val2","Val3"

# Delete a key (must be empty — no subkeys)
Remove-Item -Path HKCU:\Software\MyApp

# Delete a key and ALL subkeys recursively
Remove-Item -Path HKCU:\Software\MyApp -Recurse

# Delete without confirmation prompt
Remove-Item -Path HKCU:\Software\MyApp -Recurse -Force

# Preview before deleting (WhatIf)
Remove-Item -Path HKCU:\Software\MyApp -Recurse -WhatIf
Remove-ItemProperty -Path HKCU:\Software\MyApp -Name "OldVal" -WhatIf

# Delete all values inside a key (but keep the key)
$path = "HKCU:\Software\MyApp"
(Get-Item $path).Property | ForEach-Object {
    Remove-ItemProperty -Path $path -Name $_
}

# Remove from startup
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "MyApp" -ErrorAction SilentlyContinue
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -Name "MyApp" -ErrorAction SilentlyContinue
```

---

## 7. Search the Registry

```powershell
# Search key names for a pattern
Get-ChildItem HKLM:\SOFTWARE -Recurse -ErrorAction SilentlyContinue |
    Where-Object Name -like "*Java*"

# Search value names for a pattern
Get-ChildItem HKCU:\Software -Recurse -ErrorAction SilentlyContinue |
    ForEach-Object {
        $key = $_
        $key.Property | Where-Object { $_ -like "*password*" } |
            ForEach-Object {
                [PSCustomObject]@{ Key = $key.Name; Value = $_ }
            }
    }

# Search value DATA for a string (slow on large hives)
Get-ChildItem HKLM:\SOFTWARE -Recurse -ErrorAction SilentlyContinue |
    ForEach-Object {
        $key = $_
        $key.Property | ForEach-Object {
            $val = $key.GetValue($_)
            if ($val -like "*MyApp*") {
                [PSCustomObject]@{ Key = $key.Name; Name = $_; Data = $val }
            }
        }
    }

# Find all autorun entries
$runKeys = @(
    "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run",
    "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce",
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run",
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\RunOnce"
)
$runKeys | ForEach-Object {
    $path = $_
    if (Test-Path $path) {
        Get-ItemProperty $path |
            Select-Object * -ExcludeProperty PS* |
            ForEach-Object {
                $obj = $_
                $obj.PSObject.Properties | ForEach-Object {
                    [PSCustomObject]@{ Hive = $path; Name = $_.Name; Value = $_.Value }
                }
            }
    }
} | Format-Table -AutoSize

# Find installed software (uninstall keys)
$uninst = @(
    "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall",
    "HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall",
    "HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall"
)
$uninst | ForEach-Object {
    Get-ChildItem $_ -ErrorAction SilentlyContinue |
        ForEach-Object { Get-ItemProperty $_.PSPath }
} | Where-Object DisplayName |
    Select-Object DisplayName, DisplayVersion, Publisher, InstallDate |
    Sort-Object DisplayName |
    Format-Table -AutoSize
```

---

## 8. Copy & Move Keys

```powershell
# Copy a registry key (and its values) to a new location
Copy-Item -Path HKCU:\Software\MyApp -Destination HKCU:\Software\MyApp_Backup

# Copy recursively (including all subkeys)
Copy-Item -Path HKCU:\Software\MyApp -Destination HKCU:\Software\MyApp_Backup -Recurse

# Move a key to a new location
Move-Item -Path HKCU:\Software\MyApp\OldSettings -Destination HKCU:\Software\MyApp\NewSettings

# Rename a key (Move-Item to same parent with new name)
Move-Item -Path HKCU:\Software\OldApp -Destination HKCU:\Software\NewApp

# Duplicate a key across hives (HKCU → HKLM requires elevation)
Copy-Item -Path HKCU:\Software\MyApp -Destination HKLM:\SOFTWARE\MyApp -Recurse

# Backup a key before modifying
Copy-Item -Path "HKCU:\Software\MyApp" -Destination "HKCU:\Software\MyApp_bak_$(Get-Date -Format 'yyyyMMdd')" -Recurse
```

---

## 9. Registry Value Types

| Type Name       | PS `-PropertyType` | reg.exe type | Description                              |
|-----------------|-------------------|--------------|------------------------------------------|
| `REG_SZ`        | `String`          | `/t REG_SZ`  | Plain text string                        |
| `REG_EXPAND_SZ` | `ExpandString`    | `/t REG_EXPAND_SZ` | String with `%env%` variable expansion |
| `REG_MULTI_SZ`  | `MultiString`     | `/t REG_MULTI_SZ` | Array of strings (null-separated)      |
| `REG_DWORD`     | `DWord`           | `/t REG_DWORD` | 32-bit unsigned integer                |
| `REG_QWORD`     | `QWord`           | `/t REG_QWORD` | 64-bit unsigned integer                |
| `REG_BINARY`    | `Binary`          | `/t REG_BINARY` | Raw binary data (byte array)          |
| `REG_NONE`      | `None`            | `/t REG_NONE` | No type (rarely used)                  |

```powershell
# Check the type of an existing value
$key = Get-Item "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"
$key.GetValueKind("CurrentBuildNumber")   # returns: String, DWord, Binary, etc.

# List all values with their types
$key.Property | ForEach-Object {
    [PSCustomObject]@{
        Name = $_
        Type = $key.GetValueKind($_)
        Data = $key.GetValue($_, $null, [Microsoft.Win32.RegistryValueOptions]::DoNotExpandEnvironmentNames)
    }
} | Format-Table -AutoSize
```

---

## 10. Permissions & ACLs

```powershell
# View ACL on a registry key
Get-Acl HKLM:\SOFTWARE\MyApp | Format-List

# View ACEs (access rules) in table form
(Get-Acl HKLM:\SOFTWARE\MyApp).Access |
    Select-Object IdentityReference, RegistryRights, AccessControlType, IsInherited |
    Format-Table -AutoSize

# Get the owner
(Get-Acl HKLM:\SOFTWARE\MyApp).Owner

# Copy ACL from one key to another
Get-Acl HKLM:\SOFTWARE\SourceKey | Set-Acl HKLM:\SOFTWARE\DestKey

# Grant Full Control to a user
$acl  = Get-Acl "HKLM:\SOFTWARE\MyApp"
$rule = New-Object System.Security.AccessControl.RegistryAccessRule(
    "DOMAIN\username",
    "FullControl",
    "ContainerInherit",    # inheritance: propagate to subkeys
    "None",
    "Allow"
)
$acl.AddAccessRule($rule)
Set-Acl "HKLM:\SOFTWARE\MyApp" $acl

# Grant Read-only access
$acl  = Get-Acl "HKLM:\SOFTWARE\MyApp"
$rule = New-Object System.Security.AccessControl.RegistryAccessRule(
    "DOMAIN\username", "ReadKey", "ContainerInherit", "None", "Allow"
)
$acl.AddAccessRule($rule)
Set-Acl "HKLM:\SOFTWARE\MyApp" $acl

# Remove a specific ACE
$acl  = Get-Acl "HKLM:\SOFTWARE\MyApp"
$rule = $acl.Access | Where-Object { $_.IdentityReference -eq "DOMAIN\username" }
$acl.RemoveAccessRule($rule) | Out-Null
Set-Acl "HKLM:\SOFTWARE\MyApp" $acl

# Take ownership (requires elevation + SeTakeOwnershipPrivilege)
$acl = Get-Acl "HKLM:\SOFTWARE\MyApp"
$acl.SetOwner([System.Security.Principal.NTAccount]"Administrators")
Set-Acl "HKLM:\SOFTWARE\MyApp" $acl

# Disable inheritance and copy existing ACEs
$acl = Get-Acl "HKLM:\SOFTWARE\MyApp"
$acl.SetAccessRuleProtection($true, $true)
Set-Acl "HKLM:\SOFTWARE\MyApp" $acl

# RegistryRights values (common):
#   ReadKey         — QueryValues + EnumerateSubKeys + Notify + ReadPermissions
#   WriteKey        — SetValue + CreateSubKey + WritePermissions
#   FullControl     — all rights
#   QueryValues     — read values
#   SetValue        — write/create values
#   CreateSubKey    — create subkeys
#   EnumerateSubKeys — list subkeys
#   Delete          — delete the key
#   ChangePermissions — modify ACL
#   TakeOwnership   — take ownership
```

---

## 11. Remote Registry

```powershell
# Open a remote registry hive (.NET — requires Remote Registry service running)
$reg  = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', 'RemoteServer')
$key  = $reg.OpenSubKey("SOFTWARE\Microsoft\Windows NT\CurrentVersion")
$key.GetValue("CurrentBuildNumber")
$key.Close()
$reg.Close()

# Remote registry via Invoke-Command (WinRM — more common)
Invoke-Command -ComputerName RemoteServer -ScriptBlock {
    Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion" |
        Select-Object CurrentBuildNumber, ProductName, ReleaseId
}

# Read a remote value via Invoke-Command
Invoke-Command -ComputerName RemoteServer -ScriptBlock {
    (Get-ItemPropertyValue "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion" -Name "ProductName")
}

# Write a value to a remote registry
Invoke-Command -ComputerName RemoteServer -ScriptBlock {
    Set-ItemProperty -Path "HKLM:\SOFTWARE\MyApp" -Name "Managed" -Value 1 -Type DWord
}

# Check autorun entries on remote machines
$servers = "Server01","Server02"
Invoke-Command -ComputerName $servers -ScriptBlock {
    Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" |
        Select-Object * -ExcludeProperty PS* |
        ForEach-Object {
            $obj = $_
            $obj.PSObject.Properties | ForEach-Object {
                [PSCustomObject]@{
                    Host  = $env:COMPUTERNAME
                    Name  = $_.Name
                    Value = $_.Value
                }
            }
        }
} | Format-Table -AutoSize

# Enable Remote Registry service on a remote machine
Invoke-Command -ComputerName RemoteServer -ScriptBlock {
    Set-Service -Name RemoteRegistry -StartupType Automatic
    Start-Service -Name RemoteRegistry
}
```

---

## 12. Export & Import

```powershell
# Export a key to a .reg file (via reg.exe)
reg export "HKCU\Software\MyApp" "C:\backup\MyApp.reg"
reg export "HKLM\SOFTWARE\MyApp" "C:\backup\MyApp.reg" /y   # /y = overwrite

# Import a .reg file
reg import "C:\backup\MyApp.reg"

# Export entire hive (large — use with care)
reg export HKLM "C:\backup\HKLM.reg"
reg export HKCU "C:\backup\HKCU.reg"

# Export to .reg via PowerShell (calls reg.exe)
$key  = "HKCU\Software\MyApp"
$file = "C:\backup\MyApp_$(Get-Date -Format 'yyyyMMdd_HHmmss').reg"
reg export $key $file /y

# Export key properties to CSV
Get-ItemProperty "HKCU:\Software\MyApp" |
    Select-Object * -ExcludeProperty PS* |
    Export-Csv "C:\backup\MyApp-registry.csv" -NoTypeInformation

# Export key properties to JSON
$key    = Get-Item "HKCU:\Software\MyApp"
$values = $key.Property | ForEach-Object {
    [PSCustomObject]@{ Name = $_; Value = $key.GetValue($_); Type = $key.GetValueKind($_) }
}
$values | ConvertTo-Json | Set-Content "C:\backup\MyApp-registry.json"

# Restore from JSON export
$values = Get-Content "C:\backup\MyApp-registry.json" | ConvertFrom-Json
$values | ForEach-Object {
    Set-ItemProperty -Path "HKCU:\Software\MyApp" -Name $_.Name -Value $_.Value
}

# reg save / reg restore (binary hive format — for SAM, SYSTEM, etc.)
reg save HKLM\SAM "C:\backup\SAM.hiv"
reg restore HKLM\SAM "C:\backup\SAM.hiv"
```

---

## 13. .NET Registry Class

```powershell
# Open a key (read-only)
$key = [Microsoft.Win32.Registry]::LocalMachine.OpenSubKey("SOFTWARE\Microsoft\Windows NT\CurrentVersion")
$key.GetValue("ProductName")
$key.GetValueNames()    # list all value names
$key.GetSubKeyNames()   # list subkeys
$key.Close()

# Open a key (read-write)
$key = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey("Software\MyApp", $true)
$key.SetValue("Version", "2.0", [Microsoft.Win32.RegistryValueKind]::String)
$key.Close()

# Create a subkey
$key    = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey("Software", $true)
$newKey = $key.CreateSubKey("MyNewApp")
$newKey.SetValue("Setting1", "Value1")
$newKey.Close()
$key.Close()

# Delete a value
$key = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey("Software\MyApp", $true)
$key.DeleteValue("ObsoleteSetting")
$key.Close()

# Delete a subkey tree
$key = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey("Software", $true)
$key.DeleteSubKeyTree("MyApp")
$key.Close()

# Check if a key exists
$exists = [Microsoft.Win32.Registry]::LocalMachine.OpenSubKey("SOFTWARE\MyApp") -ne $null

# Get value type without opening key via provider
$key  = [Microsoft.Win32.Registry]::LocalMachine.OpenSubKey("SOFTWARE\Microsoft\Windows NT\CurrentVersion")
$kind = $key.GetValueKind("CurrentBuild")   # returns RegistryValueKind enum
$key.Close()

# Open remote registry via .NET
$remote = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey(
    [Microsoft.Win32.RegistryHive]::LocalMachine,
    "RemoteServer"
)
$key = $remote.OpenSubKey("SOFTWARE\MyApp")
$key.GetValue("Version")
$key.Close()
$remote.Close()

# RegistryValueKind enum values:
#   String, ExpandString, Binary, DWord, MultiString, QWord, None, Unknown
```

---

## 14. Common Registry Locations

```powershell
# OS information
"HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion"

# Autorun — current user
"HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
"HKCU:\Software\Microsoft\Windows\CurrentVersion\RunOnce"

# Autorun — all users (requires elevation)
"HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run"
"HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce"

# Installed software (64-bit)
"HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall"

# Installed software (32-bit on 64-bit OS)
"HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall"

# Installed software (current user)
"HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall"

# Environment variables (system)
"HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Environment"

# Environment variables (user)
"HKCU:\Environment"

# File associations
"HKCR:\.txt"        # handler for .txt extension
"HKCR:\txtfile"     # definition of txtfile handler

# Services
"HKLM:\SYSTEM\CurrentControlSet\Services"

# Network interfaces
"HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces"

# Windows Defender exclusions
"HKLM:\SOFTWARE\Microsoft\Windows Defender\Exclusions\Paths"
"HKLM:\SOFTWARE\Microsoft\Windows Defender\Exclusions\Extensions"

# RDP enable/disable (0 = enabled, 1 = disabled)
"HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server" # fDenyTSConnections

# UAC settings
"HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System"

# Firewall
"HKLM:\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy"

# Proxy settings (current user)
"HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings"

# DNS suffix search list
"HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters"

# Timezone
"HKLM:\SYSTEM\CurrentControlSet\Control\TimeZoneInformation"

# Mapped drives (per user)
"HKCU:\Network"

# MUI (cached file info — useful for forensics)
"HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs"
"HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU"

# USB devices (forensics)
"HKLM:\SYSTEM\CurrentControlSet\Enum\USBSTOR"
```

---

## 15. reg.exe Equivalents

```cmd
:: Query a key and all values
reg query "HKCU\Software\MyApp"

:: Query recursively
reg query "HKCU\Software\MyApp" /s

:: Query a specific value
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v ProductName

:: Query by value data (search across hive)
reg query "HKCU\Software" /f "MyApp" /s

:: Query by value name pattern
reg query "HKCU\Software" /f "Version" /v /s

:: Add a string value
reg add "HKCU\Software\MyApp" /v "Setting" /t REG_SZ /d "Value" /f

:: Add a DWORD value
reg add "HKCU\Software\MyApp" /v "Flag" /t REG_DWORD /d 1 /f

:: Add a multi-string value
reg add "HKCU\Software\MyApp" /v "List" /t REG_MULTI_SZ /d "val1\0val2\0val3" /f

:: Delete a value
reg delete "HKCU\Software\MyApp" /v "OldSetting" /f

:: Delete a key and all subkeys
reg delete "HKCU\Software\MyApp" /f

:: Export to .reg file
reg export "HKCU\Software\MyApp" "C:\backup.reg" /y

:: Import from .reg file
reg import "C:\backup.reg"

:: Copy a key to another location
reg copy "HKCU\Software\MyApp" "HKCU\Software\MyApp_Backup" /s /f

:: Compare two keys
reg compare "HKCU\Software\MyApp" "HKCU\Software\MyApp_Backup" /s

:: Load an offline hive
reg load "HKLM\TempHive" "C:\Windows\System32\config\SOFTWARE"
reg unload "HKLM\TempHive"
```

---

## 15. reg.exe vs PowerShell Equivalents

| reg.exe task                      | PowerShell equivalent                                                 |
|-----------------------------------|-----------------------------------------------------------------------|
| `reg query KEY`                   | `Get-ItemProperty KEY`                                                |
| `reg query KEY /s`                | `Get-ChildItem KEY -Recurse`                                          |
| `reg query KEY /v ValueName`      | `Get-ItemPropertyValue KEY -Name ValueName`                           |
| `reg query KEY /f pattern /s`     | `Get-ChildItem KEY -Recurse \| Where-Object ...`                      |
| `reg add KEY /v Name /t Type /d Data /f` | `New-ItemProperty KEY -Name ... -Value ... -PropertyType ...` |
| `reg add KEY /f` (create key)     | `New-Item KEY -Force`                                                 |
| `reg delete KEY /v Name /f`       | `Remove-ItemProperty KEY -Name ...`                                   |
| `reg delete KEY /f`               | `Remove-Item KEY -Recurse -Force`                                     |
| `reg copy src dst /s /f`          | `Copy-Item src dst -Recurse`                                          |
| `reg export KEY file.reg /y`      | `reg export KEY file.reg /y` *(no PS native equivalent)*             |
| `reg import file.reg`             | `reg import file.reg` *(no PS native equivalent)*                    |
| `reg load HKLM\Temp hive_file`    | *(use reg.exe — no PS cmdlet)*                                        |

---

## 16. Quick-Reference Table

| Cmdlet / Tool              | Purpose                                               |
|----------------------------|-------------------------------------------------------|
| `Get-Item`                 | Get a key object (read subkey names and value names)  |
| `Get-ChildItem`            | List subkeys of a key                                 |
| `Get-ItemProperty`         | Read all values of a key                              |
| `Get-ItemPropertyValue`    | Read a specific value's data                          |
| `New-Item`                 | Create a new registry key                             |
| `New-ItemProperty`         | Create a new registry value                           |
| `Set-Item`                 | Set a key's default value                             |
| `Set-ItemProperty`         | Modify an existing value's data                       |
| `Rename-ItemProperty`      | Rename a value                                        |
| `Remove-Item`              | Delete a key (use `-Recurse` for subkeys)             |
| `Remove-ItemProperty`      | Delete a value                                        |
| `Copy-Item`                | Copy a key to another location                        |
| `Move-Item`                | Move or rename a key                                  |
| `Test-Path`                | Check if a key or value path exists                   |
| `Get-Acl` / `Set-Acl`     | Read / write registry key permissions                 |
| `New-PSDrive`              | Map unmapped hives (HKCR, HKU, HKCC)                 |
| `reg export`               | Export key to `.reg` file                             |
| `reg import`               | Import `.reg` file                                    |
| `reg load` / `reg unload`  | Mount / unmount offline hive files                    |

---

*Generated 2026-05-08 — Windows PowerShell 5.1 / PowerShell 7.x*
