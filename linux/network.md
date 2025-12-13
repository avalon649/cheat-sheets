## Change Hostname

```bash
hostnamectl set-hostname newhostname
```
## Change IP Address in Ubuntu 20.04 LTS
1. Create a new file `/etc/netplan/01-netcfg.yaml`
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: no
      addresses:
        - 192.168.121.221/24
      gateway4: 192.168.121.1
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
```
2. Apply changes

```bash
netplay apply
```

## List All Listenting Ports on Your Machine

```bash
lsof -i -P -n |grep LISTEN

netstat -ntlp
```

## Launch A Quick Web Server

```bash
python -m http.server 7777

php -S 127.0.0.1:8888

npx http.server -p 8888
```
### Interfaces Config

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.0.21.55
netmask 255.255.255.0
gateway 10.0.21.1
```