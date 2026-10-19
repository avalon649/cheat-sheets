# Pacman Cheat Sheet

- **System:** Garuda/Arch, pacman **v7.1.0**
- **Repos:** `garuda`, `core`, `extra`, `multilib`, `chaotic-aur`
- **Extra tools installed:** `yay`, `pactree`, `paccache`, `checkupdates`, `pacdiff`, `expac`, `reflector`, `rate-mirrors`
- **Written:** 2026-10-19

---

## How pacman commands are built

Each command has **one operation** (capital letter) plus optional **modifiers** (lowercase):

| Operation | Meaning |
|---|---|
| `-S` | **Sync**: work with remote repos (install, search, upgrade) |
| `-Q` | **Query**: work with the local installed database |
| `-R` | **Remove**: uninstall packages |
| `-U` | **Upgrade**: install from a local file or URL |
| `-F` | **Files**: search the repos' file lists |
| `-D` | **Database**: change local database fields |
| `-T` | **Test**: check whether dependencies are satisfied |

```bash
pacman -h          # general help
pacman -S -h       # help for one operation
man pacman
man pacman.conf
```

---

## 1. Updating the system

```bash
sudo pacman -Syu          # refresh databases and upgrade everything (the normal update)
sudo pacman -Syyu         # force a full database refresh (after changing mirrors)
sudo pacman -Syuu         # also allow downgrades to match repo versions (after leaving testing)
checkupdates              # list pending updates safely, without touching the system DB
pacman -Qu                # list upgradable packages (uses the last sync)
```

On Garuda, `garuda-update` (alias `update`) wraps `-Syu` with snapshots, keyring and mirror refresh:

```bash
garuda-update                   # full update (refreshes mirrors and keyring first)
garuda-update --aur             # also update AUR packages via paru/yay
garuda-update remote keyring    # refresh keyrings only
garuda-update remote fix        # reset pacman.conf and refresh keyrings
garuda-update remote reinstall  # reinstall every package
```

> **Never do partial upgrades.** Avoid `pacman -Sy <pkg>`: refreshing the DB without upgrading can break library dependencies. Always use `-Syu` (optionally `sudo pacman -Syu <pkg>`).

---

## 2. Installing packages

```bash
sudo pacman -S <pkg>                     # install
sudo pacman -S <pkg1> <pkg2>             # install several
sudo pacman -S extra/<pkg>               # install from a specific repo
sudo pacman -S --needed <pkg1> <pkg2>    # skip packages that are already up to date
sudo pacman -S --asdeps <pkg>            # install marked as a dependency
sudo pacman -S --noconfirm <pkg>         # don't prompt (scripts only)
sudo pacman -Syu <pkg>                   # upgrade the system and install in one go (safe)
sudo pacman -S <group>                   # install a group, e.g. base-devel
sudo pacman -S $(pacman -Sgq <group>)    # install all members of a group, no prompt to select
```

Install from a local file or URL:

```bash
sudo pacman -U ./package-1.0-1-x86_64.pkg.tar.zst
sudo pacman -U https://example.com/package.pkg.tar.zst
sudo pacman -U /var/cache/pacman/pkg/<pkg>-<oldver>.pkg.tar.zst   # downgrade from cache
```

Download only, without installing:

```bash
sudo pacman -Sw <pkg>
```

---

## 3. Removing packages

```bash
sudo pacman -R <pkg>        # remove the package only
sudo pacman -Rs <pkg>       # also remove dependencies nothing else needs (recommended)
sudo pacman -Rns <pkg>      # same, plus delete .pacsave config backups
sudo pacman -Rc <pkg>       # remove the package AND everything that depends on it (careful)
sudo pacman -Rdd <pkg>      # remove ignoring dependency checks (dangerous)
sudo pacman -Rsu <pkg>      # skip targets other packages still need
```

Remove orphans (dependencies nothing needs any more):

```bash
pacman -Qdtq                              # list orphans
sudo pacman -Rns $(pacman -Qdtq)          # remove them (errors if there are none)
pacman -Qdtq | sudo pacman -Rns -         # same, reading from stdin
```

---

## 4. Searching and listing available packages (sync DB)

```bash
pacman -Sl                    # list ALL packages in all repos (repo name version [installed])
pacman -Sl extra              # list one repo
pacman -Slq                   # names only
pacman -Slq | wc -l           # count available packages
pacman -Ss <regex>            # search names and descriptions
pacman -Ssq <regex>           # search, names only
pacman -Ss '^python-'         # regex example
pacman -Si <pkg>              # detailed remote info (deps, size, repo, packager)
pacman -Sii <pkg>             # also show reverse dependencies
pacman -Sg                    # list all groups
pacman -Sg <group>            # list packages in a group
```

