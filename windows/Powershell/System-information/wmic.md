### wmic Command Cheat Sheet

```bash
# Basic Syntax
wmic /?
wmic [alias] /?
wmic [alias] [where clause] get [property]

# System Information
wmic computersystem get /all
wmic bios get /all
wmic os get /all
wmic cpu get /all
wmic memorychip get /all

# Disk and Storage
wmic diskdrive get /all
wmic logicaldisk get /all
wmic partition get /all

# Network
wmic nic get /all
wmic nicconfig get /all
wmic nicconfig get IPAddress

# Processes and Services
wmic process list /all
wmic process where "name='processname.exe'" get /all
wmic process where "ProcessID=pid" call terminate
wmic service list /all
wmic service where "name='servicename'" get /all

# Users and Groups
wmic useraccount get /all
wmic group get /all

# Hardware
wmic path Win32_Battery get /all
wmic path Win32_Keyboard get /all
wmic path Win32_PointingDevice get /all
wmic path Win32_SoundDevice get /all

# Miscellaneous
wmic environment get /all
wmic product get /all
wmic startup get /all
wmic nteventlog get /all

# Examples
wmic computersystem get model,manufacturer
wmic os get name,version
wmic logicaldisk get name,size
wmic nic get macaddress
wmic process get name,processid
```