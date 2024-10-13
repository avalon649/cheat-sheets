#### Network Diagnosis

```bash
ipconfig /all | findstr DNS

ipconfig /release

ipconfig /renew "INTERFACE"

ipconfig /displaydns >> output.txt

ipcinfig /flushdns

nslookup example.com 

nslookup example.com dns-server

nslookup -type=mx example.com

getmac /v

netsh wlan show wlanreport

netsh interface ip show address | findstr "IP Address" | dnsservers

netsh advfirewall set allprofiles state off

ping -t example.com

tracert example.com

Test-NetConnection -ComputerName 192.168.0.6 -InformationLevel "Detailed" -Port 3389

netstat -af -o

route -p

route add ip.address mask subnet-mask gateway
```
#### Pc Diagnosis
```bash
powercfg /energy

powercfg /batteryreport

assoc

assoc .mp4=VLC.vlc

chkdsk /f 

sfc /scannow

DISM /Online /Cleanup-Image /CheckHealth /ScanHealth /RestoreHealth

tasklist | findstr process.exe

taskkill /f /pid   processId

shutdown /r /fw /f /t 0
```
