### Grab Text

```bash
cat file.txt |cut -d " " -f2.-f3
```

```bash
cat syslog | cut -d "," -f3 | sort | uniq -c
```