# PowerShell Firewall Cheat Sheet

> Windows Defender Firewall via `NetSecurity` module (built-in, no install needed)

---

## Table of Contents

1. [Query Rules](#1-query-rules)
2. [Create Rules](#2-create-rules)
3. [Modify Rules](#3-modify-rules)
4. [Enable / Disable / Delete Rules](#4-enable--disable--delete-rules)
5. [Firewall Profiles](#5-firewall-profiles)
6. [Port & Address Filters](#6-port--address-filters)
7. [Application Rules](#7-application-rules)
8. [IPsec / Advanced Security](#8-ipsec--advanced-security)
9. [Logging & Diagnostics](#9-logging--diagnostics)
10. [Export & Import](#10-export--import)
11. [netsh Equivalents](#11-netsh-equivalents)
12. [Quick-Reference Table](#12-quick-reference-table)

---

## Disable all profiles at once
```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

## 1. Query Rules

```powershell
# List all rules
Get-NetFirewallRule

# Filter by display name (wildcard supported)
Get-NetFirewallRule -DisplayName "*RDP*"

# Filter by direction and action
Get-NetFirewallRule -Direction Inbound -Action Allow

# Filter by profile
Get-NetFirewallRule -Profile Domain

# Show enabled rules only
Get-NetFirewallRule | Where-Object Enabled -eq True

# Show rules with their port/address details (join filter objects)
Get-NetFirewallRule -DisplayName "*HTTP*" |
    Get-NetFirewallPortFilter |
    Select-Object LocalPort, RemotePort, Protocol

# Full rule detail pipeline
Get-NetFirewallRule -DisplayName "MyRule" | Format-List *

# Find rules for a specific port
Get-NetFirewallPortFilter | Where-Object LocalPort -eq 443 |
    Get-NetFirewallRule

# Find rules for a specific application
Get-NetFirewallApplicationFilter | Where-Object Program -like "*chrome*" |
    Get-NetFirewallRule

# Find rules for a specific remote address
Get-NetFirewallAddressFilter | Where-Object RemoteAddress -eq "10.0.0.5" |
    Get-NetFirewallRule

# Count rules by action
Get-NetFirewallRule | Group-Object Action | Select-Object Name, Count
```

---

## 2. Create Rules

### Basic inbound allow (TCP port)
```powershell
New-NetFirewallRule `
    -DisplayName "Allow HTTPS Inbound" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 443 `
    -Action Allow
```

### Inbound allow with profile scope
```powershell
New-NetFirewallRule `
    -DisplayName "Allow SSH - Private" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 22 `
    -Profile Private `
    -Action Allow
```

### Allow from specific remote IP
```powershell
New-NetFirewallRule `
    -DisplayName "Allow WinRM from Admin Host" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 5985 `
    -RemoteAddress 192.168.1.10 `
    -Action Allow
```

### Allow from subnet / CIDR
```powershell
New-NetFirewallRule `
    -DisplayName "Allow Internal SMB" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 445 `
    -RemoteAddress 10.0.0.0/8 `
    -Action Allow
```

### Block outbound to IP range
```powershell
New-NetFirewallRule `
    -DisplayName "Block Outbound to Bad Range" `
    -Direction Outbound `
    -Protocol Any `
    -RemoteAddress 198.51.100.0/24 `
    -Action Block
```

### Allow multiple ports (array or range)
```powershell
New-NetFirewallRule `
    -DisplayName "Allow Web Ports" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 80, 443, 8080 `
    -Action Allow

# Port range
New-NetFirewallRule `
    -DisplayName "Allow Ephemeral Range" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort "49152-65535" `
    -Action Allow
```

### Allow specific program
```powershell
New-NetFirewallRule `
    -DisplayName "Allow Python" `
    -Direction Inbound `
    -Program "C:\Python312\python.exe" `
    -Action Allow
```

### Allow with interface type restriction
```powershell
New-NetFirewallRule `
    -DisplayName "Allow RDP on LAN Only" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 3389 `
    -InterfaceType Wired, Wireless `
    -Action Allow
```

### All parameters reference
```powershell
New-NetFirewallRule `
    -Name          "RULE-UNIQUE-ID" `      # internal name (no spaces preferred)
    -DisplayName   "Human Label" `
    -Description   "Why this rule exists" `
    -Direction     Inbound `               # Inbound | Outbound
    -Protocol      TCP `                   # TCP | UDP | ICMPv4 | ICMPv6 | Any
    -LocalPort     80 `
    -RemotePort    Any `
    -LocalAddress  Any `
    -RemoteAddress Any `
    -Action        Allow `                 # Allow | Block | Allow & Bypass
    -Profile       Any `                   # Domain | Private | Public | Any
    -Enabled       True `
    -Program       Any `
    -Service       Any `
    -InterfaceType Any `                   # Any | Wired | Wireless | RemoteAccess
    -EdgeTraversalPolicy Block             # Block | Allow | DeferToUser | DeferToApp
```

---

## 3. Modify Rules

```powershell
# Rename display name
Set-NetFirewallRule -DisplayName "Old Name" -NewDisplayName "New Name"

# Change action
Set-NetFirewallRule -DisplayName "Allow HTTP" -Action Block

# Change remote address on existing rule
Set-NetFirewallRule -DisplayName "Allow WinRM" -RemoteAddress 192.168.1.20

# Update port
Set-NetFirewallRule -DisplayName "Custom App" -LocalPort 9090

# Change profile
Set-NetFirewallRule -DisplayName "Allow RDP" -Profile Domain, Private

# Bulk update — disable all public-profile inbound rules
Get-NetFirewallRule -Direction Inbound -Profile Public |
    Set-NetFirewallRule -Enabled False
```

---

## 4. Enable / Disable / Delete Rules

```powershell
# Enable by display name
Enable-NetFirewallRule -DisplayName "Allow SSH - Private"

# Disable by display name
Disable-NetFirewallRule -DisplayName "Allow SSH - Private"

# Enable all rules in a group
Enable-NetFirewallRule -Group "Remote Desktop"

# Disable all outbound block rules
Get-NetFirewallRule -Direction Outbound -Action Block |
    Disable-NetFirewallRule

# Delete a single rule
Remove-NetFirewallRule -DisplayName "Temp Debug Rule"

# Delete all disabled rules
Get-NetFirewallRule | Where-Object Enabled -eq False |
    Remove-NetFirewallRule

# Delete rules matching a pattern
Get-NetFirewallRule -DisplayName "*Test*" | Remove-NetFirewallRule
```

---

## 5. Firewall Profiles

```powershell
# View all profile settings
Get-NetFirewallProfile

# View specific profile
Get-NetFirewallProfile -Name Public

# Enable/disable firewall per profile
Set-NetFirewallProfile -Name Domain  -Enabled True
Set-NetFirewallProfile -Name Private -Enabled True
Set-NetFirewallProfile -Name Public  -Enabled True

# Block all inbound on Public (hardened)
Set-NetFirewallProfile -Name Public -DefaultInboundAction Block -DefaultOutboundAction Allow

# Allow inbound on all profiles (permissive — not recommended for prod)
Set-NetFirewallProfile -All -DefaultInboundAction Allow

# Block outbound when no rule matches (whitelist mode)
Set-NetFirewallProfile -Name Public -DefaultOutboundAction Block

# Set log file path and size
Set-NetFirewallProfile -Name Public `
    -LogFileName "%SystemRoot%\System32\LogFiles\Firewall\pfirewall.log" `
    -LogMaxSizeKilobytes 4096 `
    -LogAllowed True `
    -LogBlocked True

# Get currently active profile
(Get-NetConnectionProfile).NetworkCategory
```

---

## 6. Port & Address Filters

```powershell
# Get all port filters
Get-NetFirewallPortFilter

# Get address filters joined with rule names
Get-NetFirewallAddressFilter | Select-Object LocalAddress, RemoteAddress,
    @{N='Rule';E={(Get-NetFirewallRule -AssociatedNetFirewallAddressFilter $_).DisplayName}}

# Update address filter directly
Get-NetFirewallRule -DisplayName "MyRule" |
    Get-NetFirewallAddressFilter |
    Set-NetFirewallAddressFilter -RemoteAddress 10.0.1.0/24

# Update port filter directly
Get-NetFirewallRule -DisplayName "MyRule" |
    Get-NetFirewallPortFilter |
    Set-NetFirewallPortFilter -LocalPort 8443
```

---

## 7. Application Rules

```powershell
# Allow an app through firewall (all ports)
New-NetFirewallRule -DisplayName "Allow MyApp" `
    -Direction Inbound `
    -Program "C:\MyApp\myapp.exe" `
    -Action Allow

# Allow a Windows service (by service name)
New-NetFirewallRule -DisplayName "Allow WinRM Service" `
    -Direction Inbound `
    -Service WinRM `
    -Action Allow

# Find rules for a specific EXE
Get-NetFirewallApplicationFilter | Where-Object Program -like "*Teams*" |
    Get-NetFirewallRule | Select-Object DisplayName, Enabled, Action

# Remove app rules for an uninstalled program
Get-NetFirewallApplicationFilter |
    Where-Object Program -like "*OldApp*" |
    Get-NetFirewallRule |
    Remove-NetFirewallRule
```

---

## 8. IPsec / Advanced Security

```powershell
# View IPsec rules
Get-NetIPsecRule

# Create a simple IPsec require rule (authentication between two hosts)
New-NetIPsecRule `
    -DisplayName "Require IPsec - Server to DB" `
    -InboundSecurity Require `
    -OutboundSecurity Request `
    -RemoteAddress 10.0.1.50

# View main mode SA
Get-NetIPsecMainModeSA

# View quick mode SA
Get-NetIPsecQuickModeSA

# Flush all IPsec SAs (force renegotiation)
Remove-NetIPsecMainModeSA
Remove-NetIPsecQuickModeSA

# View connection security rules
Get-NetIPsecRule | Format-List *
```

---

## 9. Logging & Diagnostics

```powershell
# Enable firewall logging for all profiles
Set-NetFirewallProfile -All -LogAllowed True -LogBlocked True -LogIgnored True

# Check current log settings
Get-NetFirewallProfile | Select-Object Name, LogFileName, LogMaxSizeKilobytes, LogAllowed, LogBlocked

# Read the firewall log (default path)
$logPath = "$env:SystemRoot\System32\LogFiles\Firewall\pfirewall.log"
Get-Content $logPath | Select-Object -Last 50

# Parse blocked connections from log
Get-Content $logPath |
    Where-Object { $_ -match "^[^#]" -and $_ -match "DROP" } |
    ForEach-Object {
        $f = $_ -split " "
        [PSCustomObject]@{
            Date      = $f[0]
            Time      = $f[1]
            Action    = $f[2]
            Protocol  = $f[3]
            SrcIP     = $f[4]
            DstIP     = $f[5]
            SrcPort   = $f[6]
            DstPort   = $f[7]
        }
    } | Format-Table -AutoSize

# Test if a port is reachable (outbound test from local)
Test-NetConnection -ComputerName 8.8.8.8 -Port 53

# Trace route with port test
Test-NetConnection -ComputerName google.com -Port 443 -InformationLevel Detailed

# Check what's listening (active port bindings)
Get-NetTCPConnection -State Listen | Sort-Object LocalPort | Format-Table -AutoSize

# Check established connections
Get-NetTCPConnection -State Established | Format-Table LocalAddress, LocalPort, RemoteAddress, RemotePort -AutoSize

# Show process owning a port
Get-NetTCPConnection -LocalPort 8080 |
    Select-Object OwningProcess, @{N='Process';E={(Get-Process -Id $_.OwningProcess).Name}}

# Audit firewall event log (blocked connections generate event 5152 / 5157)
Get-WinEvent -LogName "Security" -FilterHashtable @{Id=5157} -MaxEvents 20 |
    Select-Object TimeCreated, Message
```

---

## 10. Export & Import

```powershell
# Export all rules to a .wfw backup (binary, use wf.msc or netsh)
netsh advfirewall export "C:\backup\firewall.wfw"

# Import from backup
netsh advfirewall import "C:\backup\firewall.wfw"

# Export rules to CSV for auditing
Get-NetFirewallRule | Select-Object Name, DisplayName, Direction, Action,
    Enabled, Profile, Description |
    Export-Csv -Path "C:\firewall-audit.csv" -NoTypeInformation

# Export to JSON
Get-NetFirewallRule |
    ConvertTo-Json -Depth 5 |
    Out-File "C:\firewall-rules.json"

# Export only enabled inbound allow rules
Get-NetFirewallRule -Direction Inbound -Action Allow | Where-Object Enabled -eq True |
    Export-Csv "C:\inbound-allow.csv" -NoTypeInformation

# Replicate rules to remote computer
$rules = Get-NetFirewallRule -DisplayName "MyRule"
Invoke-Command -ComputerName RemoteServer -ScriptBlock {
    param($r)
    New-NetFirewallRule -DisplayName $r.DisplayName -Direction $r.Direction -Action $r.Action
} -ArgumentList $rules
```

---

## 11. netsh Equivalents

| Task                          | PowerShell                                              | netsh                                              |
|-------------------------------|--------------------------------------------------------|----------------------------------------------------|
| Show all rules                | `Get-NetFirewallRule`                                  | `netsh advfirewall firewall show rule name=all`    |
| Add inbound TCP rule          | `New-NetFirewallRule -LocalPort 80 -Direction Inbound` | `netsh advfirewall firewall add rule name="HTTP" dir=in action=allow protocol=TCP localport=80` |
| Delete rule                   | `Remove-NetFirewallRule -DisplayName "..."` | `netsh advfirewall firewall delete rule name="..."` |
| Enable firewall               | `Set-NetFirewallProfile -Enabled True`                 | `netsh advfirewall set allprofiles state on`       |
| Disable firewall              | `Set-NetFirewallProfile -Enabled False`                | `netsh advfirewall set allprofiles state off`      |
| Block outbound default        | `Set-NetFirewallProfile -DefaultOutboundAction Block`  | `netsh advfirewall set allprofiles firewallpolicy blockinbound,blockoutbound` |
| Export rules                  | `netsh advfirewall export "file.wfw"`                  | same — no PS equivalent for binary format          |
| Reset to defaults             | `(netsh advfirewall reset)`                            | `netsh advfirewall reset`                          |

---

## 12. Quick-Reference Table

| Cmdlet                      | Purpose                                       |
|-----------------------------|-----------------------------------------------|
| `Get-NetFirewallRule`       | List / query rules                            |
| `New-NetFirewallRule`       | Create a new rule                             |
| `Set-NetFirewallRule`       | Modify an existing rule                       |
| `Enable-NetFirewallRule`    | Enable rule(s)                                |
| `Disable-NetFirewallRule`   | Disable rule(s)                               |
| `Remove-NetFirewallRule`    | Delete rule(s)                                |
| `Get-NetFirewallProfile`    | View profile settings (Domain/Private/Public) |
| `Set-NetFirewallProfile`    | Change profile defaults / logging             |
| `Get-NetFirewallPortFilter` | Get port-level filter objects                 |
| `Set-NetFirewallPortFilter` | Update port-level filter                      |
| `Get-NetFirewallAddressFilter` | Get IP address filter objects              |
| `Set-NetFirewallAddressFilter` | Update IP address filter                   |
| `Get-NetFirewallApplicationFilter` | Get program/service filter objects   |
| `Get-NetIPsecRule`          | List IPsec / connection security rules        |
| `New-NetIPsecRule`          | Create IPsec rule                             |
| `Test-NetConnection`        | Port connectivity test (telnet replacement)   |
| `Get-NetTCPConnection`      | Show active TCP connections and listeners     |

---

### Common Direction / Action / Protocol Values

| Parameter   | Values                                      |
|-------------|---------------------------------------------|
| `-Direction`  | `Inbound`, `Outbound`                     |
| `-Action`     | `Allow`, `Block`                          |
| `-Protocol`   | `TCP`, `UDP`, `ICMPv4`, `ICMPv6`, `Any`  |
| `-Profile`    | `Domain`, `Private`, `Public`, `Any`      |
| `-Enabled`    | `True`, `False`                           |

---

*Generated 2026-05-08 — Windows Defender Firewall with Advanced Security (NetSecurity module)*
