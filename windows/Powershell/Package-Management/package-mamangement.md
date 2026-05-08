## Package Management

```powershell
# winget (Windows Package Manager)
winget search <app>
winget install <app>
winget upgrade --all

# Chocolatey
choco list --local-only
choco install <package>
choco upgrade all

# PowerShell modules
Find-Module <name>
Install-Module <name>
Update-Module <name>
Get-InstalledModule
```