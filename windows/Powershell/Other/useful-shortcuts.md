## Useful Shortcuts

```powershell
# Grep equivalent
Select-String -Path .\*.log -Pattern "error"
Get-Content file.log | Select-String "error"

# Measure command execution time
Measure-Command { Get-Process }

# Run as Administrator
Start-Process powershell -Verb RunAs

# Execution policy
Get-ExecutionPolicy
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser

# Command history
Get-History
Invoke-History 42        # re-run command #42

# Export output to CSV
Get-Process | Export-Csv processes.csv -NoTypeInformation

# Export to JSON
Get-Process | ConvertTo-Json | Out-File processes.json

# Base64 encode/decode
[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes("hello"))
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("aGVsbG8="))
```