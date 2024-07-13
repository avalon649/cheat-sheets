### Generate key

```bash
ssh-keygen -t ecdsa -b 521
ssh-keygen -t rsa -b 4096 -f test.rsa
```

### Import public ssh key

```bash
ssh-copy-id -i key.pub admin@192.168.5.55
```

### Port fowarding

```bash
ssh -D <port> -N -C root@127.0.0.1
```