# Linux Find Command Examples

## Basic Search

```bash
# Find by name (case-sensitive)
find /home/avalon -name "*.log"

# Find by name (case-insensitive)
find /home/avalon -iname "*.LOG"

# Find in current directory
find . -name "config.json"
```

## By Type

```bash
# Find only files
find /home/avalon -type f -name "*.sh"

# Find only directories
find /home/avalon -type d -name "backup*"

# Find symbolic links
find /home/avalon -type l
```

## By Size

```bash
# Files larger than 100MB
find /home/avalon -type f -size +100M

# Files smaller than 1KB
find /home/avalon -type f -size -1k

# Files exactly 512 bytes
find /home/avalon -type f -size 512c
```

## By Time

```bash
# Modified in last 7 days
find /home/avalon -mtime -7

# Modified more than 30 days ago
find /home/avalon -mtime +30

# Accessed in last 24 hours
find /home/avalon -atime -1

# Changed in last hour
find /home/avalon -cmin -60
```

## Depth Control

```bash
# Only current directory (no subdirectories)
find /home/avalon -maxdepth 1 -name "*.json"

# Between 2-4 levels deep
find /home/avalon -mindepth 2 -maxdepth 4 -type f
```

## Execute Commands

```bash
# Delete found files (use with caution!)
find /tmp -name "*.tmp" -delete

# Execute command on each file
find . -name "*.txt" -exec cat {} \;

# Execute with confirmation
find . -name "*.bak" -ok rm {} \;

# Use with xargs (more efficient)
find . -name "*.log" -print0 | xargs -0 gzip
```

## Combining Conditions

```bash
# AND (both conditions must match)
find . -name "*.sh" -type f

# OR (either condition matches)
find . -name "*.txt" -o -name "*.md"

# NOT (exclude pattern)
find . -type f ! -name "*.log"

# Complex combinations
find . \( -name "*.txt" -o -name "*.md" \) -size +1M
```

## Practical Examples

```bash
# Find all Python files modified today
find . -name "*.py" -mtime 0

# Find empty files
find /home/avalon -type f -empty

# Find empty directories
find /home/avalon -type d -empty

# Find files with specific permissions
find . -type f -perm 0644

# Find and count files
find . -type f -name "*.js" | wc -l

# Find files owned by specific user
find /home -user avalon -type f

# Find large log files and compress them
find /var/log -name "*.log" -size +50M -exec gzip {} \;

# Exclude certain directories
find . -type f -name "*.py" ! -path "*/node_modules/*" ! -path "*/.git/*"
```

## Size Units

- `c` - bytes
- `k` - kilobytes
- `M` - megabytes
- `G` - gigabytes

## Time Units

- `-mtime` - modification time (days)
- `-atime` - access time (days)
- `-ctime` - change time (days)
- `-mmin` - modification time (minutes)
- `-amin` - access time (minutes)
- `-cmin` - change time (minutes)

Use `-n` for "less than n", `+n` for "more than n", or `n` for "exactly n".
