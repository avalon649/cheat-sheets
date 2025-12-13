#### SSH Tunneling

```bash
ssh -R port:ip:port server@ip
ssh -L port:ip:port server@ip
ssh -R 8080:127.0.0.1:8080 server@192.168.0.2
```