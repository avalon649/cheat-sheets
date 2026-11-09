# journalctl Cheat Sheet

> Written for systemd 262 (Arch / Garuda). Run `journalctl --help` or `man journalctl` to check the options on your installed version.

---

## Basics

```bash
journalctl                 # Whole journal, oldest first, opened in the pager
journalctl -e              # Jump to the end (the last page)
journalctl -r              # Newest entries first
journalctl -n 50           # Print the last 50 entries (the default is 10)
journalctl -n +20          # Print the first 20 entries
journalctl -f              # Follow live, like tail -f
journalctl -f -n 100       # Print the last 100 entries, then keep following
journalctl --no-pager      # Print straight to stdout
journalctl -x              # Add catalog explanations to entries where available
journalctl -xe             # Common combo: explanations, starting at the end
journalctl -q              # Hide info messages and the privilege warning
```

### Permissions

```bash
sudo journalctl                       # Full system journal
groups                                # Check whether you're in wheel or systemd-journal
sudo usermod -aG systemd-journal $USER  # Read the system journal without sudo (log in again afterwards)
```

---

## Pager keys (less)

```text
G           Jump to the end
g           Jump to the start
/pattern    Search forward
?pattern    Search backward
n / N       Next / previous match
F           Follow mode (Ctrl+C to stop)
←  →        Scroll sideways when lines are long
q           Quit
```

```bash
SYSTEMD_LESS=FRXMK journalctl         # Wrap long lines instead of cutting them off (drops the S flag)
journalctl --no-pager | less -R       # Use your own pager
```

---

## Boots

```bash
journalctl --list-boots     # List recorded boots with their index and ID
journalctl -b               # Current boot
journalctl -b -1            # Previous boot
journalctl -b -2            # The boot before that
journalctl -b <BOOT_ID>     # A specific boot, by ID
journalctl -b -1 -e         # End of the previous boot (handy after a crash)
journalctl -b -1 -p err     # Errors from the previous boot
```

> If `--list-boots` shows only the current boot, logs aren't being kept on disk. See [Persistent storage](#persistent-storage).

---

## Time filtering

```bash
journalctl --since "2026-11-09 08:00"
journalctl --since "2026-11-09" --until "2026-11-09 12:00"
journalctl --since today
journalctl --since yesterday --until today
journalctl --since "1 hour ago"
journalctl --since "-30m"
journalctl -S -2h -U -1h            # Short forms of --since and --until
journalctl --since "10:00" --until "10:15"
journalctl --utc --since "08:00"    # Read and show times in UTC
```

---

## Units and services

```bash
journalctl -u sshd                  # One system unit
journalctl -u sshd -u NetworkManager  # Several units together
journalctl -u 'docker*'             # Glob pattern
journalctl -u sddm -b -e            # One unit, this boot, starting at the end
journalctl -u nginx -f              # Follow one unit live
journalctl -u nginx -I              # Only the unit's latest run (since its last start)
journalctl -u nginx --list-invocations  # List the unit's runs (invocation IDs)
journalctl --invocation=<ID>        # Logs from one run

journalctl --user                   # Your user journal
journalctl --user-unit=pipewire     # One user unit
journalctl --user -u plasma-kwin_wayland -b
```

---

## Priority / severity

| Value | Name    | Meaning               |
|-------|---------|-----------------------|
| 0     | emerg   | System is unusable    |
| 1     | alert   | Act immediately       |
| 2     | crit    | Critical              |
| 3     | err     | Error                 |
| 4     | warning | Warning               |
| 5     | notice  | Normal but significant |
| 6     | info    | Informational         |
| 7     | debug   | Debug                 |

```bash
journalctl -p err               # err and anything more severe (0–3)
journalctl -p warning -b        # Warnings and worse, this boot
journalctl -p 3                 # Same as -p err
journalctl -p warning..err      # Only warning through err
journalctl -p err -b -1         # Errors from the previous boot
journalctl --facility=auth      # Filter by syslog facility
journalctl --facility=help      # List facility names
```

---

## Kernel messages