Packages in a repo that are not installed:

```bash
comm -23 <(pacman -Slq extra | sort) <(pacman -Qq | sort)
```

---

## 5. Querying installed packages (local DB)

```bash
pacman -Q                 # list all installed packages with versions
pacman -Qq                # names only
pacman -Qe                # explicitly installed
pacman -Qeq               # explicitly installed, names only
pacman -Qd                # installed as dependencies
pacman -Qdt               # orphans
pacman -Qm                # foreign packages (AUR / manual, not in any repo)
pacman -Qn                # native packages (from sync repos)
pacman -Qen               # explicit and native (good for reinstall lists)
pacman -Qu                # upgradable
pacman -Qs <regex>        # search installed packages
pacman -Qi <pkg>          # detailed local info (install date, reason, size, required by)
pacman -Qii <pkg>         # also list backup/config files and whether they were modified
pacman -Qg <group>        # installed packages in a group
pacman -Qc <pkg>          # show changelog (if the package ships one)
pacman -Qk <pkg>          # check that the package's files are present
pacman -Qkk <pkg>         # deep check (checksums, permissions, mtime)
pacman -Qkk 2>/dev/null | grep -v ' 0 altered'   # check the whole system
```

---

## 6. Files: which package owns what

```bash
pacman -Ql <pkg>              # list files installed by a package
pacman -Qlq <pkg>             # paths only
pacman -Ql <pkg> | grep bin/  # just its binaries
pacman -Qo /usr/bin/ls        # which installed package owns a file
pacman -Qo ls                 # also works with a command name in PATH
pacman -Qp ./x.pkg.tar.zst    # query a package file instead of the DB
pacman -Qlp ./x.pkg.tar.zst   # list files inside a package file
```

Search files in packages you have **not** installed (needs the files DB):

```bash
sudo pacman -Fy               # download/refresh the file databases (do this first)
pacman -F <filename>          # which repo package provides this file, e.g. pacman -F libssl.so
pacman -F /usr/bin/dig        # full path search
pacman -Fx '<regex>'          # regex search
pacman -Fl <pkg>              # list files of a remote package
```

Files on disk not owned by any package:

```bash
sudo find /usr /etc /opt -xdev -type f 2>/dev/null \
  | sort > /tmp/fs.txt
pacman -Qlq | sort -u > /tmp/pkg.txt
comm -23 /tmp/fs.txt /tmp/pkg.txt | less
```

---

## 7. Dependencies

```bash
pactree <pkg>                 # dependency tree
pactree -r <pkg>              # reverse: what depends on <pkg>
pactree -d 1 <pkg>            # direct dependencies only
pactree -s <pkg>              # use the sync DB (package not installed)
pacman -Qi <pkg> | grep -E 'Depends On|Required By|Optional'
pacman -Sp --print-format '%n %s' <pkg>   # what would be pulled in, without installing
pacman -T <dep>               # print deps that are NOT satisfied (no output means satisfied)
```

Change install reason:

```bash
sudo pacman -D --asexplicit <pkg>   # keep it when cleaning orphans
sudo pacman -D --asdeps <pkg>       # let orphan cleanup remove it when unneeded
```

---

## 8. Package cache (`/var/cache/pacman/pkg`)

```bash
du -sh /var/cache/pacman/pkg
paccache -d                   # dry run: what would be removed
sudo paccache -r              # keep the last 3 versions of each package (default)
sudo paccache -rk1            # keep only 1 version
sudo paccache -ruk0           # remove ALL cached versions of uninstalled packages
sudo pacman -Sc               # remove cached uninstalled packages and unused sync DBs
sudo pacman -Scc              # wipe the whole cache (no downgrades possible afterwards)
```

Automatic cleanup with a hook (`pacman-contrib` provides the timer):

```bash
sudo systemctl enable --now paccache.timer
```

---

## 9. Downgrading and rolling back

From the local cache:

```bash
ls /var/cache/pacman/pkg/ | grep <pkg>
sudo pacman -U /var/cache/pacman/pkg/<pkg>-<oldver>-x86_64.pkg.tar.zst
```

From the Arch Linux Archive:

```bash
sudo pacman -U https://archive.archlinux.org/packages/<first-letter>/<pkg>/<pkg>-<ver>-x86_64.pkg.tar.zst
```

Hold the package at that version in `/etc/pacman.conf`:

```ini
IgnorePkg   = <pkg>
IgnoreGroup = <group>
```

