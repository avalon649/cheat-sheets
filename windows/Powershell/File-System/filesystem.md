# PowerShell File System Cheat Sheet

> File and directory management via `Microsoft.PowerShell.Management` and .NET `System.IO`

---

## Table of Contents

1. [Navigation & Location](#1-navigation--location)
2. [List & Explore](#2-list--explore)
3. [Create Files & Directories](#3-create-files--directories)
4. [Read Files](#4-read-files)
5. [Write & Append Files](#5-write--append-files)
6. [Copy, Move & Rename](#6-copy-move--rename)
7. [Delete Files & Directories](#7-delete-files--directories)
8. [Search & Find](#8-search--find)
9. [File Attributes & Metadata](#9-file-attributes--metadata)
10. [Permissions & ACLs](#10-permissions--acls)
11. [Symbolic Links & Junctions](#11-symbolic-links--junctions)
12. [Drives & Providers](#12-drives--providers)
13. [Compression & Archives](#13-compression--archives)
14. [Hashing & Integrity](#14-hashing--integrity)
15. [Streams (ADS)](#15-streams-ads)
16. [Monitoring & Watching](#16-monitoring--watching)
17. [.NET System.IO Methods](#17-net-systemio-methods)
18. [cmd.exe Equivalents](#18-cmdexe-equivalents)
19. [Quick-Reference Table](#19-quick-reference-table)

---

## 1. Navigation & Location

```powershell
# Show current directory
Get-Location          # full object
(Get-Location).Path   # string only
$PWD                  # automatic variable
$PWD.Path

# Change directory
Set-Location C:\Windows
Set-Location ..           # up one level
Set-Location ~            # home directory ($HOME)
Set-Location -            # previous location (toggle)
Set-Location HKLM:        # switch to registry provider

# Push/pop location stack
Push-Location C:\Temp
# ... do work ...
Pop-Location              # return to previous location

# Stack multiple locations
Push-Location C:\One
Push-Location C:\Two
Get-Location -Stack       # view stack
Pop-Location              # back to C:\One
Pop-Location              # back to original

# Resolve a relative path to absolute
Resolve-Path .\subdir
Resolve-Path ~\Documents

# Test if a path exists
Test-Path C:\Windows                      # True/False
Test-Path C:\file.txt -PathType Leaf      # file only
Test-Path C:\Windows  -PathType Container # directory only
Test-Path C:\*.log                        # wildcard

# Split path components
Split-Path "C:\Users\Avalon\file.txt" -Parent    # C:\Users\Avalon
Split-Path "C:\Users\Avalon\file.txt" -Leaf      # file.txt
Split-Path "C:\Users\Avalon\file.txt" -LeafBase  # file  (PS 6+)
Split-Path "C:\Users\Avalon\file.txt" -Extension # .txt  (PS 6+)
Split-Path "C:\Users\Avalon\file.txt" -Qualifier # C:
Split-Path "C:\Users\Avalon\file.txt" -NoQualifier # \Users\Avalon\file.txt

# Join path components safely
Join-Path C:\Users Avalon Documents         # C:\Users\Avalon\Documents
Join-Path $env:USERPROFILE "Desktop\notes.txt"
```

---

## 2. List & Explore

```powershell
# Basic listing
Get-ChildItem                       # current directory
Get-ChildItem C:\Windows
Get-ChildItem ~\Documents

# Aliases: ls, dir, gci

# Include hidden and system files
Get-ChildItem -Force
Get-ChildItem C:\Users -Force

# Recursive listing
Get-ChildItem C:\Logs -Recurse

# Recurse with depth limit (PS 5+)
Get-ChildItem C:\Projects -Recurse -Depth 2

# Filter by extension
Get-ChildItem C:\Scripts -Filter *.ps1
Get-ChildItem C:\Scripts -Include *.ps1,*.psm1 -Recurse

# Exclude patterns
Get-ChildItem C:\ -Exclude *.tmp -Recurse

# Files only
Get-ChildItem -File
Get-ChildItem C:\Logs -Recurse -File

# Directories only
Get-ChildItem -Directory
Get-ChildItem C:\Projects -Recurse -Directory

# Sort output
Get-ChildItem | Sort-Object Length -Descending          # largest first
Get-ChildItem | Sort-Object LastWriteTime -Descending   # newest first
Get-ChildItem | Sort-Object Name                        # alphabetical

# Select specific properties
Get-ChildItem | Select-Object Name, Length, LastWriteTime

# Human-readable size column
Get-ChildItem C:\Logs -File |
    Select-Object Name, LastWriteTime,
        @{N='Size (KB)'; E={[math]::Round($_.Length / 1KB, 2)}} |
    Sort-Object 'Size (KB)' -Descending

# Total size of a directory
(Get-ChildItem C:\Logs -Recurse -File | Measure-Object Length -Sum).Sum / 1MB

# Count items
(Get-ChildItem C:\Scripts -Recurse -File).Count
(Get-ChildItem C:\Logs -Recurse -File | Where-Object Extension -eq '.log').Count

# Largest files in a directory tree
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue |
    Sort-Object Length -Descending |
    Select-Object -First 20 FullName, @{N='MB'; E={[math]::Round($_.Length/1MB,2)}}

# Files modified in the last 24 hours
Get-ChildItem C:\Logs -Recurse -File |
    Where-Object LastWriteTime -gt (Get-Date).AddDays(-1)

# Files older than 30 days
Get-ChildItem C:\Temp -Recurse -File |
    Where-Object LastWriteTime -lt (Get-Date).AddDays(-30)

# Empty files
Get-ChildItem C:\ -Recurse -File | Where-Object Length -eq 0

# Empty directories
Get-ChildItem C:\ -Recurse -Directory |
    Where-Object { (Get-ChildItem $_.FullName -Force).Count -eq 0 }
```

---

## 3. Create Files & Directories

```powershell
# Create a directory
New-Item -ItemType Directory -Path C:\MyFolder
New-Item -ItemType Directory -Path C:\A\B\C    # creates parent chain
mkdir C:\MyFolder                              # alias

# Create multiple directories at once
"Logs","Config","Data" | ForEach-Object { New-Item -ItemType Directory -Path "C:\App\$_" }

# Create an empty file
New-Item -ItemType File -Path C:\Temp\file.txt

# Create a file with content
New-Item -ItemType File -Path C:\Temp\file.txt -Value "Hello, World!"

# Create file only if it doesn't exist
if (!(Test-Path C:\Temp\file.txt)) {
    New-Item -ItemType File -Path C:\Temp\file.txt
}

# Create directory structure from array
$dirs = @("C:\App\bin","C:\App\logs","C:\App\config","C:\App\data")
$dirs | ForEach-Object { New-Item -ItemType Directory -Force -Path $_ }

# Touch a file (create or update timestamp)
$file = "C:\Temp\touched.txt"
if (Test-Path $file) {
    (Get-Item $file).LastWriteTime = Get-Date
} else {
    New-Item -ItemType File -Path $file
}
```

---

## 4. Read Files

```powershell
# Read entire file as array of lines
Get-Content C:\file.txt
$lines = Get-Content C:\file.txt

# Read as single string
Get-Content C:\file.txt -Raw

# Read first N lines
Get-Content C:\file.txt -TotalCount 10   # head
Get-Content C:\file.txt -First 10        # alias parameter

# Read last N lines
Get-Content C:\file.txt -Tail 10         # tail
Get-Content C:\file.txt -Last 10

# Tail and follow (like tail -f)
Get-Content C:\Logs\app.log -Tail 20 -Wait

# Read with specific encoding
Get-Content C:\file.txt -Encoding UTF8
Get-Content C:\file.txt -Encoding Unicode
Get-Content C:\file.txt -Encoding ASCII
# Encoding values: ASCII, UTF8, UTF8BOM, UTF8NoBOM, Unicode, UTF32, BigEndianUnicode, Default, OEM

# Read a binary file as bytes
$bytes = [System.IO.File]::ReadAllBytes("C:\file.bin")
Get-Content C:\file.bin -Encoding Byte   # PS 5.1
Get-Content C:\file.bin -AsByteStream    # PS 6+

# Read and process line by line (memory efficient for large files)
foreach ($line in [System.IO.File]::ReadLines("C:\bigfile.txt")) {
    # process $line
}

# Read specific line numbers
$lines = Get-Content C:\file.txt
$lines[0]       # first line (0-indexed)
$lines[-1]      # last line
$lines[5..10]   # lines 6-11

# Read a CSV
Import-Csv C:\data.csv
Import-Csv C:\data.csv -Delimiter ";"

# Read a JSON file
Get-Content C:\config.json | ConvertFrom-Json

# Read an XML file
[xml]$xml = Get-Content C:\config.xml
$xml.SelectNodes("//element")

# Read file with StreamReader (large files, low memory)
$reader = [System.IO.StreamReader]::new("C:\bigfile.txt")
while (($line = $reader.ReadLine()) -ne $null) {
    # process $line
}
$reader.Close()
```

---

## 5. Write & Append Files

```powershell
# Write (overwrite) a file
Set-Content -Path C:\file.txt -Value "Line one"
"Line one" | Set-Content C:\file.txt

# Write multiple lines
Set-Content C:\file.txt -Value "Line 1","Line 2","Line 3"
@("Line 1","Line 2","Line 3") | Set-Content C:\file.txt

# Append to a file
Add-Content -Path C:\file.txt -Value "New line"
"New line" | Add-Content C:\file.txt

# Write with specific encoding
Set-Content C:\file.txt -Value "content" -Encoding UTF8
Set-Content C:\file.txt -Value "content" -Encoding Unicode

# Write raw string (no newlines added)
[System.IO.File]::WriteAllText("C:\file.txt", "content")

# Append raw string
[System.IO.File]::AppendAllText("C:\file.txt", "more content")

# Write bytes (binary)
$bytes = [byte[]](0x48,0x65,0x6C,0x6C,0x6F)
[System.IO.File]::WriteAllBytes("C:\file.bin", $bytes)
Set-Content C:\file.bin -Value $bytes -AsByteStream   # PS 6+

# Write CSV
$data = @(
    [PSCustomObject]@{Name="Alice";Age=30}
    [PSCustomObject]@{Name="Bob";  Age=25}
)
$data | Export-Csv C:\people.csv -NoTypeInformation
$data | Export-Csv C:\people.csv -NoTypeInformation -Append   # append rows

# Write JSON
$obj = @{Name="Alice"; Roles=@("Admin","User")}
$obj | ConvertTo-Json | Set-Content C:\config.json
$obj | ConvertTo-Json -Depth 10 | Set-Content C:\config.json  # deep objects

# Write XML
$xml = [xml]"<root><item>value</item></root>"
$xml.Save("C:\config.xml")

# Here-string to file
@"
Line one
Line two
Line three
"@ | Set-Content C:\multiline.txt

# Out-File (preserves formatted output)
Get-Process | Out-File C:\processes.txt
Get-Process | Out-File C:\processes.txt -Append
Get-Process | Out-File C:\processes.txt -Width 200   # prevent line wrapping

# Redirect operators
Get-Process > C:\processes.txt       # overwrite
Get-Process >> C:\processes.txt      # append
Get-Process 2> C:\errors.txt         # stderr only
Get-Process *> C:\all.txt            # stdout + stderr

# StreamWriter (high-performance, large output)
$writer = [System.IO.StreamWriter]::new("C:\output.txt", $false)  # $false = overwrite
for ($i = 0; $i -lt 100000; $i++) {
    $writer.WriteLine("Line $i")
}
$writer.Close()
```

---

## 6. Copy, Move & Rename

```powershell
# Copy a file
Copy-Item C:\src\file.txt C:\dst\
Copy-Item C:\src\file.txt C:\dst\file_copy.txt

# Copy and overwrite without prompt
Copy-Item C:\src\file.txt C:\dst\ -Force

# Copy a directory recursively
Copy-Item C:\src -Destination C:\dst -Recurse

# Copy contents of directory (not the directory itself)
Copy-Item C:\src\* -Destination C:\dst -Recurse

# Copy multiple files (wildcard)
Copy-Item C:\Logs\*.log C:\Backup\

# Copy with verbose output
Copy-Item C:\src C:\dst -Recurse -Verbose

# Move a file
Move-Item C:\file.txt C:\NewFolder\
Move-Item C:\file.txt C:\NewFolder\renamed.txt

# Move directory
Move-Item C:\OldFolder C:\NewFolder

# Move and overwrite
Move-Item C:\file.txt C:\dst\file.txt -Force

# Rename a file
Rename-Item C:\file.txt newname.txt
Rename-Item C:\file.txt -NewName "newname.txt"

# Rename extension of all files in a directory
Get-ChildItem C:\Logs -Filter *.log |
    Rename-Item -NewName { $_.Name -replace '\.log$', '.txt' }

# Bulk rename with counter
$i = 1
Get-ChildItem C:\Photos -Filter *.jpg | ForEach-Object {
    Rename-Item $_.FullName -NewName ("photo_{0:D4}.jpg" -f $i++)
}

# Rename to add date prefix
Get-ChildItem C:\Reports -Filter *.docx |
    Rename-Item -NewName { (Get-Date -Format "yyyy-MM-dd") + "_" + $_.Name }

# Copy preserving ACLs
Copy-Item C:\src\file.txt C:\dst\file.txt
Get-Acl C:\src\file.txt | Set-Acl C:\dst\file.txt
```

---

## 7. Delete Files & Directories

```powershell
# Delete a file
Remove-Item C:\file.txt

# Delete without confirmation prompt
Remove-Item C:\file.txt -Force

# Delete a directory and all contents
Remove-Item C:\MyFolder -Recurse

# Delete without any prompt (force + recurse)
Remove-Item C:\MyFolder -Recurse -Force

# Delete by wildcard
Remove-Item C:\Logs\*.tmp
Remove-Item C:\Temp\* -Recurse -Force   # empty a directory without removing it

# Delete files older than 30 days
Get-ChildItem C:\Logs -Recurse -File |
    Where-Object LastWriteTime -lt (Get-Date).AddDays(-30) |
    Remove-Item -Force

# Delete empty directories
Get-ChildItem C:\ -Recurse -Directory |
    Where-Object { (Get-ChildItem $_.FullName -Force).Count -eq 0 } |
    Remove-Item

# Safe delete: preview before removing (WhatIf)
Remove-Item C:\Logs\*.log -WhatIf
Get-ChildItem C:\Temp -Recurse | Remove-Item -WhatIf

# Delete using .NET (bypasses some path-length limits)
[System.IO.File]::Delete("C:\file.txt")
[System.IO.Directory]::Delete("C:\MyFolder", $true)  # $true = recursive

# Secure delete (overwrite before deleting — basic)
$path = "C:\sensitive.txt"
$bytes = [System.IO.File]::ReadAllBytes($path)
[Array]::Clear($bytes, 0, $bytes.Length)
[System.IO.File]::WriteAllBytes($path, $bytes)
Remove-Item $path -Force
```

---

## 8. Search & Find

```powershell
# Find files by name (wildcard)
Get-ChildItem C:\ -Recurse -Filter "*.ps1" -ErrorAction SilentlyContinue

# Find files by name pattern
Get-ChildItem C:\ -Recurse -Include "*config*","*settings*" -ErrorAction SilentlyContinue

# Find files containing a string (grep equivalent)
Get-ChildItem C:\Scripts -Recurse -Filter *.ps1 |
    Select-String -Pattern "password" |
    Select-Object Path, LineNumber, Line

# Case-sensitive search
Select-String -Path C:\file.txt -Pattern "Error" -CaseSensitive

# Search with regex
Get-ChildItem C:\Logs -Recurse -Filter *.log |
    Select-String -Pattern "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"

# Search multiple patterns
Select-String -Path C:\log.txt -Pattern "ERROR","FATAL","CRITICAL"

# Count matches per file
Get-ChildItem C:\Logs -Recurse -Filter *.log |
    Select-String -Pattern "ERROR" |
    Group-Object Path |
    Select-Object Name, Count |
    Sort-Object Count -Descending

# Find files by size (larger than 100MB)
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue |
    Where-Object Length -gt 100MB |
    Select-Object FullName, @{N='MB';E={[math]::Round($_.Length/1MB,2)}}

# Find files by extension
Get-ChildItem C:\ -Recurse -File |
    Group-Object Extension |
    Sort-Object Count -Descending |
    Select-Object -First 10 Name, Count

# Find duplicate files by name
Get-ChildItem C:\ -Recurse -File |
    Group-Object Name |
    Where-Object Count -gt 1 |
    Select-Object Name, Count

# Find duplicate files by hash (true duplicates)
Get-ChildItem C:\Documents -Recurse -File |
    Group-Object { (Get-FileHash $_.FullName -Algorithm MD5).Hash } |
    Where-Object Count -gt 1 |
    ForEach-Object { $_.Group | Select-Object FullName, Length }

# Find files with specific attributes
Get-ChildItem C:\ -Force | Where-Object { $_.Attributes -match "Hidden" }
Get-ChildItem C:\ -Force | Where-Object { $_.Attributes -match "ReadOnly" }
Get-ChildItem C:\ -Force | Where-Object { $_.Attributes -match "System" }

# Find recently accessed files
Get-ChildItem C:\Users -Recurse -File |
    Where-Object LastAccessTime -gt (Get-Date).AddHours(-1)
```

---

## 9. File Attributes & Metadata

```powershell
# View file properties
Get-Item C:\file.txt | Format-List *
Get-Item C:\file.txt | Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime, Attributes

# Get file size in different units
$f = Get-Item C:\file.txt
$f.Length                                      # bytes
[math]::Round($f.Length / 1KB, 2)             # KB
[math]::Round($f.Length / 1MB, 2)             # MB
[math]::Round($f.Length / 1GB, 4)             # GB

# Set file attributes
$f = Get-Item C:\file.txt
$f.Attributes = "ReadOnly"
$f.Attributes = "Hidden"
$f.Attributes = "ReadOnly,Hidden"
$f.Attributes = "Normal"                       # clear all special attributes

# Set ReadOnly via Set-ItemProperty
Set-ItemProperty C:\file.txt -Name IsReadOnly -Value $true
Set-ItemProperty C:\file.txt -Name IsReadOnly -Value $false

# Set timestamps
$f = Get-Item C:\file.txt
$f.CreationTime    = "2024-01-01 00:00:00"
$f.LastWriteTime   = Get-Date
$f.LastAccessTime  = Get-Date

# Get file version info (executables/DLLs)
(Get-Item "C:\Windows\System32\notepad.exe").VersionInfo
(Get-Item "C:\Windows\System32\notepad.exe").VersionInfo.FileVersion
(Get-Item "C:\Windows\System32\notepad.exe").VersionInfo.ProductName

# Get file owner
(Get-Acl C:\file.txt).Owner

# Get extended file attributes via Shell.Application (COM)
$shell  = New-Object -ComObject Shell.Application
$folder = $shell.Namespace("C:\Users\Avalon\Documents")
$file   = $folder.ParseName("document.docx")
0..320 | ForEach-Object {
    $name = $folder.GetDetailsOf($null, $_)
    $val  = $folder.GetDetailsOf($file, $_)
    if ($name -and $val) { "$name : $val" }
}
```

---

## 10. Permissions & ACLs

```powershell
# View ACL (Access Control List)
Get-Acl C:\file.txt
Get-Acl C:\MyFolder | Format-List

# View ACL as formatted table
(Get-Acl C:\file.txt).Access |
    Select-Object IdentityReference, FileSystemRights, AccessControlType, IsInherited

# Get owner
(Get-Acl C:\file.txt).Owner

# Copy ACL from one item to another
Get-Acl C:\source.txt | Set-Acl C:\destination.txt
Get-Acl C:\SourceFolder | Set-Acl C:\DestFolder

# Grant Full Control to a user
$acl  = Get-Acl C:\MyFolder
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "DOMAIN\username",
    "FullControl",
    "ContainerInherit,ObjectInherit",   # inheritance flags
    "None",                              # propagation flags
    "Allow"
)
$acl.AddAccessRule($rule)
Set-Acl C:\MyFolder $acl

# Deny read to a user
$acl  = Get-Acl C:\Secrets
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
    "DOMAIN\username", "Read", "Allow"
)
$acl.RemoveAccessRule($rule)
Set-Acl C:\Secrets $acl

# Remove a specific ACE
$acl  = Get-Acl C:\file.txt
$rule = ($acl.Access | Where-Object { $_.IdentityReference -eq "DOMAIN\username" })
$acl.RemoveAccessRule($rule)
Set-Acl C:\file.txt $acl

# Set owner
$acl = Get-Acl C:\file.txt
$acl.SetOwner([System.Security.Principal.NTAccount]"DOMAIN\newowner")
Set-Acl C:\file.txt $acl

# Disable inheritance and copy existing ACEs
$acl = Get-Acl C:\folder
$acl.SetAccessRuleProtection($true, $true)   # isProtected, preserveInheritance
Set-Acl C:\folder $acl

# Enable inheritance
$acl = Get-Acl C:\folder
$acl.SetAccessRuleProtection($false, $false)
Set-Acl C:\folder $acl

# FileSystemRights values:
#   FullControl, Modify, ReadAndExecute, Read, Write,
#   ListDirectory, CreateFiles, CreateDirectories, Delete,
#   ReadPermissions, ChangePermissions, TakeOwnership

# InheritanceFlags values:
#   None, ContainerInherit (subdirs), ObjectInherit (files)

# icacls via PowerShell
icacls "C:\MyFolder" /grant "DOMAIN\user:(OI)(CI)F"   # Full Control recursive
icacls "C:\file.txt" /deny  "Everyone:(W)"             # Deny write
icacls "C:\MyFolder" /reset /T                         # Reset to inherited
icacls "C:\MyFolder" /inheritance:r                    # Remove inherited ACEs
```

---

## 11. Symbolic Links & Junctions

```powershell
# Create a symbolic link to a file
New-Item -ItemType SymbolicLink -Path C:\link.txt -Target C:\original.txt

# Create a symbolic link to a directory
New-Item -ItemType SymbolicLink -Path C:\LinkDir -Target C:\OriginalDir

# Create a directory junction (Windows-specific, local paths only)
New-Item -ItemType Junction -Path C:\Junction -Target C:\OriginalDir

# Create a hard link (file only, same volume)
New-Item -ItemType HardLink -Path C:\hardlink.txt -Target C:\original.txt

# Create using cmd.exe mklink
cmd /c mklink "C:\link.txt" "C:\original.txt"           # file symlink
cmd /c mklink /D "C:\LinkDir" "C:\OriginalDir"          # dir symlink
cmd /c mklink /J "C:\Junction" "C:\OriginalDir"         # junction
cmd /c mklink /H "C:\hardlink.txt" "C:\original.txt"    # hard link

# List symlinks and junctions in a directory
Get-ChildItem C:\ -Force |
    Where-Object { $_.LinkType -in "SymbolicLink","Junction","HardLink" } |
    Select-Object Name, LinkType, Target

# Check if item is a symlink
(Get-Item C:\link.txt).LinkType            # "SymbolicLink" or $null
(Get-Item C:\link.txt).Target              # target path

# Remove a symlink (do NOT use -Recurse — it would delete target contents)
Remove-Item C:\LinkDir                     # removes link, not target
(Get-Item C:\LinkDir).Delete()             # alternative
```

---

## 12. Drives & Providers

```powershell
# List all PS drives (filesystem, registry, env, etc.)
Get-PSDrive

# List filesystem drives only
Get-PSDrive -PSProvider FileSystem

# Map a network drive
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\server\share"
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\server\share" -Persist   # survives session
New-PSDrive -Name Z -PSProvider FileSystem -Root "\\server\share" -Credential (Get-Credential) -Persist

# Remove a mapped drive
Remove-PSDrive -Name Z

# Net use (traditional mapping)
net use Z: \\server\share /persistent:yes
net use Z: /delete

# Access environment variables via provider
Get-Item Env:PATH
Set-Item Env:MYVAR "value"
$env:MYVAR = "value"

# List all environment variables
Get-ChildItem Env:

# Disk space info
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Root, Used, Free,
    @{N='Used (GB)'; E={[math]::Round($_.Used/1GB,2)}},
    @{N='Free (GB)'; E={[math]::Round($_.Free/1GB,2)}}

# Disk info via WMI
Get-WmiObject Win32_LogicalDisk |
    Select-Object DeviceID, VolumeName, FileSystem,
        @{N='Size GB';  E={[math]::Round($_.Size/1GB,2)}},
        @{N='Free GB';  E={[math]::Round($_.FreeSpace/1GB,2)}},
        @{N='Free %';   E={[math]::Round($_.FreeSpace/$_.Size*100,1)}}

# CIM alternative
Get-CimInstance Win32_LogicalDisk |
    Select-Object DeviceID, VolumeName,
        @{N='Free GB'; E={[math]::Round($_.FreeSpace/1GB,2)}}
```

---

## 13. Compression & Archives

```powershell
# Compress files into a ZIP
Compress-Archive -Path C:\MyFolder -DestinationPath C:\archive.zip

# Compress multiple items
Compress-Archive -Path C:\file1.txt, C:\file2.txt -DestinationPath C:\archive.zip

# Compress with wildcard
Compress-Archive -Path C:\Logs\*.log -DestinationPath C:\logs-backup.zip

# Update existing ZIP (add/update files)
Compress-Archive -Path C:\newfile.txt -DestinationPath C:\archive.zip -Update

# Overwrite existing ZIP
Compress-Archive -Path C:\folder -DestinationPath C:\archive.zip -Force

# Extract ZIP
Expand-Archive -Path C:\archive.zip -DestinationPath C:\extracted

# Extract and overwrite existing files
Expand-Archive -Path C:\archive.zip -DestinationPath C:\extracted -Force

# Extract ZIP contents using .NET (progress, filter)
Add-Type -AssemblyName System.IO.Compression.FileSystem
$zip = [System.IO.Compression.ZipFile]::OpenRead("C:\archive.zip")
$zip.Entries | ForEach-Object { Write-Host $_.FullName }  # list contents
$zip.Dispose()

# Extract specific file from ZIP
Add-Type -AssemblyName System.IO.Compression.FileSystem
$zip   = [System.IO.Compression.ZipFile]::OpenRead("C:\archive.zip")
$entry = $zip.Entries | Where-Object Name -eq "target.txt"
[System.IO.Compression.ZipFileExtensions]::ExtractToFile($entry, "C:\target.txt", $true)
$zip.Dispose()
```

---

## 14. Hashing & Integrity

```powershell
# Get file hash (default: SHA256)
Get-FileHash C:\file.txt
Get-FileHash C:\file.txt -Algorithm SHA256

# Other algorithms
Get-FileHash C:\file.txt -Algorithm MD5
Get-FileHash C:\file.txt -Algorithm SHA1
Get-FileHash C:\file.txt -Algorithm SHA384
Get-FileHash C:\file.txt -Algorithm SHA512

# Hash just the value
(Get-FileHash C:\file.txt -Algorithm MD5).Hash

# Hash multiple files
Get-ChildItem C:\Logs -Filter *.log | Get-FileHash | Select-Object Hash, Path

# Verify file integrity (compare expected vs actual)
$expected = "ABC123DEF456..."
$actual   = (Get-FileHash C:\downloaded.iso -Algorithm SHA256).Hash
if ($expected -eq $actual) { "MATCH — file is intact" } else { "MISMATCH — file is corrupt" }

# Hash a string (not a file)
$stream = [System.IO.MemoryStream]::new([System.Text.Encoding]::UTF8.GetBytes("Hello"))
(Get-FileHash -InputStream $stream -Algorithm SHA256).Hash

# Find files with a specific hash (malware hunt / duplicate find)
$targetHash = "DEADBEEF..."
Get-ChildItem C:\ -Recurse -File -ErrorAction SilentlyContinue |
    ForEach-Object {
        if ((Get-FileHash $_.FullName -Algorithm MD5).Hash -eq $targetHash) {
            $_.FullName
        }
    }
```

---

## 15. Streams (ADS)

Alternate Data Streams (ADS) are a Windows NTFS feature that can hide data alongside a file.

```powershell
# List all streams on a file
Get-Item C:\file.txt -Stream *

# Read a specific stream
Get-Content C:\file.txt -Stream mystream

# Write to a named stream
Set-Content C:\file.txt -Stream mystream -Value "Hidden data"

# Read the Zone.Identifier stream (added when downloading files)
Get-Content C:\downloaded.exe -Stream Zone.Identifier

# Zone.Identifier ZoneId values:
#   0 = My Computer
#   1 = Local Intranet
#   2 = Trusted Sites
#   3 = Internet
#   4 = Restricted Sites

# Unblock a downloaded file (remove Zone.Identifier)
Unblock-File C:\downloaded.zip
# or manually:
Remove-Item C:\downloaded.zip -Stream Zone.Identifier

# Unblock all files in a directory
Get-ChildItem C:\Downloads -Recurse -File | Unblock-File

# Create a hidden stream on a directory
Set-Content C:\MyFolder -Stream secret -Value "Stashed data"
Get-Content C:\MyFolder -Stream secret

# List all files in a folder with ADS (beyond just the default data stream)
Get-ChildItem C:\Folder -Recurse |
    ForEach-Object { Get-Item $_.FullName -Stream * } |
    Where-Object Stream -ne ':$DATA'
```

---

## 16. Monitoring & Watching

```powershell
# Watch a directory for any changes (FileSystemWatcher)
$watcher = New-Object System.IO.FileSystemWatcher
$watcher.Path   = "C:\WatchDir"
$watcher.Filter = "*.*"
$watcher.IncludeSubdirectories = $true
$watcher.EnableRaisingEvents   = $true

# Define actions
$action = {
    $e    = $Event.SourceEventArgs
    $type = $Event.SourceEventArgs.ChangeType
    Write-Host "[$type] $($e.FullPath) at $(Get-Date -Format 'HH:mm:ss')"
}

# Register events
Register-ObjectEvent $watcher "Created" -Action $action | Out-Null
Register-ObjectEvent $watcher "Changed" -Action $action | Out-Null
Register-ObjectEvent $watcher "Deleted" -Action $action | Out-Null
Register-ObjectEvent $watcher "Renamed" -Action $action | Out-Null

# Watch interactively (Ctrl+C to stop)
Write-Host "Watching C:\WatchDir... Press Ctrl+C to stop"
try { while ($true) { Start-Sleep 1 } }
finally {
    $watcher.Dispose()
    Get-EventSubscriber | Unregister-Event
}

# Tail a log file (poll method)
Get-Content C:\Logs\app.log -Tail 0 -Wait

# Watch for a specific file to appear
while (-not (Test-Path "C:\trigger.txt")) { Start-Sleep 2 }
Write-Host "trigger.txt appeared!"
```

---

## 17. .NET System.IO Methods

```powershell
# File operations
[System.IO.File]::Exists("C:\file.txt")
[System.IO.File]::ReadAllText("C:\file.txt")
[System.IO.File]::ReadAllLines("C:\file.txt")
[System.IO.File]::ReadAllBytes("C:\file.bin")
[System.IO.File]::WriteAllText("C:\file.txt", "content")
[System.IO.File]::WriteAllLines("C:\file.txt", @("line1","line2"))
[System.IO.File]::WriteAllBytes("C:\file.bin", $bytes)
[System.IO.File]::AppendAllText("C:\file.txt", "more")
[System.IO.File]::Copy("C:\src.txt", "C:\dst.txt", $true)    # $true = overwrite
[System.IO.File]::Move("C:\src.txt", "C:\dst.txt")
[System.IO.File]::Delete("C:\file.txt")
[System.IO.File]::GetAttributes("C:\file.txt")
[System.IO.File]::SetAttributes("C:\file.txt", "Hidden")
[System.IO.File]::GetCreationTime("C:\file.txt")
[System.IO.File]::SetLastWriteTime("C:\file.txt", (Get-Date))

# Directory operations
[System.IO.Directory]::Exists("C:\dir")
[System.IO.Directory]::CreateDirectory("C:\dir\subdir")
[System.IO.Directory]::Delete("C:\dir", $true)        # $true = recursive
[System.IO.Directory]::Move("C:\src", "C:\dst")
[System.IO.Directory]::GetFiles("C:\dir", "*.txt", [System.IO.SearchOption]::AllDirectories)
[System.IO.Directory]::GetDirectories("C:\dir")
[System.IO.Directory]::GetParent("C:\dir\sub")

# Path operations
[System.IO.Path]::Combine("C:\Users", "Avalon", "file.txt")
[System.IO.Path]::GetFileName("C:\dir\file.txt")          # file.txt
[System.IO.Path]::GetFileNameWithoutExtension("file.txt")  # file
[System.IO.Path]::GetExtension("file.txt")                 # .txt
[System.IO.Path]::GetDirectoryName("C:\dir\file.txt")      # C:\dir
[System.IO.Path]::GetFullPath(".\relative\path")
[System.IO.Path]::GetTempFileName()                         # creates temp file, returns path
[System.IO.Path]::GetTempPath()                             # %TEMP% directory
[System.IO.Path]::GetRandomFileName()                       # random filename (no creation)
[System.IO.Path]::IsPathRooted("C:\abs")                   # True
[System.IO.Path]::InvalidPathChars                          # chars not allowed in paths
```

---

## 18. cmd.exe Equivalents

| Task                          | PowerShell                                          | cmd.exe                             |
|-------------------------------|-----------------------------------------------------|-------------------------------------|
| Show current directory        | `Get-Location` / `$PWD`                             | `cd` / `echo %CD%`                  |
| Change directory              | `Set-Location C:\path`                              | `cd C:\path`                        |
| List files                    | `Get-ChildItem`                                     | `dir`                               |
| List hidden files             | `Get-ChildItem -Force`                              | `dir /a`                            |
| Recursive list                | `Get-ChildItem -Recurse`                            | `dir /s`                            |
| Create directory              | `New-Item -ItemType Directory`                      | `mkdir` / `md`                      |
| Create file                   | `New-Item -ItemType File`                           | `type nul > file.txt`               |
| Copy file                     | `Copy-Item src dst`                                 | `copy src dst`                      |
| Copy directory                | `Copy-Item src dst -Recurse`                        | `xcopy src dst /E /I`               |
| Move file                     | `Move-Item src dst`                                 | `move src dst`                      |
| Rename                        | `Rename-Item old new`                               | `ren old new`                       |
| Delete file                   | `Remove-Item file`                                  | `del file`                          |
| Delete directory              | `Remove-Item dir -Recurse -Force`                   | `rmdir /s /q dir`                   |
| Read file                     | `Get-Content file`                                  | `type file`                         |
| Find string in file           | `Select-String -Pattern "text" -Path file`          | `findstr "text" file`               |
| Find files                    | `Get-ChildItem -Recurse -Filter name`               | `dir /s /b name`                    |
| Disk space                    | `Get-PSDrive -PSProvider FileSystem`                | `dir C:\`                           |
| Map network drive             | `New-PSDrive -Persist ...`                          | `net use Z: \\server\share`         |
| Symbolic link                 | `New-Item -ItemType SymbolicLink`                   | `mklink`                            |
| File hash                     | `Get-FileHash file`                                 | `certutil -hashfile file SHA256`    |
| Compress to ZIP               | `Compress-Archive`                                  | *(no built-in)*                     |
| Extract ZIP                   | `Expand-Archive`                                    | *(no built-in)*                     |

---

## 19. Quick-Reference Table

| Cmdlet / Method             | Purpose                                             |
|-----------------------------|-----------------------------------------------------|
| `Get-Location`              | Current directory                                   |
| `Set-Location`              | Change directory                                    |
| `Push-Location`/`Pop-Location` | Directory stack navigation                       |
| `Resolve-Path`              | Resolve relative to absolute path                   |
| `Split-Path`                | Extract parent, filename, or extension              |
| `Join-Path`                 | Build a path from components                        |
| `Test-Path`                 | Check if path exists                                |
| `Get-ChildItem`             | List files and directories                          |
| `Get-Item`                  | Get a single item's properties                      |
| `New-Item`                  | Create file, directory, symlink, junction           |
| `Copy-Item`                 | Copy files or directories                           |
| `Move-Item`                 | Move files or directories                           |
| `Rename-Item`               | Rename files or directories                         |
| `Remove-Item`               | Delete files or directories                         |
| `Get-Content`               | Read file contents                                  |
| `Set-Content`               | Write (overwrite) file contents                     |
| `Add-Content`               | Append to file                                      |
| `Out-File`                  | Redirect formatted output to file                   |
| `Select-String`             | Search file contents (grep equivalent)              |
| `Get-Acl` / `Set-Acl`      | Read / write file permissions                       |
| `Get-FileHash`              | Compute file hash (MD5, SHA256, etc.)               |
| `Compress-Archive`          | Create ZIP archive                                  |
| `Expand-Archive`            | Extract ZIP archive                                 |
| `Unblock-File`              | Remove Zone.Identifier (downloaded file mark)       |
| `Get-PSDrive`               | List drives and providers                           |
| `New-PSDrive`               | Map a network drive or create a PS drive            |
| `Import-Csv`                | Read CSV file into objects                          |
| `Export-Csv`                | Write objects to CSV                                |
| `ConvertFrom-Json`          | Parse JSON string to object                         |
| `ConvertTo-Json`            | Serialize object to JSON                            |

---

### Common Path Variables

| Variable               | Expands To                                      |
|------------------------|-------------------------------------------------|
| `$HOME`                | Current user's home directory                   |
| `$env:USERPROFILE`     | Same as `$HOME` (Windows)                       |
| `$env:TEMP`            | Temporary files directory                       |
| `$env:SystemRoot`      | `C:\Windows`                                    |
| `$env:ProgramFiles`    | `C:\Program Files`                              |
| `$env:ProgramData`     | `C:\ProgramData`                                |
| `$env:APPDATA`         | `%USERPROFILE%\AppData\Roaming`                 |
| `$env:LOCALAPPDATA`    | `%USERPROFILE%\AppData\Local`                   |
| `$PSScriptRoot`        | Directory of the currently running script       |
| `$PSCommandPath`       | Full path of the currently running script       |

---

*Generated 2026-05-08 — Windows PowerShell 5.1 / PowerShell 7.x*
