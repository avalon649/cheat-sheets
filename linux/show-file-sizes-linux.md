# Linux Commands to Show File Sizes in the Current Folder

This guide summarizes various Linux terminal commands and options to display file sizes for files and folders in the current directory (`.`).

---

## 1. List Files with Human-Readable Sizes

```bash
ls -lh
```
- `l`: long format
- `h`: human-readable (e.g., KB, MB, GB)

## 2. Show Only Regular Files (No Dirs) by Size, Descending

```bash
ls -lhS --group-directories-first
```
- `S`: sort by size
- `--group-directories-first`: puts folders at the top (optional)

## 3. Display Only File Sizes (Bytes/KB) with `stat`

```bash
stat -c "%s %n" *
```
- Shows size in bytes and filename.

## 4. Summarize Directory Sizes with `du`

```bash
du -sh *
```
- `s`: summarize (don't list files inside subdirs)
- `h`: human-readable

Show total size of the current directory:
```bash
du -sh .
```

## 5. List All Files (Recursive) with Size

```bash
find . -type f -exec du -h {} +
```

## 6. Sort Files by Actual Disk Usage (Largest First)

```bash
du -ah --max-depth=1 | sort -hr
```
- `a`: list all (files + dirs)
- `--max-depth=1`: only top-level directory
- `sort -hr`: sort human-readable, reverse (largest on top)

## 7. Show Only Largest Files/Folders (e.g., Top 10)

```bash
du -ah . | sort -hr | head -n 10
```

## 8. Alternative: List Only Files (No Directories)

```bash
find . -maxdepth 1 -type f -exec ls -lh {} +
```

---

## Quick Reference Table

| Command                     | Purpose                                 |
|----------------------------|-----------------------------------------|
| `ls -lh`                   | List files with human sizes             |
| `ls -lhS`                  | List files sorted by size               |
| `du -sh *`                 | Summarize size of each file & folder    |
| `du -sh .`                 | Show total size of the current folder   |
| `du -ah | sort -hr`        | Show all, sorted largest first          |
| `find . -type f`           | List only files                         |
| `stat -c "%s %n" *`      | Show size in bytes for each file        |

---

**Tip**: Some commands may require `sudo` for system folders or restricted files.
