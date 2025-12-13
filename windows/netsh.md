### netsh Commands

```bash
# Basic Syntax
netsh /?
netsh [context] /?
netsh [context] [command]

# Network Interfaces
netsh interface show interface
netsh interface set interface name="InterfaceName" admin=enabled
netsh interface set interface name="InterfaceName" admin=disabled

# IP Address Configuration
netsh interface ip show config
netsh interface ip set address name="InterfaceName" source=dhcp
netsh interface ip set address name="InterfaceName" static [IP Address] [Subnet Mask] [Gateway] [Metric]
netsh interface ip add dns name="InterfaceName" [DNS IP] index=1
netsh interface ip set dns name="InterfaceName" source=dhcp

# Wireless Networks
netsh wlan show profiles
netsh wlan show profile name="ProfileName" key=clear
netsh wlan connect name="ProfileName"
netsh wlan disconnect

# Firewall
netsh advfirewall show allprofiles
netsh advfirewall set allprofiles state on
netsh advfirewall set allprofiles state off
netsh advfirewall firewall add rule name="RuleName" protocol=[TCP/UDP] dir=in localport=[Port] action=allow
netsh advfirewall firewall delete rule name="RuleName"

# Port Forwarding
netsh interface portproxy add v4tov4 listenport=[Port] listenaddress=[Listen Address] connectport=[Port] connectaddress=[Connect Address]
netsh interface portproxy show all
netsh interface portproxy delete v4tov4 listenport=[Port] listenaddress=[Listen Address]

# Routing
netsh interface ipv4 show route
netsh interface ipv4 add route [Destination] [PrefixLength] [Gateway] [InterfaceName]
netsh interface ipv4 delete route [Destination] [PrefixLength] [Gateway] [InterfaceName]

# DHCP Client
netsh dhcp show server
netsh dhcp add server [Server] [IP Address]
netsh dhcp delete server [Server] [IP Address]

# HTTP
netsh http show iplisten
netsh http add iplisten [IP Address]
netsh http delete iplisten [IP Address]
netsh http show sslcert
netsh http add sslcert ipport=[IP Address:Port] certhash=[Cert Hash] appid=[App ID]
netsh http delete sslcert ipport=[IP Address:Port]

# Winsock
netsh winsock show catalog
netsh winsock reset

# Examples
netsh interface show interface
netsh interface ip set address name="Ethernet" static 192.168.1.10 255.255.255.0 192.168.1.1
netsh interface ip add dns name="Ethernet" 8.8.8.8 index=1
netsh wlan connect name="MyWiFiNetwork"
netsh advfirewall set allprofiles state off
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=192.168.1.10 connectport=80 connectaddress=192.168.1.20
netsh winsock reset
```