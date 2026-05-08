# PowerShell Services Cheat Sheet

> Windows Service management via `Microsoft.PowerShell.Management` (built-in) and `sc.exe` equivalents

---

## Table of Contents

1. [Query Services](#1-query-services)
2. [Start / Stop / Restart / Pause](#2-start--stop--restart--pause)
3. [Enable & Disable Services](#3-enable--disable-services)
4. [Create & Delete Services](#4-create--delete-services)
5. [Modify Service Properties](#5-modify-service-properties)
6. [Service Recovery / Failure Actions](#6-service-recovery--failure-actions)
7. [Service Permissions (SDDL)](#7-service-permissions-sddl)
8. [Service Dependencies](#8-service-dependencies)
9. [Remote Service Management](#9-remote-service-management)
10. [WMI / CIM Alternative Queries](#10-wmi--cim-alternative-queries)
11. [Logging & Diagnostics](#11-logging--diagnostics)
12. [sc.exe Equivalents](#12-scexe-equivalents)
13. [Quick-Reference Table](#13-quick-reference-table)

---

## 1. Query Services

```powershell
# List all services
Get-Service

# Get a specific service by name
Get-Service -Name wuauserv

# Get by display name (wildcard)
Get-Service -DisplayName "*Windows Update*"

# Multiple services at once
Get-Service -Name wuauserv, spooler, bits

# Filter by status
Get-Service | Where-Object Status -eq Running
Get-Service | Where-Object Status -eq Stopped

# Show only automatically-starting services that are stopped (broken auto-starts)
Get-Service |
    Where-Object { $_.StartType -eq 'Automatic' -and $_.Status -eq 'Stopped' }

# Show service start type
Get-Service | Select-Object Name, DisplayName, Status, StartType | Sort-Object StartType

# Show service with process ID
Get-Service | Where-Object Status -eq Running |
    Select-Object Name, DisplayName, Status,
        @{N='PID';E={(Get-WmiObject Win32_Service -Filter "Name='$($_.Name)'").ProcessId}}

# Show services grouped by status
Get-Service | Group-Object Status

# Show services grouped by start type
Get-Service | Group-Object StartType | Select-Object Name, Count

# Count running vs stopped
Get-Service | Group-Object Status | Select-Object Name, Count

# Check if a specific service exists
[bool](Get-Service -Name "nonexistent" -ErrorAction SilentlyContinue)

# Find services running as a specific account
Get-WmiObject Win32_Service | Where-Object StartName -like "*LocalSystem*" |
    Select-Object Name, DisplayName, StartName, State

# Find services NOT running as LocalSystem
Get-WmiObject Win32_Service |
    Where-Object { $_.StartName -notlike "LocalSystem" -and $_.StartName -ne $null } |
    Select-Object Name, DisplayName, StartName, State | Format-Table -AutoSize
```

---

## 2. Start / Stop / Restart / Pause

```powershell
# Start a service
Start-Service -Name spooler
Start-Service -DisplayName "Print Spooler"

# Stop a service
Stop-Service -Name spooler

# Stop forcefully (kills dependent services too)
Stop-Service -Name spooler -Force

# Restart a service
Restart-Service -Name spooler

# Restart and wait for completion
Restart-Service -Name spooler -PassThru | Wait-Service

# Pause a service (only if it supports pause)
Suspend-Service -Name spooler

# Resume a paused service
Resume-Service -Name spooler

# Start multiple services
"wuauserv","bits","cryptsvc" | Start-Service

# Stop all services matching a pattern
Get-Service -DisplayName "*Adobe*" | Stop-Service -Force

# Wait for service to reach a specific status (poll up to 30s)
$svc = Get-Service -Name spooler
$svc.WaitForStatus('Running', '00:00:30')

# Start service and confirm it reached Running state
Start-Service -Name wuauserv -PassThru | Select-Object Name, Status

# Stop service with timeout handling
try {
    Stop-Service -Name spooler -NoWait
    $svc = Get-Service -Name spooler
    $svc.WaitForStatus('Stopped', [TimeSpan]::FromSeconds(30))
    Write-Host "Service stopped."
} catch {
    Write-Warning "Timed out waiting for service to stop."
}
```

---

## 3. Enable & Disable Services

```powershell
# Set startup type to Automatic
Set-Service -Name wuauserv -StartupType Automatic

# Set startup type to Automatic (Delayed)
Set-Service -Name wuauserv -StartupType AutomaticDelayedStart

# Set startup type to Manual
Set-Service -Name spooler -StartupType Manual

# Disable a service (also stops it if running)
Set-Service -Name spooler -StartupType Disabled
Stop-Service -Name spooler -Force

# Enable and start in one go
Set-Service -Name spooler -StartupType Automatic
Start-Service -Name spooler

# Disable a list of services
"XblAuthManager","XblGameSave","XboxNetApiSvc" | ForEach-Object {
    Stop-Service -Name $_ -Force -ErrorAction SilentlyContinue
    Set-Service  -Name $_ -StartupType Disabled
}

# StartupType values:
#   Automatic            — starts at boot
#   AutomaticDelayedStart — starts shortly after boot (reduces boot time)
#   Manual               — starts on demand
#   Disabled             — cannot be started
#   Boot                 — kernel-level driver (use sc.exe for this)
#   System               — loaded by I/O subsystem (use sc.exe for this)
```

---

## 4. Create & Delete Services

```powershell
# Create a new service (basic)
New-Service `
    -Name        "MyService" `
    -DisplayName "My Custom Service" `
    -BinaryPathName "C:\MyApp\myapp.exe" `
    -StartupType Automatic `
    -Description "Runs MyApp as a background service"

# Create with a specific service account
New-Service `
    -Name           "MyService" `
    -BinaryPathName "C:\MyApp\myapp.exe" `
    -Credential     (Get-Credential)

# Create with dependencies (service must start after listed services)
New-Service `
    -Name           "MyService" `
    -BinaryPathName "C:\MyApp\myapp.exe" `
    -DependsOn      "LanmanServer", "Tcpip"

# Create using sc.exe (more options, e.g. kernel drivers)
sc.exe create MySvc binPath= "C:\MyApp\myapp.exe" start= auto DisplayName= "My Svc"

# Delete a service (PS 6+ / PS 7)
Remove-Service -Name "MyService"

# Delete a service (Windows PowerShell 5.1 — no Remove-Service)
sc.exe delete MyService

# Delete via WMI (5.1 compatible)
(Get-WmiObject Win32_Service -Filter "Name='MyService'").Delete()
```

---

## 5. Modify Service Properties

```powershell
# Change display name and description
Set-Service -Name MyService -DisplayName "My Updated Service" -Description "New description"

# Change the binary path (sc.exe required — Set-Service lacks this)
sc.exe config MyService binPath= "C:\NewPath\myapp.exe"

# Change the service account to LocalSystem
sc.exe config MyService obj= LocalSystem

# Change the service account to Network Service
sc.exe config MyService obj= "NT AUTHORITY\NetworkService" password= ""

# Change to a named user account
sc.exe config MyService obj= "DOMAIN\svcaccount" password= "P@ssw0rd"

# Change to LocalService
sc.exe config MyService obj= "NT AUTHORITY\LocalService" password= ""

# Change error control level
sc.exe config MyService error= normal   # normal | severe | critical | ignore

# Change service type
sc.exe config MyService type= own       # own | share | interact | kernel | filesys | rec

# View current config via sc.exe
sc.exe qc MyService

# Verify change
Get-Service -Name MyService | Format-List *
```

---

## 6. Service Recovery / Failure Actions

Recovery actions are configured with `sc.exe failure` (PowerShell has no native cmdlet for this).

```powershell
# Set failure actions: restart on 1st and 2nd failure, reboot on 3rd
sc.exe failure MyService `
    reset= 86400 `           # reset failure count after 86400 seconds (1 day)
    actions= restart/5000/restart/10000/reboot/0
#   format: action/delay_ms  (restart | reboot | run | "")

# Set to restart service after 5s on every failure
sc.exe failure MyService reset= 60 actions= restart/5000/restart/5000/restart/5000

# Enable failure actions even on clean exit (non-zero exit code NOT required)
sc.exe failureflag MyService 1    # 1 = trigger on any stop; 0 = only on crash

# View current failure configuration
sc.exe qfailure MyService

# Run a custom command on failure
sc.exe failure MyService reset= 3600 command= "C:\scripts\alert.bat" actions= run/0

# Disable all failure actions
sc.exe failure MyService reset= 0 actions= ""
```

---

## 7. Service Permissions (SDDL)

```powershell
# View current SDDL for a service
sc.exe sdshow MyService

# Grant a user permission to start/stop a service (modifying SDDL)
# Get current SDDL first, then append an ACE
$sddl = (sc.exe sdshow spooler) -join ""

# Common SDDL permission bits for services:
#   CC = SERVICE_QUERY_CONFIG      RP = SERVICE_START
#   LC = SERVICE_QUERY_STATUS      WP = SERVICE_STOP
#   SW = SERVICE_ENUMERATE_DEPENDENTS
#   LO = SERVICE_INTERROGATE
#   CR = SERVICE_USER_DEFINED_CONTROL
#   RC = READ_CONTROL               WD = WRITE_DAC
#   GA = GENERIC_ALL

# Set new SDDL (example: grant NETWORK SERVICE start/stop)
sc.exe sdset MyService "D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;RPWPCR;;;IU)"

# Use PowerShell to read service permissions via .NET
$svc = Get-Service -Name spooler
$sd  = (New-Object System.ServiceProcess.ServiceController($svc.Name)).GetType()
# For full ACL editing, use the SetSD method via sc.exe or secedit
```

---

## 8. Service Dependencies

```powershell
# Show what a service depends on
Get-Service -Name spooler | Select-Object -ExpandProperty ServicesDependedOn

# Show services that depend ON this service (dependents)
Get-Service -Name RpcSs | Select-Object -ExpandProperty DependentServices

# Full dependency tree (recursive)
function Get-ServiceDependencyTree {
    param([string]$Name, [int]$Depth = 0)
    $svc = Get-Service -Name $Name
    Write-Host ("  " * $Depth + "-> " + $svc.DisplayName + " [$($svc.Status)]")
    $svc.ServicesDependedOn | ForEach-Object {
        Get-ServiceDependencyTree -Name $_.Name -Depth ($Depth + 1)
    }
}
Get-ServiceDependencyTree -Name "spooler"

# Stop a service and all its dependents
Stop-Service -Name LanmanServer -Force   # -Force stops dependents automatically

# Add a dependency (sc.exe required)
sc.exe config MyService depend= LanmanServer/Tcpip

# Remove all dependencies
sc.exe config MyService depend= ""
```

---

## 9. Remote Service Management

```powershell
# Get services on a remote machine
Get-Service -ComputerName RemoteServer

# Start/stop service on remote machine
Start-Service  -InputObject (Get-Service -Name spooler -ComputerName RemoteServer)
Stop-Service   -InputObject (Get-Service -Name spooler -ComputerName RemoteServer)

# Restart a service on multiple remote machines
$servers = "Server01","Server02","Server03"
Invoke-Command -ComputerName $servers -ScriptBlock {
    Restart-Service -Name wuauserv -Force
    Get-Service -Name wuauserv | Select-Object Name, Status
}

# Create a service on a remote machine
Invoke-Command -ComputerName RemoteServer -ScriptBlock {
    New-Service -Name "MyService" -BinaryPathName "C:\MyApp\myapp.exe" -StartupType Automatic
}

# Check for stopped auto-start services across multiple servers
$servers = "Server01","Server02","Server03"
Invoke-Command -ComputerName $servers -ScriptBlock {
    Get-Service | Where-Object { $_.StartType -eq 'Automatic' -and $_.Status -eq 'Stopped' } |
        Select-Object Name, DisplayName, Status, @{N='Server';E={$env:COMPUTERNAME}}
} | Format-Table -AutoSize

# WMI-based remote service query (no WinRM needed, uses DCOM)
Get-WmiObject -Class Win32_Service -ComputerName RemoteServer |
    Where-Object State -eq "Running" |
    Select-Object Name, DisplayName, StartName
```

---

## 10. WMI / CIM Alternative Queries

CIM cmdlets are the modern replacement for WMI and work over WS-Man (firewall-friendly).

```powershell
# List all services via CIM
Get-CimInstance -ClassName Win32_Service

# Filter by state
Get-CimInstance -ClassName Win32_Service -Filter "State='Running'"
Get-CimInstance -ClassName Win32_Service -Filter "State='Stopped' AND StartMode='Auto'"

# Get service with PID and account info
Get-CimInstance Win32_Service | Select-Object Name, DisplayName, State, ProcessId, StartName, StartMode

# Find service by PID
Get-CimInstance Win32_Service | Where-Object ProcessId -eq 1234

# Find the process behind a service
$svc = Get-CimInstance Win32_Service -Filter "Name='spooler'"
Get-Process -Id $svc.ProcessId

# Start/stop via CIM method
$svc = Get-CimInstance -ClassName Win32_Service -Filter "Name='spooler'"
Invoke-CimMethod -InputObject $svc -MethodName StartService
Invoke-CimMethod -InputObject $svc -MethodName StopService

# Remote CIM query (WS-Man, no DCOM)
$session = New-CimSession -ComputerName RemoteServer
Get-CimInstance -CimSession $session -ClassName Win32_Service | Where-Object State -eq Stopped
Remove-CimSession $session

# Get services that share a process (svchost-hosted)
Get-CimInstance Win32_Service |
    Group-Object ProcessId |
    Where-Object Count -gt 1 |
    ForEach-Object {
        [PSCustomObject]@{
            PID      = $_.Name
            Services = ($_.Group.Name -join ", ")
        }
    }
```

---

## 11. Logging & Diagnostics

```powershell
# View System event log for service failures (Event ID 7034 = crashed, 7036 = state change)
Get-WinEvent -LogName System -FilterHashtable @{Id=7034,7036} -MaxEvents 30 |
    Select-Object TimeCreated, Id, Message | Format-List

# Filter service crash events only (7034)
Get-WinEvent -LogName System | Where-Object Id -eq 7034 |
    Select-Object TimeCreated, Message | Format-List

# Service control manager events
Get-WinEvent -ProviderName "Service Control Manager" -MaxEvents 50 |
    Select-Object TimeCreated, Id, Message | Format-Table -AutoSize -Wrap

# Check if a service process is hung (not responding)
$svc = Get-WmiObject Win32_Service -Filter "Name='spooler'"
$proc = Get-Process -Id $svc.ProcessId -ErrorAction SilentlyContinue
if ($proc) { $proc.Responding } else { "Process not found" }

# Show service uptime (time since process started)
$svc = Get-WmiObject Win32_Service -Filter "Name='spooler'"
$proc = Get-Process -Id $svc.ProcessId
$uptime = (Get-Date) - $proc.StartTime
"$($svc.Name) has been running for: $($uptime.ToString('dd\d\ hh\h\ mm\m'))"

# Log all service state changes to a file (run as scheduled task or in background)
Register-WmiEvent -Class "Win32_ServiceStopEvent" -Action {
    $e = $EventArgs.NewEvent
    Add-Content -Path "C:\logs\service-stops.log" -Value "$(Get-Date) - $($e.SourceName) stopped"
}

# Find services with missing or invalid binaries
Get-WmiObject Win32_Service | ForEach-Object {
    $path = ($_.PathName -replace '"','') -split ' ' | Select-Object -First 1
    if ($path -and !(Test-Path $path)) {
        [PSCustomObject]@{ Name = $_.Name; MissingPath = $path }
    }
} | Format-Table -AutoSize

# Audit services not running as standard accounts (potential persistence)
Get-WmiObject Win32_Service |
    Where-Object {
        $_.StartName -notin @(
            "LocalSystem","NT AUTHORITY\LocalService",
            "NT AUTHORITY\NetworkService","NT AUTHORITY\System"
        ) -and $_.StartName -ne $null
    } |
    Select-Object Name, DisplayName, StartName, State | Format-Table -AutoSize
```

---

## 12. sc.exe Equivalents

| Task                          | PowerShell                                        | sc.exe                                                  |
|-------------------------------|---------------------------------------------------|---------------------------------------------------------|
| List all services             | `Get-Service`                                     | `sc.exe query type= all state= all`                     |
| Query one service             | `Get-Service -Name spooler`                       | `sc.exe query spooler`                                  |
| Start service                 | `Start-Service spooler`                           | `sc.exe start spooler`                                  |
| Stop service                  | `Stop-Service spooler`                            | `sc.exe stop spooler`                                   |
| Restart service               | `Restart-Service spooler`                         | `sc.exe stop spooler && sc.exe start spooler`           |
| Pause service                 | `Suspend-Service spooler`                         | `sc.exe pause spooler`                                  |
| Resume service                | `Resume-Service spooler`                          | `sc.exe continue spooler`                               |
| Create service                | `New-Service -Name ... -BinaryPathName ...`       | `sc.exe create MySvc binPath= "..." start= auto`        |
| Delete service                | `Remove-Service -Name ...` (PS7) / `sc.exe delete`| `sc.exe delete MySvc`                                   |
| Change start type             | `Set-Service -StartupType Automatic`              | `sc.exe config MySvc start= auto`                       |
| Disable service               | `Set-Service -StartupType Disabled`               | `sc.exe config MySvc start= disabled`                   |
| Change binary path            | *(not supported)*                                 | `sc.exe config MySvc binPath= "C:\new\path.exe"`        |
| Change account                | *(not supported)*                                 | `sc.exe config MySvc obj= "DOMAIN\user" password= "..."` |
| Set failure actions           | *(not supported)*                                 | `sc.exe failure MySvc reset= 86400 actions= restart/5000` |
| View failure config           | *(not supported)*                                 | `sc.exe qfailure MySvc`                                 |
| View full config              | *(partial via Get-Service)*                       | `sc.exe qc MySvc`                                       |
| View/set SDDL                 | *(not supported)*                                 | `sc.exe sdshow MySvc` / `sc.exe sdset MySvc "..."`      |

---

## 13. Quick-Reference Table

| Cmdlet                  | Purpose                                              |
|-------------------------|------------------------------------------------------|
| `Get-Service`           | Query services (local or remote)                     |
| `Start-Service`         | Start a stopped service                              |
| `Stop-Service`          | Stop a running service                               |
| `Restart-Service`       | Stop then start a service                            |
| `Suspend-Service`       | Pause a running service                              |
| `Resume-Service`        | Resume a paused service                              |
| `Set-Service`           | Change display name, description, or startup type    |
| `New-Service`           | Create a new Windows service                         |
| `Remove-Service`        | Delete a service (PowerShell 6+)                     |
| `Get-WmiObject Win32_Service` | Detailed query: PID, account, path           |
| `Get-CimInstance Win32_Service` | Modern WMI alternative (WS-Man transport)  |
| `Invoke-CimMethod`      | Call WMI methods (Start/Stop) programmatically       |
| `Get-WinEvent`          | Read service-related events from System log          |

### StartupType Values

| Value                   | Meaning                                              |
|-------------------------|------------------------------------------------------|
| `Automatic`             | Starts at boot                                       |
| `AutomaticDelayedStart` | Starts shortly after boot (reduces boot time)        |
| `Manual`                | Starts on demand only                                |
| `Disabled`              | Cannot be started                                    |
| `Boot`                  | Kernel driver — loaded by boot loader                |
| `System`                | Kernel driver — loaded by I/O subsystem              |

### Common sc.exe start= Values

| sc.exe value | Equivalent StartupType         |
|--------------|-------------------------------|
| `auto`       | `Automatic`                   |
| `delayed-auto` | `AutomaticDelayedStart`     |
| `demand`     | `Manual`                      |
| `disabled`   | `Disabled`                    |
| `boot`       | `Boot`                        |
| `system`     | `System`                      |

---

*Generated 2026-05-08 — Windows PowerShell 5.1 / PowerShell 7.x*