```bash
journalctl -k                   # Kernel messages, current boot (like dmesg)
journalctl -k -b -1             # Kernel messages, previous boot
journalctl -k -p err            # Kernel errors only
journalctl -k -f                # Follow kernel messages live
journalctl -k -g "usb|nvme"     # Kernel messages matching a regex
```

---

## Searching messages (grep)

```bash
journalctl -g "failed"                  # Messages matching a regex (PCRE2)
journalctl -g "fail|error|denied" -b    # Several patterns, this boot
journalctl -g "timeout" -u sshd         # Search one unit
journalctl -g "OOM" --case-sensitive    # Force case-sensitive matching
journalctl -g "(?i)segfault"            # Force case-insensitive matching
```

> If the pattern is all lowercase, matching ignores case. If it has any uppercase letters, matching is case-sensitive.

---

## Filter by identifier, process or user

```bash
journalctl -t sudo                       # By syslog identifier
journalctl -t kwin_wayland -t kded6      # Several identifiers
journalctl -T systemd                    # Hide an identifier
journalctl _PID=1234                     # By process ID
journalctl _UID=1000                     # By user ID
journalctl _COMM=sshd                    # By command name
journalctl _EXE=/usr/bin/sddm            # By executable path
journalctl /usr/bin/sddm                 # Shortcut: give the executable path directly
journalctl _SYSTEMD_UNIT=sshd.service    # Raw unit field
journalctl _TRANSPORT=kernel             # By transport (kernel, syslog, journal, stdout, audit)
journalctl _HOSTNAME=myhost
journalctl /dev/sda                      # Entries about a device node
```

### Combining matches

```bash
journalctl _COMM=sshd _UID=0                  # Different fields: AND
journalctl _COMM=sshd _COMM=sudo              # Same field twice: OR
journalctl _COMM=sshd + _COMM=sudo _UID=1000  # "+" means OR between groups
```

---

## Discovering fields

```bash
journalctl -N                   # List all field names in use
journalctl -F _SYSTEMD_UNIT     # List every value a field takes
journalctl -F _COMM
journalctl -F SYSLOG_IDENTIFIER
journalctl -F PRIORITY
journalctl -o verbose -n 1      # Show every field of the newest entry
```

---

## Output formats

```bash
journalctl -o short             # Default, syslog-style
journalctl -o short-precise     # Timestamps to the microsecond
journalctl -o short-iso         # ISO 8601 timestamps
journalctl -o short-iso-precise
journalctl -o short-full        # Full date and weekday
journalctl -o short-monotonic   # Time since boot
journalctl -o short-unix        # Unix epoch timestamps
journalctl -o with-unit         # Show the unit name instead of the identifier
journalctl -o cat               # Message text only
journalctl -o verbose           # Every field
journalctl -o json              # One JSON object per line
journalctl -o json-pretty       # Indented JSON
journalctl -o json-sse          # Server-sent events
journalctl -o json-seq          # RFC 7464 JSON sequence
journalctl -o export            # Binary-safe export format
```

```bash
journalctl -o json --output-fields=MESSAGE,_PID,PRIORITY -n 5
journalctl -W                   # Hide the hostname column
journalctl -a                   # Show long and unprintable fields in full
journalctl --no-full            # Shorten long fields with "…"
journalctl --truncate-newline   # Show only the first line of multi-line messages
```

### With jq

```bash
journalctl -u sshd -o json -n 20 | jq -r '.MESSAGE'
journalctl -b -p err -o json | jq -r '"\(._SYSTEMD_UNIT // .SYSLOG_IDENTIFIER): \(.MESSAGE)"'
journalctl -b -o json | jq -r '._SYSTEMD_UNIT // empty' | sort | uniq -c | sort -rn | head
```

---

## Disk usage and cleanup

```bash
journalctl --disk-usage                 # Total size of the journal files
sudo journalctl --vacuum-size=500M      # Delete old files until the total is under 500M
sudo journalctl --vacuum-time=2weeks    # Delete files older than two weeks
sudo journalctl --vacuum-files=5        # Keep only the 5 newest archived files
sudo journalctl --rotate                # Archive the active files now
sudo journalctl --rotate --vacuum-time=1d   # Rotate first, then vacuum (so the active file counts too)
```