On Garuda (Btrfs + snapper), the easiest rollback is to boot a pre-update snapshot from the GRUB menu, or use `snapper-tools` / `btrfs-assistant`.

---

## 10. Mirrors

```bash
cat /etc/pacman.d/mirrorlist

# rate-mirrors (fast; Garuda default)
rate-mirrors --allow-root --protocol https arch | sudo tee /etc/pacman.d/mirrorlist

# reflector
sudo reflector --country Italy,Germany --protocol https --age 12 \
  --sort rate --latest 20 --save /etc/pacman.d/mirrorlist

sudo pacman -Syyu          # force refresh after changing mirrors
```

On Garuda, `garuda-update` refreshes the mirrorlist with rate-mirrors before each update. To skip that:

```bash
garuda-update --skip-mirrorlist
```

---

## 11. Keyring and signatures

```bash
sudo pacman -Sy archlinux-keyring && sudo pacman -Syu   # fix "invalid or corrupted package (PGP signature)"
sudo pacman-key --init                                   # initialise the keyring
sudo pacman-key --populate archlinux chaotic             # re-add distro keys
sudo pacman-key --refresh-keys                           # refresh from keyservers (slow)
sudo pacman-key --recv-keys <KEYID>                      # import a key
sudo pacman-key --lsign-key <KEYID>                      # locally trust it
pacman-key --list-keys | less
```

Last resort, a full keyring reset:

```bash
sudo rm -rf /etc/pacman.d/gnupg
sudo pacman-key --init
sudo pacman-key --populate archlinux chaotic
sudo pacman -Sy archlinux-keyring chaotic-keyring garuda-hooks
```

---

## 12. `/etc/pacman.conf` useful options

```ini
[options]
HoldPkg      = pacman glibc
Architecture = auto
CheckSpace
ParallelDownloads = 10      # faster downloads
Color                       # coloured output
VerbosePkgLists             # table of old/new versions on upgrade
ILoveCandy                  # Pac-Man progress bar
#IgnorePkg   = linux        # hold packages
#NoUpgrade   = etc/somefile # never overwrite these files
#NoExtract   = usr/share/locale/* !usr/share/locale/en*

SigLevel    = Required DatabaseOptional
LocalFileSigLevel = Optional

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

[multilib]
Include = /etc/pacman.d/mirrorlist

[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```

Check the parsed configuration:

```bash
pacman-conf                 # dump the effective config
pacman-conf --repo-list     # list configured repos
pacman-conf ParallelDownloads
```

---

## 13. Config file merges (`.pacnew` / `.pacsave`)

When an updated package ships a new config file and you've edited your copy, pacman writes the new one as `<file>.pacnew`. When you remove a package with a modified config, your copy is kept as `<file>.pacsave`.

```bash
sudo find /etc -name '*.pac*' 2>/dev/null   # find them
sudo pacdiff                                # interactive merge (uses vimdiff by default)
sudo DIFFPROG=meld pacdiff                  # use meld instead
sudo DIFFPROG='code --wait --diff' pacdiff  # use VS Code
```

---

## 14. Logs and history

```bash
less /var/log/pacman.log
grep -E '\[ALPM\] (installed|upgraded|removed)' /var/log/pacman.log | tail -50
grep -i 'installed <pkg>' /var/log/pacman.log
grep "$(date +%Y-%m-%d)" /var/log/pacman.log            # today's transactions
grep -E 'starting full system upgrade' /var/log/pacman.log | tail -5   # last updates
```

Recently installed packages, newest first (needs `expac`):

```bash
expac --timefmt='%F %T' '%l  %n' | sort -r | head -30
```

---

## 15. Backing up and restoring package lists

```bash
pacman -Qqen > pkglist-native.txt    # explicit repo packages
pacman -Qqem > pkglist-foreign.txt   # explicit AUR / foreign packages
```

Restore on a new system:

```bash
sudo pacman -S --needed - < pkglist-native.txt
yay -S --needed - < pkglist-foreign.txt
```

Remove anything not on the list (careful):

```bash
sudo pacman -Rsu $(comm -23 <(pacman -Qqe | sort) <(sort pkglist-native.txt pkglist-foreign.txt))
```

---

## 16. `expac`: custom package queries

```bash
expac -H M '%m\t%n' | sort -h | tail -20          # 20 largest installed packages
expac -H M -Q '%m\t%n' $(pacman -Qeq) | sort -h   # size of explicit packages
expac '%n %v' <pkg>                                # name and version
expac -S '%r/%n %v' <pkg>                          # remote repo/name and version
expac '%n: %D' <pkg>                               # dependencies
expac -Q '%n %w' | grep explicit                   # install reason
```

