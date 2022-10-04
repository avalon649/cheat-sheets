# Linux Basics

## Find File or Folder

```bash
find / -name filenamme 2>/dev/null
```

## Generate Basics Auth Password

```bash
echo $(htpasswd -nb <USER> <PASSWORD>) | sed -e s/\\$/\\$\\$/g
```
## Grab Text

```bash
cat file.txt |cut -d " " -f2.-f3
```

## Kill Commands

```bash
ps -u(userid) | grep (processname)
kill (process-id)
pgrep
kgrep
top
htop
```

## Install CIFS

```bash
apt-get install cifs-utls -y

nano /etc/fstab

touch .smbcredentials

chmmod 600 .smbcredentials
```
## Install NFS
Install NFS Client on Ubuntu

```bash
sudo apt -y update
sudo apt -y install nfs-common
```
### Configuration
*TEMP EXAMPLE*:
`/srv/nfs 192.168.1.2(rw,sync,no_root_squash,subtree_check)`

### root rw permissions
Note the **root_squash** mount option. This option is set by default and must be disabled if not wanted.
*Fix:* enable `no_root_squash`in the `/etc/exports` file and reload the permissions with `sudo exportfs -ra`


## SCP

```bash
scp file.txt hostname@ipaddress:/path/to/destination/file.txt
```

## Debian PKG Search

```bash
dpkg --list
dpkg --get-selections | grep name
```

## APT Search

```bash
apt-cache search package name
apt list --installed | grep (package-name)
```
## Word Count 

```bash
| wc -l
| less
```
## Launch A Quick Web Server

```bash
python -m http.server 7777

php -S 127.0.0.1:8888

npx http.server -p 8888
```

## Change Sudo to NoPass

`cd /etc/sudoers`

```bash
# Allow members of group sudo to execute any command
#%sudo  ALL=(ALL:ALL) ALL
%sudo ALL=(ALL) NOPASSWD: ALL
```

Clear All Logs

```bash
truncate -s 0 /var/log/*log
```

## Encode Data

```bash
echo -n 'secret_key' | openssl base64
echo p@ssw0rd | base64
echo cEA1NXdvcmQK | base64 --decode
```

## Linux Unattended-Upgrade

```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
sudo unnattended-upgrade --dry-run -debug
```

## Add User

```bash
useradd username -m -s /bin/bash -c "comment"
usermmod -aG sudo,adm,docker username
passwd username
```

## List Using Wildcards
```bash
ls -l *.conf
```