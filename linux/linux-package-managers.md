# Linux Distributions and Their Package Managers

## Debian-based

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Debian | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Ubuntu | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Linux Mint | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Pop!_OS | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Elementary OS | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Zorin OS | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| KDE Neon | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| MX Linux | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Kali Linux | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Parrot OS | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Raspberry Pi OS | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Tails | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Deepin | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| AntiX | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Devuan | dpkg / APT | `apt`, `apt-get`, `dpkg` |
| Peppermint OS | dpkg / APT | `apt`, `apt-get`, `dpkg` |

## Red Hat-based

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| RHEL (Red Hat Enterprise Linux) | RPM / DNF | `dnf`, `yum`, `rpm` |
| Fedora | RPM / DNF | `dnf`, `rpm` |
| CentOS | RPM / DNF (or YUM) | `dnf`, `yum`, `rpm` |
| CentOS Stream | RPM / DNF | `dnf`, `rpm` |
| Rocky Linux | RPM / DNF | `dnf`, `rpm` |
| AlmaLinux | RPM / DNF | `dnf`, `rpm` |
| Oracle Linux | RPM / DNF | `dnf`, `yum`, `rpm` |
| Amazon Linux | RPM / DNF (or YUM) | `dnf`, `yum`, `rpm` |
| Scientific Linux | RPM / YUM | `yum`, `rpm` |
| ClearOS | RPM / YUM | `yum`, `rpm` |

## SUSE-based

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| openSUSE Leap | RPM / Zypper | `zypper`, `rpm` |
| openSUSE Tumbleweed | RPM / Zypper | `zypper`, `rpm` |
| SUSE Linux Enterprise (SLES) | RPM / Zypper | `zypper`, `rpm` |
| GeckoLinux | RPM / Zypper | `zypper`, `rpm` |

## Arch-based

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Arch Linux | Pacman | `pacman` |
| Manjaro | Pacman | `pacman`, `pamac` |
| EndeavourOS | Pacman | `pacman`, `yay` |
| Garuda Linux | Pacman | `pacman`, `paru` |
| ArcoLinux | Pacman | `pacman`, `yay` |
| Artix Linux | Pacman | `pacman` |
| BlackArch | Pacman | `pacman` |
| Parabola | Pacman | `pacman` |
| Hyperbola | Pacman | `pacman` |
| CachyOS | Pacman | `pacman`, `paru` |

## Gentoo-based

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Gentoo | Portage | `emerge` |
| Funtoo | Portage | `emerge` |
| Calculate Linux | Portage | `emerge` |
| Sabayon | Portage / Entropy | `emerge`, `equo` |
| Redcore Linux | Portage / Sisyphus | `sisyphus` |

## Slackware-based

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Slackware | pkgtools | `installpkg`, `removepkg`, `slackpkg` |
| Salix OS | pkgtools / slapt-get | `slapt-get`, `slapt-src` |
| Slackel | pkgtools | `slackpkg` |
| Porteus | pkgtools | `slackpkg` |
| Zenwalk | pkgtools / netpkg | `netpkg` |

## Alpine-based

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Alpine Linux | apk | `apk` |
| Postmarket OS | apk | `apk` |
| Adélie Linux | apk | `apk` |

## Void Linux

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Void Linux | XBPS | `xbps-install`, `xbps-remove`, `xbps-query` |

## Solus

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Solus | eopkg | `eopkg` |

## NixOS

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| NixOS | Nix | `nix-env`, `nix` |

## Clear Linux

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Clear Linux | swupd | `swupd` |

## Mageia

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Mageia | RPM / urpmi | `urpmi`, `dnf`, `rpm` |

## PCLinuxOS

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| PCLinuxOS | RPM / APT-RPM | `apt-get`, `synaptic`, `rpm` |

## Puppy Linux

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Puppy Linux | PETget | `petget`, `ppm` |

## Tiny Core Linux

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Tiny Core Linux | tce | `tce-load` |

## Guix System

| Distribution | Package Manager | Command-line Tool |
|--------------|-----------------|-------------------|
| Guix System | Guix | `guix` |

---

## Universal Package Managers

These work across multiple distributions:

| Package Manager | Format | Command |
|-----------------|--------|---------|
| Flatpak | Flatpak | `flatpak` |
| Snap | Snap | `snap` |
| AppImage | AppImage | Self-contained executables |
| Homebrew (Linuxbrew) | Various | `brew` |
| Nix | Nix | `nix-env` |

---

## Quick Reference: Common Commands

| Action | APT (Debian/Ubuntu) | DNF (Fedora/RHEL) | Pacman (Arch) | Zypper (openSUSE) | apk (Alpine) |
|--------|---------------------|-------------------|---------------|-------------------|--------------|
| Update repos | `apt update` | `dnf check-update` | `pacman -Sy` | `zypper refresh` | `apk update` |
| Upgrade all | `apt upgrade` | `dnf upgrade` | `pacman -Syu` | `zypper update` | `apk upgrade` |
| Install | `apt install pkg` | `dnf install pkg` | `pacman -S pkg` | `zypper install pkg` | `apk add pkg` |
| Remove | `apt remove pkg` | `dnf remove pkg` | `pacman -R pkg` | `zypper remove pkg` | `apk del pkg` |
| Search | `apt search pkg` | `dnf search pkg` | `pacman -Ss pkg` | `zypper search pkg` | `apk search pkg` |
| Info | `apt show pkg` | `dnf info pkg` | `pacman -Si pkg` | `zypper info pkg` | `apk info pkg` |
