# Windows Diagnostics Cheat Sheet

  ---

  ## Network Diagnosis

  | Command | Description |
  |---|---|
  | `ipconfig /all \| findstr DNS` | Show DNS server addresses |
  | `ipconfig /release` | Release the current DHCP IP address |
  | `ipconfig /renew "INTERFACE"` | Renew DHCP lease on a specific interface |
  | `ipconfig /displaydns >> output.txt` | Dump DNS cache to a file |
  | `ipconfig /flushdns` | Clear the DNS resolver cache |
  | `nslookup example.com` | DNS lookup using default server |
  | `nslookup example.com dns-server` | DNS lookup using a specific server |
  | `nslookup -type=mx example.com` | Query MX (mail) records |
  | `getmac /v` | Show MAC addresses with interface details |
  | `netsh wlan show wlanreport` | Generate a Wi-Fi health report |
  | `netsh interface ip show address \| findstr "IP Address"` | Show IP addresses for all
  interfaces |
  | `netsh interface ip show dnsservers` | Show configured DNS servers |
  | `netsh advfirewall set allprofiles state off` | Disable Windows Firewall on all
  profiles |
  | `ping -t example.com` | Continuous ping until stopped (`Ctrl+C`) |
  | `tracert example.com` | Trace the route packets take to a host |
  | `netstat -af -o` | Show all connections, FQDNs, and owning PIDs |
  | `route -p` | Show persistent static routes |
  | `route add <ip> mask <subnet> <gateway>` | Add a persistent static route |
  | `Test-NetConnection -ComputerName 192.168.0.6 -Port 3389 -InformationLevel Detailed` |
   Test TCP port reachability with detail |

  ---

  ## PC Diagnosis

  | Command | Description |
  |---|---|
  | `powercfg /energy` | Generate a power efficiency report |
  | `powercfg /batteryreport` | Generate a battery health report |
  | `assoc` | List all file extension associations |
  | `assoc .mp4=VLC.vlc` | Associate `.mp4` files with VLC |
  | `chkdsk /f` | Scan and fix filesystem errors on reboot |
  | `sfc /scannow` | Scan and repair protected system files |
  | `DISM /Online /Cleanup-Image /CheckHealth` | Check image for corruption (no repair) |
  | `DISM /Online /Cleanup-Image /ScanHealth` | Deep scan image for corruption |
  | `DISM /Online /Cleanup-Image /RestoreHealth` | Repair the Windows image |
  | `tasklist \| findstr process.exe` | Find a running process by name |
  | `taskkill /f /pid <processId>` | Force-kill a process by PID |
  | `shutdown /r /fw /f /t 0` | Reboot immediately into UEFI/BIOS firmware |