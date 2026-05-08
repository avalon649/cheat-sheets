## Process Management

```powershell
# List all processes
Get-Process

# Find process by name
Get-Process -Name chrome

# Find process by PID
Get-Process -Id 1234

# Kill process by name
Stop-Process -Name notepad
Stop-Process -Name notepad -Force

# Kill process by PID
Stop-Process -Id 1234

# Get process with full path
Get-Process | Select-Object Id, ProcessName, Path

# Top CPU-consuming processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Top memory-consuming processes
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10 Name, Id, @{N="RAM(MB)"; E={[math]::Round($_.WorkingSet/1MB,2)}}