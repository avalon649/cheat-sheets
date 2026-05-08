## Users & Groups

```powershell
# Local users
Get-LocalUser

# Local groups
Get-LocalGroup

# Group members
Get-LocalGroupMember -Group "Administrators"

# Current user
whoami
[System.Security.Principal.WindowsIdentity]::GetCurrent().Name

# Check if running as admin
([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```