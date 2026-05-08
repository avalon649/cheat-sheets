## Scheduled Tasks

```powershell
# List all tasks
Get-ScheduledTask

# Filter by state
Get-ScheduledTask | Where-Object State -eq Running

# Run a task
Start-ScheduledTask -TaskName "MyTask"

# Disable a task
Disable-ScheduledTask -TaskName "MyTask"
```