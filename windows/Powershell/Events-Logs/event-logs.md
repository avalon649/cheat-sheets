## Event Logs

```powershell
# List available logs
Get-EventLog -List

# Last 20 system errors
Get-EventLog -LogName System -EntryType Error -Newest 20

# Last 20 application warnings
Get-EventLog -LogName Application -EntryType Warning -Newest 20

# Filter by event ID
Get-EventLog -LogName System -InstanceId 7036 -Newest 10

# Modern log access (preferred)
Get-WinEvent -LogName System -MaxEvents 20 |
    Where-Object LevelDisplayName -eq "Error" |
    Select-Object TimeCreated, Id, Message
```