Total installed size:

```bash
expac -H M '%m' | awk '{s+=$1} END {print s " MiB"}'
```

---

## 17. AUR with `yay`

`yay` takes the same flags as pacman and adds AUR support.

```bash
yay                         # same as yay -Syu: update repos and AUR
yay <term>                  # interactive search and install
yay -S <pkg>                # install from repos or AUR
yay -Ss <term>              # search repos and AUR
yay -Si <pkg>               # info
yay -Sua                    # update AUR packages only
yay -Qua                    # list AUR updates
yay -Sl aur | less          # list all AUR packages
yay -Yc                     # remove unneeded dependencies
yay -Ps                     # system statistics
yay -G <pkg>                # download the PKGBUILD only
yay -S --editmenu <pkg>     # review or edit the PKGBUILD before building
yay -Sc                     # clean the yay build cache
```

Building manually from the AUR:

```bash
git clone https://aur.archlinux.org/<pkg>.git
cd <pkg>
less PKGBUILD               # ALWAYS review it
makepkg -si                 # build and install (-s pulls deps, -i installs)
```

---

## 18. Troubleshooting

### `unable to lock database`

Make sure no other pacman or yay is running first:

```bash
pgrep -a pacman || pgrep -a yay
sudo rm /var/lib/pacman/db.lck
```

### `failed to commit transaction (conflicting files)`

Find out who owns the conflicting file:

```bash
pacman -Qo /path/to/file
```

If no package owns it (a stray file), you can overwrite it, scoped to that path only:

```bash
sudo pacman -Syu --overwrite '/path/to/file'
```

### `invalid or corrupted package (PGP signature)`

```bash
sudo pacman -Sy archlinux-keyring chaotic-keyring && sudo pacman -Syu
```

### Corrupted download or cache

```bash
sudo rm /var/cache/pacman/pkg/<bad-file>.pkg.tar.zst
sudo pacman -Syu
```

### Reinstall every native package (repairs broken files)

```bash
sudo pacman -S $(pacman -Qnq)
```

### Find packages with missing or modified files

```bash
sudo pacman -Qkk 2>&1 | grep -v ', 0 altered files' | grep -v 'no mtree'
```

### Pacman itself is broken

Boot a live ISO, `arch-chroot` into the system, then use `pacman --sysroot /mnt` (or `pacman-static`).

### Show what a transaction would do, without doing it

```bash
sudo pacman -Syu --print            # list the URLs it would download
sudo pacman -Rns --print <pkg>      # list what would be removed
```

---

## 19. Handy one-liners

```bash
pacman -Qqe | wc -l                                   # count explicit packages
pacman -Qq | wc -l                                    # count all installed packages
pacman -Qm                                            # AUR/foreign packages (check after repo changes)
pacman -Qtdq                                          # orphans (alternative flag order)
pacman -Qi | awk '/^Name/{n=$3} /^Installed Size/{print $4$5, n}' | sort -h | tail -15
pacman -Sl chaotic-aur | grep -c installed            # installed packages from chaotic-aur
pacman -Sl | awk '/\[installed/ {print $1}' | sort | uniq -c   # installed packages per repo
pacman -Qkq 2>/dev/null                               # packages with missing files
LANG=C pacman -Sii <pkg> | grep 'Required By'         # reverse deps from the remote DB
```

---

## Quick reference card

| Task | Command |
|---|---|
| Update system | `sudo pacman -Syu` |
| Install | `sudo pacman -S <pkg>` |
| Install local file | `sudo pacman -U <file>` |
| Remove cleanly | `sudo pacman -Rns <pkg>` |
| Remove orphans | `sudo pacman -Rns $(pacman -Qdtq)` |
| Search repos | `pacman -Ss <term>` |
| Search installed | `pacman -Qs <term>` |
| List all available | `pacman -Sl` |
| List installed | `pacman -Q` |
| List explicit | `pacman -Qe` |
| List AUR/foreign | `pacman -Qm` |
| Package info | `pacman -Si <pkg>` / `pacman -Qi <pkg>` |
| Files of a package | `pacman -Ql <pkg>` |
| Owner of a file | `pacman -Qo <file>` |
| Find package by file | `pacman -F <file>` |
| Dependency tree | `pactree <pkg>` |
| Reverse deps | `pactree -r <pkg>` |
| Clean cache | `sudo paccache -r` |
| Pending updates | `checkupdates` |
| Merge .pacnew | `sudo pacdiff` |
| Fix keyring | `sudo pacman -Sy archlinux-keyring` |
| AUR update | `yay` |
