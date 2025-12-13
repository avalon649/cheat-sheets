### Sudo Without Password 

```bash
cd /etc/sudoers
```

```bash
# Allow members of group sudo to execute any command
#%sudo  ALL=(ALL:ALL) ALL
%sudo ALL=(ALL) NOPASSWD: ALL
```