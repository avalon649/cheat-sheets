## Install CIFS

```bash
apt -y update
apt-get install cifs-utls -y
```

```bash
nano /etc/fstab
```

```bash
touch .smbcredentials
user=user
password=pass
```

```bash
chmmod 600 .smbcredentials
```

## Install NFS
Install NFS Client on Ubuntu

```bash
 apt -y update
 apt -y install nfs-common
```

### Configuration
*TEMP EXAMPLE*:
`/srv/nfs 192.168.1.2(rw,sync,no_root_squash,subtree_check)`

### root rw permissions
Note the **root_squash** mount option. This option is set by default and must be disabled if not wanted.
*Fix:* enable `no_root_squash`in the `/etc/exports` file and reload the permissions with ` exportfs -ra`