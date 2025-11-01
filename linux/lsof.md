### List of Files LSOF Trouble Shooting Commands

#### Which Port?

```bash
lsof -i :22
lsof -i TCP:22
```

#### Which IpAddress?

```bash
lsof -i @127.0.0.1
lsof -i TCP@127.0.0.1
```

#### Which Process?

```bash
lsof -p 890
```

#### Which Files From Which User?

```bash
lsof -u admin
```
#### List Open Files

```bash
lsof +D /mnt/nfs_mount_point
```