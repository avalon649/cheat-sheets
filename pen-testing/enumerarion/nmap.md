### nmap scan
```bash
nmap -v -sV -sC $ip -oN output.txt
```

### vuln scan
```bash
nmap -v -sV --script vuln $ip -oN output.txt

```