---

## Persistent storage

```bash
ls /var/log/journal                     # If this directory exists, logs survive reboots
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo journalctl --flush                 # Move logs from /run into /var
```

### Size limits: `/etc/systemd/journald.conf.d/size.conf`

```ini
[Journal]
Storage=persistent
SystemMaxUse=1G
SystemKeepFree=2G
SystemMaxFileSize=100M
MaxRetentionSec=1month
RuntimeMaxUse=100M
```

```bash
sudo systemctl restart systemd-journald
systemd-analyze cat-config systemd/journald.conf   # Show the merged config
```

---

## Maintenance and integrity

```bash
sudo journalctl --verify                # Check the journal files for corruption
sudo journalctl --sync                  # Write pending entries to disk
sudo journalctl --header                # Show journal file header info
journalctl --list-namespaces
journalctl --namespace=NAME
sudo journalctl --setup-keys            # Generate Forward Secure Sealing keys
```

---

## Other journal sources

```bash
journalctl -D /mnt/var/log/journal      # Journal directory from another system or a rescue mount
journalctl --root=/mnt                  # Use another root filesystem
journalctl --file=/path/system.journal  # One journal file
journalctl -m                           # Merge all available journals
journalctl -M mycontainer               # A local container (systemd-nspawn)
```

---

## Cursors (incremental reading)

```bash
journalctl -n 1 --show-cursor                     # Print the cursor after the output
journalctl --after-cursor="s=...;i=..."           # Everything after that point
journalctl --cursor-file=/tmp/jcursor -u sshd     # Read from the saved point and update it (for scripts)
```

---

## Writing to the journal

```bash
logger "hello from logger"                        # syslog-style message
logger -t myscript -p user.err "something broke"  # With an identifier and priority
systemd-cat -t backup echo "backup started"       # Send a command's output to the journal
systemd-cat -t backup -p warning ./script.sh
echo "test" | systemd-cat -t test
journalctl -t backup -e                           # Read it back
```

---

## Message catalog

```bash
journalctl --list-catalog       # All message IDs with explanations
journalctl --dump-catalog       # Full catalog text
journalctl -x -u sshd           # Show catalog help inline
```

---

## Handy recipes

```bash
# What went wrong this boot?
journalctl -b -p err --no-pager

# Why did the last boot end or crash?
journalctl -b -1 -e

# A unit's logs since its last restart
journalctl -u myservice -I

# Count errors per unit this boot
journalctl -b -p err -o json | jq -r '._SYSTEMD_UNIT // .SYSLOG_IDENTIFIER' | sort | uniq -c | sort -rn

# Failed SSH logins today
journalctl -u sshd --since today -g "Failed password|Invalid user"

# sudo usage
journalctl _COMM=sudo --since "1 week ago"

# OOM killer events
journalctl -k -g "out of memory|oom-kill|Killed process"

# GPU / driver problems
journalctl -k -b -g "amdgpu|nvidia|drm" -p warning

# Suspend and resume
journalctl -b -g "suspend|resume|PM:" -o short-precise

# Units that failed
systemctl --failed
journalctl -b -g "Failed to start"

# Export a window of logs to a file
journalctl --since "1 hour ago" --no-pager > ~/journal-last-hour.log

# Follow several units at once
journalctl -f -u sshd -u NetworkManager -u docker

# Follow everything at warning level or worse
journalctl -f -p warning

# Which processes log the most?
journalctl -b -o json | jq -r '.SYSLOG_IDENTIFIER // empty' | sort | uniq -c | sort -rn | head -20
```

---

## Related commands

```bash
systemctl status sshd           # Unit status plus its last few log lines
systemctl status sshd -n 50     # Same, with 50 log lines
systemctl --failed              # Units that failed
coredumpctl list                # Crash dumps
coredumpctl info <PID>          # Details and stack trace for one crash
dmesg -T                        # Kernel ring buffer with readable timestamps
loginctl                        # Login sessions
```
