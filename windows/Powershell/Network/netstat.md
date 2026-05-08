# PowerShell Netstat Cheat Sheet

> Network connection and socket inspection via `NetTCPIP` module, `Get-Net*` cmdlets, and `netstat.exe`

---

## Table of Contents

1. [TCP Connections](#1-tcp-connections)
2. [UDP Endpoints](#2-udp-endpoints)
3. [Listening Ports](#3-listening-ports)
4. [Process-to-Port Mapping](#4-process-to-port-mapping)
5. [Connection States](#5-connection-states)
6. [Filter by Address & Port](#6-filter-by-address--port)
7. [Network Statistics](#7-network-statistics)
8. [Routing Table](#8-routing-table)
9. [ARP & Neighbor Cache](#9-arp--neighbor-cache)
10. [DNS Cache](#10-dns-cache)
11. [Interface Information](#11-interface-information)
12. [Remote / WMI Queries](#12-remote--wmi-queries)
13. [Real-Time Monitoring](#13-real-time-monitoring)
14. [netstat.exe Reference](#14-netstatexe-reference)
15. [netstat.exe vs PowerShell Equivalents](#15-netstatexe-vs-powershell-equivalents)
16. [Quick-Reference Table](#16-quick-reference-table)

---

## 1. TCP Connections

```powershell
# All TCP connections (all states)
Get-NetTCPConnection

# Formatted table view
Get-NetTCPConnection | Format-Table -AutoSize

# Select key columns only
Get-NetTCPConnection |
    Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess |
    Format-Table -AutoSize

# Sort by local port
Get-NetTCPConnection | Sort-Object LocalPort | Format-Table -AutoSize

# Sort by remote address
Get-NetTCPConnection | Sort-Object RemoteAddress | Format-Table -AutoSize

# Count total connections
(Get-NetTCPConnection).Count

# Count connections grouped by state
Get-NetTCPConnection | Group-Object State | Select-Object Name, Count | Sort-Object Count -Descending

# Count connections grouped by remote address
Get-NetTCPConnection |
    Where-Object State -eq 'Established' |
    Group-Object RemoteAddress |
    Select-Object Name, Count |
    Sort-Object Count -Descending

# Show only IPv4 connections
Get-NetTCPConnection | Where-Object LocalAddress -notlike "*:*"

# Show only IPv6 connections
Get-NetTCPConnection | Where-Object LocalAddress -like "*:*"
```

---

## 2. UDP Endpoints

```powershell
# All UDP endpoints (UDP is stateless — no connection state)
Get-NetUDPEndpoint

# Formatted view
Get-NetUDPEndpoint | Format-Table -AutoSize

# Select key columns
Get-NetUDPEndpoint |
    Select-Object LocalAddress, LocalPort, OwningProcess |
    Sort-Object LocalPort |
    Format-Table -AutoSize

# Add process name to UDP output
Get-NetUDPEndpoint |
    Select-Object LocalAddress, LocalPort,
        @{N='PID';     E={ $_.OwningProcess }},
        @{N='Process'; E={ (Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name }} |
    Sort-Object LocalPort |
    Format-Table -AutoSize

# Filter UDP by port
Get-NetUDPEndpoint | Where-Object LocalPort -eq 53

# Count UDP endpoints by owning process
Get-NetUDPEndpoint |
    Group-Object OwningProcess |
    ForEach-Object {
        [PSCustomObject]@{
            PID     = $_.Name
            Process = (Get-Process -Id $_.Name -ErrorAction SilentlyContinue).Name
            Count   = $_.Count
        }
    } | Sort-Object Count -Descending
```

---

## 3. Listening Ports

```powershell
# TCP listening ports
Get-NetTCPConnection -State Listen

# TCP listening — with process names
Get-NetTCPConnection -State Listen |
    Select-Object LocalAddress, LocalPort,
        @{N='PID';     E={ $_.OwningProcess }},
        @{N='Process'; E={ (Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name }} |
    Sort-Object LocalPort |
    Format-Table -AutoSize

# UDP listening endpoints (all UDP is effectively "listening")
Get-NetUDPEndpoint |
    Select-Object LocalAddress, LocalPort,
        @{N='PID';     E={ $_.OwningProcess }},
        @{N='Process'; E={ (Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name }} |
    Sort-Object LocalPort |
    Format-Table -AutoSize

# All listening ports (TCP + UDP combined)
$tcp = Get-NetTCPConnection -State Listen |
    Select-Object @{N='Proto';E={'TCP'}}, LocalAddress, LocalPort, OwningProcess
$udp = Get-NetUDPEndpoint |
    Select-Object @{N='Proto';E={'UDP'}}, LocalAddress, LocalPort, OwningProcess

($tcp + $udp) |
    Select-Object Proto, LocalAddress, LocalPort,
        @{N='PID';     E={ $_.OwningProcess }},
        @{N='Process'; E={ (Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name }} |
    Sort-Object Proto, LocalPort |
    Format-Table -AutoSize

# Check if a specific port is listening
$port = 443
$listener = Get-NetTCPConnection -State Listen -LocalPort $port -ErrorAction SilentlyContinue
if ($listener) {
    "Port $port is OPEN — PID $($listener.OwningProcess)"
} else {
    "Port $port is NOT listening"
}

# List all unique listening TCP ports (numbers only)
(Get-NetTCPConnection -State Listen).LocalPort | Sort-Object -Unique
```

---

## 4. Process-to-Port Mapping

```powershell
# All TCP connections with process names (netstat -b equivalent)
Get-NetTCPConnection |
    Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State,
        @{N='PID';     E={ $_.OwningProcess }},
        @{N='Process'; E={ (Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name }} |
    Format-Table -AutoSize

# Find which process owns a specific port
$port = 8080
Get-NetTCPConnection -LocalPort $port -ErrorAction SilentlyContinue |
    ForEach-Object {
        $proc = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue
        [PSCustomObject]@{
            Port       = $_.LocalPort
            State      = $_.State
            PID        = $_.OwningProcess
            ProcessName = $proc.Name
            Path       = $proc.Path
        }
    }

# Find which process is listening on port 443
$conn = Get-NetTCPConnection -LocalPort 443 -State Listen -ErrorAction SilentlyContinue
if ($conn) { Get-Process -Id $conn.OwningProcess }

# Show all ports used by a specific process name
$targetProcess = "chrome"
$pid = (Get-Process -Name $targetProcess -ErrorAction SilentlyContinue).Id
Get-NetTCPConnection | Where-Object OwningProcess -in $pid |
    Select-Object LocalPort, RemoteAddress, RemotePort, State |
    Sort-Object LocalPort

# Show all ports used by a specific PID
Get-NetTCPConnection -OwningProcess 1234

# Comprehensive port-process table (TCP + UDP)
function Get-PortProcess {
    $tcp = Get-NetTCPConnection | ForEach-Object {
        $p = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue
        [PSCustomObject]@{
            Proto       = 'TCP'
            LocalAddr   = $_.LocalAddress
            LocalPort   = $_.LocalPort
            RemoteAddr  = $_.RemoteAddress
            RemotePort  = $_.RemotePort
            State       = $_.State
            PID         = $_.OwningProcess
            ProcessName = $p.Name
            Path        = $p.Path
        }
    }
    $udp = Get-NetUDPEndpoint | ForEach-Object {
        $p = Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue
        [PSCustomObject]@{
            Proto       = 'UDP'
            LocalAddr   = $_.LocalAddress
            LocalPort   = $_.LocalPort
            RemoteAddr  = '*'
            RemotePort  = '*'
            State       = 'N/A'
            PID         = $_.OwningProcess
            ProcessName = $p.Name
            Path        = $p.Path
        }
    }
    $tcp + $udp | Sort-Object Proto, LocalPort
}

Get-PortProcess | Format-Table -AutoSize
```

---

## 5. Connection States

```powershell
# Filter by specific state
Get-NetTCPConnection -State Established
Get-NetTCPConnection -State Listen
Get-NetTCPConnection -State TimeWait
Get-NetTCPConnection -State CloseWait
Get-NetTCPConnection -State SynSent
Get-NetTCPConnection -State SynReceived
Get-NetTCPConnection -State FinWait1
Get-NetTCPConnection -State FinWait2
Get-NetTCPConnection -State Closing
Get-NetTCPConnection -State LastAck
Get-NetTCPConnection -State Bound          # bound but not yet listening
Get-NetTCPConnection -State Closed

# State summary dashboard
Get-NetTCPConnection |
    Group-Object State |
    Select-Object Name, Count |
    Sort-Object Count -Descending |
    Format-Table -AutoSize

# Connections stuck in TIME_WAIT (post-close cleanup)
Get-NetTCPConnection -State TimeWait | Measure-Object | Select-Object Count

# Connections stuck in CLOSE_WAIT (remote closed, local hasn't — possible app bug)
Get-NetTCPConnection -State CloseWait |
    Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort,
        @{N='Process'; E={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name}}

# Half-open connections (SYN_SENT — possible failed outbound attempts)
Get-NetTCPConnection -State SynSent |
    Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort,
        @{N='Process'; E={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name}}

# TCP State descriptions:
#   Listen       — waiting for incoming connection requests
#   SynSent      — sent SYN, waiting for SYN-ACK
#   SynReceived  — received SYN, sent SYN-ACK, waiting for ACK
#   Established  — connection open, data transfer in progress
#   FinWait1     — sent FIN, waiting for ACK or FIN
#   FinWait2     — received ACK of FIN, waiting for remote FIN
#   CloseWait    — remote sent FIN, local app hasn't closed yet
#   Closing      — both sides sent FIN simultaneously
#   LastAck      — sent FIN, waiting for final ACK
#   TimeWait     — waiting for stray packets (2×MSL timeout, ~120s)
#   Closed       — connection closed
#   Bound        — socket bound to address/port, not yet listening
```

---

## 6. Filter by Address & Port

```powershell
# Filter by local port
Get-NetTCPConnection -LocalPort 80
Get-NetTCPConnection -LocalPort 443
Get-NetTCPConnection -LocalPort 3389   # RDP

# Filter by remote port
Get-NetTCPConnection -RemotePort 443

# Filter by local address
Get-NetTCPConnection -LocalAddress 0.0.0.0       # all interfaces
Get-NetTCPConnection -LocalAddress 127.0.0.1     # loopback only
Get-NetTCPConnection -LocalAddress 192.168.1.10  # specific interface

# Filter by remote address
Get-NetTCPConnection -RemoteAddress 8.8.8.8
Get-NetTCPConnection | Where-Object RemoteAddress -like "10.0.*"

# Combine filters (local port + state)
Get-NetTCPConnection -LocalPort 443 -State Established

# Filter by port range
Get-NetTCPConnection |
    Where-Object { $_.LocalPort -ge 8000 -and $_.LocalPort -le 9000 }

# Find connections to a specific subnet
Get-NetTCPConnection -State Established |
    Where-Object RemoteAddress -like "192.168.1.*"

# Find connections to external IPs (not loopback or private RFC1918)
Get-NetTCPConnection -State Established |
    Where-Object {
        $ip = $_.RemoteAddress
        $ip -ne '0.0.0.0'   -and
        $ip -ne '::'         -and
        $ip -notlike '127.*' -and
        $ip -notlike '10.*'  -and
        $ip -notlike '192.168.*' -and
        $ip -notlike '172.1[6-9].*' -and
        $ip -notlike '172.2[0-9].*' -and
        $ip -notlike '172.3[0-1].*'
    } |
    Select-Object LocalPort, RemoteAddress, RemotePort,
        @{N='Process'; E={(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).Name}}

# Show connections to a specific remote host (resolve first)
$ip = [System.Net.Dns]::GetHostAddresses("google.com").IPAddressToString
Get-NetTCPConnection | Where-Object RemoteAddress -in $ip

# Port scan check — find which common ports are locally open
$commonPorts = 21,22,23,25,53,80,110,135,139,143,443,445,3306,3389,5985,8080,8443
foreach ($p in $commonPorts) {
    $c = Get-NetTCPConnection -LocalPort $p -State Listen -ErrorAction SilentlyContinue
    if ($c) { "OPEN: $p — $((Get-Process -Id $c.OwningProcess -EA SilentlyContinue).Name)" }
}
```

---

## 7. Network Statistics

```powershell
# TCP statistics
Get-NetTCPSetting
Get-NetTcpStatistics

# Per-interface TCP stats (CIM)
Get-CimInstance -ClassName Win32_PerfRawData_Tcpip_TCPv4 |
    Select-Object ConnectionsEstablished, ConnectionsActive, ConnectionsPassive,
                  ConnectionFailures, ConnectionsReset, SegmentsReceived, SegmentsSent

# UDP statistics
Get-CimInstance -ClassName Win32_PerfRawData_Tcpip_UDPv4 |
    Select-Object DatagramsReceived, DatagramsSent, DatagramsNoPort, DatagramsReceivedErrors

# IP statistics
Get-CimInstance -ClassName Win32_PerfRawData_Tcpip_IPv4 |
    Select-Object DatagramsReceived, DatagramsForwarded, DatagramsSent,
                  DatagramsOutboundDiscarded, DatagramsReceivedDiscarded

# ICMP statistics
Get-CimInstance -ClassName Win32_PerfRawData_Tcpip_ICMPv4 |
    Select-Object MessagesReceived, MessagesSent,
                  EchoRequestsReceived, EchoRequestsSent,
                  EchoRepliesReceived,  EchoRepliesSent

# Network interface byte counts (bandwidth usage)
Get-CimInstance -ClassName Win32_PerfRawData_Tcpip_NetworkInterface |
    Select-Object Name,
        @{N='Recv MB'; E={[math]::Round($_.BytesReceivedPerSec/1MB,2)}},
        @{N='Sent MB'; E={[math]::Round($_.BytesSentPerSec/1MB,2)}}

# Netstat -s equivalent (protocol stats)
netstat -s
netstat -s -p tcp
netstat -s -p udp
netstat -s -p ip
netstat -s -p icmp
```

---

## 8. Routing Table

```powershell
# View routing table (equivalent to route print)
Get-NetRoute

# IPv4 routes only
Get-NetRoute -AddressFamily IPv4

# IPv6 routes only
Get-NetRoute -AddressFamily IPv6

# Formatted routing table
Get-NetRoute -AddressFamily IPv4 |
    Select-Object ifIndex, DestinationPrefix, NextHop, RouteMetric, ifMetric |
    Sort-Object DestinationPrefix |
    Format-Table -AutoSize

# Show default gateway(s)
Get-NetRoute -DestinationPrefix '0.0.0.0/0'
Get-NetRoute -DestinationPrefix '::/0'     # IPv6 default

# Get default gateway as string
(Get-NetRoute -DestinationPrefix '0.0.0.0/0' | Sort-Object RouteMetric)[0].NextHop

# Add a static route
New-NetRoute -DestinationPrefix '10.5.0.0/16' -NextHop '192.168.1.1' -InterfaceIndex 12

# Remove a static route
Remove-NetRoute -DestinationPrefix '10.5.0.0/16' -Confirm:$false

# Flush all non-persistent routes (similar to route -f)
Get-NetRoute | Where-Object Protocol -ne 'NetMgmt' | Remove-NetRoute -Confirm:$false

# Route print via cmd (still useful for full output)
route print
route print -4    # IPv4 only
route print -6    # IPv6 only

# Add persistent route (survives reboot)
route add 10.5.0.0 mask 255.255.0.0 192.168.1.1 -p
```

---

## 9. ARP & Neighbor Cache

```powershell
# View ARP table (IPv4 neighbors)
Get-NetNeighbor -AddressFamily IPv4

# View NDP table (IPv6 neighbors)
Get-NetNeighbor -AddressFamily IPv6

# Formatted ARP table
Get-NetNeighbor -AddressFamily IPv4 |
    Select-Object ifIndex, IPAddress, LinkLayerAddress, State |
    Format-Table -AutoSize

# Filter by state
Get-NetNeighbor | Where-Object State -eq Reachable
Get-NetNeighbor | Where-Object State -eq Stale
Get-NetNeighbor | Where-Object State -eq Permanent

# Find a specific MAC address
Get-NetNeighbor | Where-Object LinkLayerAddress -eq "00-11-22-33-44-55"

# Find a specific IP in ARP cache
Get-NetNeighbor -IPAddress 192.168.1.1

# Flush ARP cache
Remove-NetNeighbor -Confirm:$false
# or per interface:
Remove-NetNeighbor -InterfaceIndex 12 -Confirm:$false

# Add a static ARP entry
New-NetNeighbor -InterfaceIndex 12 -IPAddress 192.168.1.50 -LinkLayerAddress "AA-BB-CC-DD-EE-FF" -State Permanent

# Traditional arp command
arp -a                   # view all
arp -a 192.168.1.1       # view specific
arp -d 192.168.1.1       # delete entry
arp -s 192.168.1.1 00-11-22-33-44-55  # add static

# Neighbor state values:
#   Reachable  — confirmed reachable (fresh)
#   Stale      — reachable but not confirmed recently
#   Delay      — waiting to confirm reachability
#   Probe      — actively probing
#   Permanent  — static entry
#   Incomplete — MAC not yet resolved
#   Unreachable — not reachable
```

---

## 10. DNS Cache

```powershell
# View DNS resolver cache
Get-DnsClientCache

# Formatted DNS cache
Get-DnsClientCache |
    Select-Object Entry, RecordName, RecordType, Status, DataLength, TTL, Data |
    Sort-Object Entry |
    Format-Table -AutoSize

# Filter by record type
Get-DnsClientCache | Where-Object RecordType -eq A     # IPv4
Get-DnsClientCache | Where-Object RecordType -eq AAAA  # IPv6
Get-DnsClientCache | Where-Object RecordType -eq CNAME

# Find cached entry for a specific domain
Get-DnsClientCache | Where-Object Entry -like "*google*"

# Flush DNS cache
Clear-DnsClientCache

# Flush and confirm
Clear-DnsClientCache
Get-DnsClientCache   # should be empty

# DNS lookup (nslookup equivalent)
Resolve-DnsName google.com                             # A + AAAA
Resolve-DnsName google.com -Type A                     # IPv4 only
Resolve-DnsName google.com -Type AAAA                  # IPv6 only
Resolve-DnsName google.com -Type MX                    # mail servers
Resolve-DnsName google.com -Type TXT                   # TXT records
Resolve-DnsName google.com -Type NS                    # name servers
Resolve-DnsName google.com -Type SOA                   # start of authority
Resolve-DnsName 8.8.8.8    -Type PTR                   # reverse lookup

# Query specific DNS server
Resolve-DnsName google.com -Server 1.1.1.1

# ipconfig equivalents
ipconfig /displaydns          # view DNS cache
ipconfig /flushdns            # flush DNS cache
ipconfig /registerdns         # re-register DNS
```

---

## 11. Interface Information

```powershell
# List network interfaces
Get-NetAdapter

# Detailed interface info
Get-NetAdapter | Format-List *

# Only connected interfaces
Get-NetAdapter | Where-Object Status -eq Up

# Only disconnected interfaces
Get-NetAdapter | Where-Object Status -eq Disconnected

# Interface speed (in bits/sec)
Get-NetAdapter | Select-Object Name, Status, LinkSpeed, MacAddress

# IP addresses assigned to interfaces
Get-NetIPAddress

# IPv4 only
Get-NetIPAddress -AddressFamily IPv4

# Non-loopback IPv4 addresses
Get-NetIPAddress -AddressFamily IPv4 | Where-Object IPAddress -ne '127.0.0.1'

# Interface + IP combined view
Get-NetIPAddress -AddressFamily IPv4 |
    Where-Object IPAddress -ne '127.0.0.1' |
    Select-Object InterfaceAlias, IPAddress, PrefixLength,
        @{N='Subnet'; E={"/$($_.PrefixLength)"}} |
    Format-Table -AutoSize

# Get gateway per interface
Get-NetIPConfiguration |
    Select-Object InterfaceAlias,
        @{N='IPv4';    E={ $_.IPv4Address.IPAddress }},
        @{N='Gateway'; E={ $_.IPv4DefaultGateway.NextHop }},
        @{N='DNS';     E={ $_.DNSServer.ServerAddresses -join ', ' }}

# Interface statistics
Get-NetAdapterStatistics |
    Select-Object Name, ReceivedBytes, SentBytes,
        @{N='Recv MB'; E={[math]::Round($_.ReceivedBytes/1MB,2)}},
        @{N='Sent MB'; E={[math]::Round($_.SentBytes/1MB,2)}}

# Enable / disable interface
Disable-NetAdapter -Name "Wi-Fi" -Confirm:$false
Enable-NetAdapter  -Name "Wi-Fi"

# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.100 -PrefixLength 24 -DefaultGateway 192.168.1.1

# Remove static IP / switch to DHCP
Remove-NetIPAddress -InterfaceAlias "Ethernet" -Confirm:$false
Set-NetIPInterface  -InterfaceAlias "Ethernet" -Dhcp Enabled

# Set DNS servers
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 1.1.1.1, 8.8.8.8

# Reset to automatic DNS
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses

# ipconfig equivalents
ipconfig                   # basic IP info
ipconfig /all              # full details (MAC, DHCP, DNS, etc.)
ipconfig /release          # release DHCP lease
ipconfig /renew            # renew DHCP lease
```

---

## 12. Remote / WMI Queries

```powershell
# Get TCP connections on a remote machine (requires WinRM)
Invoke-Command -ComputerName RemoteServer -ScriptBlock {
    Get-NetTCPConnection -State Established |
        Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort,
            @{N='Process'; E={(Get-Process -Id $_.OwningProcess -EA SilentlyContinue).Name}}
}

# Get listening ports on multiple servers
$servers = "Server01","Server02","Server03"
Invoke-Command -ComputerName $servers -ScriptBlock {
    Get-NetTCPConnection -State Listen |
        Select-Object LocalPort,
            @{N='Process'; E={(Get-Process -Id $_.OwningProcess -EA SilentlyContinue).Name}},
            @{N='Host';    E={$env:COMPUTERNAME}} |
        Sort-Object LocalPort
} | Format-Table -AutoSize

# WMI-based connection query (no WinRM, uses DCOM)
Get-WmiObject -Class Win32_NetworkConnection -ComputerName RemoteServer

# CIM session approach
$session = New-CimSession -ComputerName RemoteServer
Get-NetTCPConnection -CimSession $session -State Established
Remove-CimSession $session

# Test port reachability from local to remote
Test-NetConnection -ComputerName RemoteServer -Port 443
Test-NetConnection -ComputerName RemoteServer -Port 22 -InformationLevel Detailed

# Test multiple ports on multiple hosts
$hosts = "10.0.0.1","10.0.0.2","10.0.0.3"
$ports = 22, 80, 443, 3389
foreach ($h in $hosts) {
    foreach ($p in $ports) {
        $r = Test-NetConnection -ComputerName $h -Port $p -WarningAction SilentlyContinue
        [PSCustomObject]@{
            Host    = $h
            Port    = $p
            Open    = $r.TcpTestSucceeded
        }
    }
} | Format-Table -AutoSize

# Traceroute
Test-NetConnection -ComputerName 8.8.8.8 -TraceRoute
(Test-NetConnection -ComputerName 8.8.8.8 -TraceRoute).TraceRoute
```

---

## 13. Real-Time Monitoring

```powershell
# Watch active connections refresh every 2 seconds (like watch netstat)
while ($true) {
    Clear-Host
    Write-Host "$(Get-Date -Format 'HH:mm:ss') — Active Connections" -ForegroundColor Cyan
    Get-NetTCPConnection -State Established |
        Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort,
            @{N='Process'; E={(Get-Process -Id $_.OwningProcess -EA SilentlyContinue).Name}} |
        Sort-Object LocalPort |
        Format-Table -AutoSize
    Start-Sleep 2
}

# Monitor for new connections (delta detection)
$prev = @{}
while ($true) {
    $curr = Get-NetTCPConnection -State Established |
        Group-Object { "$($_.LocalPort)-$($_.RemoteAddress)-$($_.RemotePort)" } |
        ForEach-Object { $_.Name }

    $new = $curr | Where-Object { -not $prev.ContainsKey($_) }
    $gone = $prev.Keys | Where-Object { $_ -notin $curr }

    foreach ($c in $new)  { Write-Host "[NEW]  $c" -ForegroundColor Green }
    foreach ($c in $gone) { Write-Host "[GONE] $c" -ForegroundColor Red }

    $prev = @{}
    $curr | ForEach-Object { $prev[$_] = $true }
    Start-Sleep 3
}

# Count connections per second (connection rate)
$prev = (Get-NetTCPConnection -State Established).Count
while ($true) {
    Start-Sleep 5
    $curr = (Get-NetTCPConnection -State Established).Count
    Write-Host "$(Get-Date -Format 'HH:mm:ss') | Established: $curr | Delta: $($curr - $prev)"
    $prev = $curr
}

# Bandwidth usage snapshot per interface
while ($true) {
    Clear-Host
    Get-NetAdapterStatistics | Select-Object Name,
        @{N='Recv MB'; E={[math]::Round($_.ReceivedBytes/1MB,2)}},
        @{N='Sent MB'; E={[math]::Round($_.SentBytes/1MB,2)}} |
        Format-Table -AutoSize
    Start-Sleep 2
}

# Alert when a specific port gets a new connection
$port = 3389
$known = @{}
while ($true) {
    Get-NetTCPConnection -LocalPort $port -State Established -ErrorAction SilentlyContinue |
        ForEach-Object {
            $key = "$($_.RemoteAddress):$($_.RemotePort)"
            if (-not $known.ContainsKey($key)) {
                Write-Warning "New connection on port $port from $key at $(Get-Date)"
                $known[$key] = $true
            }
        }
    Start-Sleep 5
}
```

---

## 14. netstat.exe Reference

```cmd
:: All connections and listening ports
netstat -a

:: Show numeric addresses (no DNS resolution — faster)
netstat -n

:: Show owning process ID
netstat -o

:: Show executable name (requires elevation)
netstat -b

:: All + numeric + PID
netstat -ano

:: All + numeric + PID + executable name (elevated)
netstat -abno

:: Protocol filter
netstat -a -p tcp
netstat -a -p udp
netstat -a -p tcpv6
netstat -a -p udpv6

:: Routing table
netstat -r

:: Interface statistics
netstat -e

:: Protocol statistics
netstat -s
netstat -s -p tcp
netstat -s -p udp
netstat -s -p ip
netstat -s -p icmp

:: Refresh every N seconds (Ctrl+C to stop)
netstat -ano 5

:: Combined: all TCP, numeric, PID, refresh every 3s
netstat -ano -p tcp 3

:: Find what's on a specific port (pipe to findstr)
netstat -ano | findstr ":443"
netstat -ano | findstr "ESTABLISHED"
netstat -ano | findstr "LISTENING"
netstat -ano | findstr ":3389 "

:: Then resolve PID to process
tasklist /fi "PID eq 1234"
```

---

## 15. netstat.exe vs PowerShell Equivalents

| netstat flag / task          | PowerShell equivalent                                                |
|------------------------------|----------------------------------------------------------------------|
| `netstat -a`                 | `Get-NetTCPConnection; Get-NetUDPEndpoint`                           |
| `netstat -n`                 | `Get-NetTCPConnection` (already numeric)                             |
| `netstat -o`                 | `Get-NetTCPConnection \| Select-Object ..., OwningProcess`           |
| `netstat -b`                 | `... \| Select @{N='Proc';E={(Get-Process -Id $_.OwningProcess).Name}}` |
| `netstat -ano`               | `Get-NetTCPConnection \| Select LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess` |
| `netstat -r`                 | `Get-NetRoute`                                                       |
| `netstat -e`                 | `Get-NetAdapterStatistics`                                           |
| `netstat -s`                 | `Get-NetTcpStatistics; Get-CimInstance Win32_PerfRawData_Tcpip_*`   |
| `netstat -a -p tcp`          | `Get-NetTCPConnection`                                               |
| `netstat -a -p udp`          | `Get-NetUDPEndpoint`                                                 |
| Listening ports              | `Get-NetTCPConnection -State Listen`                                 |
| Established connections      | `Get-NetTCPConnection -State Established`                            |
| Filter by port               | `Get-NetTCPConnection -LocalPort 443`                                |
| Filter by PID                | `Get-NetTCPConnection -OwningProcess 1234`                           |
| ARP table (`arp -a`)         | `Get-NetNeighbor -AddressFamily IPv4`                                |
| Routing table (`route print`)| `Get-NetRoute -AddressFamily IPv4`                                   |
| DNS cache (`ipconfig /displaydns`) | `Get-DnsClientCache`                                         |
| Flush DNS (`ipconfig /flushdns`)   | `Clear-DnsClientCache`                                       |
| Interface config (`ipconfig /all`) | `Get-NetIPConfiguration`; `Get-NetAdapter`                   |
| Ping (`ping`)                | `Test-Connection -ComputerName host`                                 |
| Port test (`telnet`)         | `Test-NetConnection -ComputerName host -Port 443`                    |
| Traceroute (`tracert`)       | `Test-NetConnection -ComputerName host -TraceRoute`                  |
| nslookup                     | `Resolve-DnsName hostname`                                           |

---

## 16. Quick-Reference Table

| Cmdlet                        | Purpose                                                  |
|-------------------------------|----------------------------------------------------------|
| `Get-NetTCPConnection`        | List TCP connections with state and PID                  |
| `Get-NetUDPEndpoint`          | List UDP endpoints with PID                              |
| `Get-NetRoute`                | View routing table                                       |
| `Get-NetNeighbor`             | View ARP / NDP neighbor cache                            |
| `Get-NetAdapter`              | List network interfaces                                  |
| `Get-NetIPAddress`            | IP addresses assigned to interfaces                      |
| `Get-NetIPConfiguration`      | Full interface config (IP, gateway, DNS)                 |
| `Get-NetAdapterStatistics`    | Per-interface byte/packet counters                       |
| `Get-NetTcpStatistics`        | TCP protocol statistics                                  |
| `Get-DnsClientCache`          | View DNS resolver cache                                  |
| `Clear-DnsClientCache`        | Flush DNS cache                                          |
| `Resolve-DnsName`             | DNS lookup (nslookup replacement)                        |
| `Test-NetConnection`          | Port reachability test + traceroute                      |
| `Test-Connection`             | ICMP ping                                                |
| `New-NetRoute`                | Add a static route                                       |
| `Remove-NetRoute`             | Delete a route                                           |
| `New-NetNeighbor`             | Add a static ARP entry                                   |
| `Remove-NetNeighbor`          | Delete ARP entries / flush ARP cache                     |
| `New-NetIPAddress`            | Assign a static IP to an interface                       |
| `Set-DnsClientServerAddress`  | Configure DNS servers for an interface                   |

### Common Filter Patterns

```powershell
# Quick one-liners for daily use

# What's listening right now?
Get-NetTCPConnection -State Listen | Select-Object LocalPort,
    @{N='Process';E={(Get-Process -Id $_.OwningProcess -EA 0).Name}} | Sort-Object LocalPort

# Who is connected to me?
Get-NetTCPConnection -State Established | Where-Object RemoteAddress -ne '127.0.0.1' |
    Select-Object RemoteAddress, RemotePort, LocalPort,
        @{N='Process';E={(Get-Process -Id $_.OwningProcess -EA 0).Name}}

# What's phoning home (outbound established)?
Get-NetTCPConnection -State Established |
    Where-Object { $_.LocalPort -gt 1024 -and $_.RemotePort -in 80,443,8080,8443 } |
    Select-Object LocalPort, RemoteAddress, RemotePort,
        @{N='Process';E={(Get-Process -Id $_.OwningProcess -EA 0).Name}}

# What owns port X?
$p = 8080; Get-NetTCPConnection -LocalPort $p -EA 0 |
    ForEach-Object { "PID $($_.OwningProcess) — $((Get-Process -Id $_.OwningProcess -EA 0).Name)" }
```

---

*Generated 2026-05-08 — Windows PowerShell 5.1 / PowerShell 7.x*
