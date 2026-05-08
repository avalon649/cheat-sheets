## System Information

```powershell
# OS info
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsArchitecture

# CPU info
Get-WmiObject Win32_Processor | Select-Object Name, NumberOfCores, MaxClockSpeed

# RAM
Get-WmiObject Win32_PhysicalMemory | Measure-Object Capacity -Sum |
    Select-Object @{N="TotalRAM(GB)"; E={[math]::Round($_.Sum/1GB,2)}}

# Disk usage
Get-PSDrive -PSProvider FileSystem | Select-Object Name,
    @{N="Used(GB)";  E={[math]::Round(($_.Used/1GB),2)}},
    @{N="Free(GB)";  E={[math]::Round(($_.Free/1GB),2)}},
    @{N="Total(GB)"; E={[math]::Round(($_.Used/1GB + $_.Free/1GB),2)}}

# Uptime
(Get-Date) - (gcim Win32_OperatingSystem).LastBootUpTime

# Environment variables
$env:PATH
[System.Environment]::GetEnvironmentVariables()
```

---