### Adding find result to a varible

```bash
log=$(find / -name *.log)

echo "$log"
```

### Find all files that contain a string

```bash
find / -type f -exec grep -l "string" {} +
```

### Find all files that contain a name, case insensitive
```bash
find / -iname "file.log"
```

### Find by file

```bash
find / -type f "file.log"
```

### Find by directory

```bash
find / -type d "directory"
```

### Find by size

```bash
find . -size +1G
```