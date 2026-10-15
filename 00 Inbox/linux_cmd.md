---
title: Linux Command Reference
tags: [linux, cli, reference, cheatsheet]
created: 2026-10-15
---
```
insta-toc
```
# Linux Command Reference

Categorised list of Linux/Unix commands with brief descriptions. `cmd [args]` marks commands that usually need arguments. Paths like `/proc` are pseudo-filesystems.

Notation used below:

- `dir` — directory path
- `file` — file path
- `pat` — pattern/regex
- `host` — hostname or IP
- `pid` — process ID

---

## Table of Contents

- [[#1. File & Directory Management|1. File & Directory Management]]
- [[#2. File Viewing & Editing|2. File Viewing & Editing]]
- [[#3. Text Processing|3. Text Processing]]
- [[#4. Searching & Finding|4. Searching & Finding]]
- [[#5. File Permissions & Ownership|5. File Permissions & Ownership]]
- [[#6. Archiving & Compression|6. Archiving & Compression]]
- [[#7. Process Management|7. Process Management]]
- [[#8. System Information & Monitoring|8. System Information & Monitoring]]
- [[#9. User & Group Management|9. User & Group Management]]
- [[#10. Package Management|10. Package Management]]
    - [[#Debian / Ubuntu (APT)|Debian / Ubuntu (APT)]]
    - [[#Fedora / RHEL (DNF / YUM)|Fedora / RHEL (DNF / YUM)]]
    - [[#Arch (Pacman)|Arch (Pacman)]]
    - [[#Other|Other]]
- [[#11. Networking|11. Networking]]
- [[#12. Disk & Filesystem|12. Disk & Filesystem]]
- [[#13. systemd, Services & Boot|13. systemd, Services & Boot]]
- [[#14. Scheduling & Automation|14. Scheduling & Automation]]
- [[#15. Shell, Environment & Scripting|15. Shell, Environment & Scripting]]
- [[#16. Security, Encryption & Auditing|16. Security, Encryption & Auditing]]
- [[#17. Logs & Diagnostics|17. Logs & Diagnostics]]
- [[#18. Terminals, Sessions & Remote|18. Terminals, Sessions & Remote]]
- [[#19. File Transfer & Sync|19. File Transfer & Sync]]
- [[#20. Multimedia & Documents|20. Multimedia & Documents]]
- [[#21. Printing, Hardware & Peripherals|21. Printing, Hardware & Peripherals]]
- [[#22. Kernel & Boot|22. Kernel & Boot]]
- [[#23. Containers & Virtualisation|23. Containers & Virtualisation]]
- [[#24. Development & Build Tools|24. Development & Build Tools]]
- [[#25. Miscellaneous Utilities|25. Miscellaneous Utilities]]
- [[#Handy One-Liners|Handy One-Liners]]
- [[#Signal Reference|Signal Reference]]
- [[#Exit Status Reference|Exit Status Reference]]
- [[#Permission Bit Quick Table|Permission Bit Quick Table]]
- [[#Useful Paths|Useful Paths]]

---


## 1. File & Directory Management

| Command                 | Description                                          |
| ----------------------- | ---------------------------------------------------- |
| `ls`                    | List directory contents                              |
| `ls -la`                | Long listing, including dotfiles (hidden)            |
| `cd dir`                | Change current working directory                     |
| `pwd`                   | Print working (current) directory                    |
| `mkdir dir`             | Create a directory                                   |
| `mkdir -p a/b/c`        | Create nested parents as needed                      |
| `rmdir dir`             | Remove an empty directory                            |
| `rm file`               | Delete a file                                        |
| `rm -r dir`             | Delete a directory and its contents recursively      |
| `rm -rf dir`            | Force recursive delete, **no prompt — irreversible** |
| `cp src dst`            | Copy a file or directory tree                        |
| `cp -a src dst`         | Archive copy: recursive, preserves perms/times/links |
| `mv src dst`            | Move or rename a file/directory                      |
| `touch file`            | Create empty file / update timestamps                |
| `ln -s target link`     | Create a symbolic (soft) link                        |
| `ln target link`        | Create a hard link                                   |
| `install -m755 src dst` | Copy and set mode/owner (atomic via `install f f.new && mv`) |
| `basename path`         | Strip directory, keep filename                       |
| `dirname path`          | Strip filename, keep directory                       |
| `realpath path`         | Resolve to canonical absolute path                   |
| `readlink -f path`      | Resolve symlinks to absolute target                  |
| `tree dir`              | Display directory structure as a tree (needs `tree`) |
| `file file`             | Identify file type by content signature              |
| `stat file`             | Show detailed inode/metadata/timestamps              |
| `mktemp`                | Create a unique temporary file/directory             |
| `shred -u file`         | Overwrite a file before deleting (best-effort)       |
| `truncate -s 0 file`    | Zero out a file's content in place                   |
| `rename 's/a/b/' *.txt` | Bulk-rename via Perl regex (Debian; else `perl-rename`) |
| `find dir`              | Recursively list paths (see Finding Files)           |

## 2. File Viewing & Editing

| Command                 | Description                                        |
| ----------------------- | -------------------------------------------------- |
| `cat file`              | Print entire file to stdout                        |
| `cat -n file`           | Print file with line numbers                       |
| `less file`             | Pager: scroll/search without loading all (q quits) |
| `more file`             | Minimal pager                                      |
| `head -n 20 file`       | Show first 20 lines                                |
| `tail -n 20 file`       | Show last 20 lines                                 |
| `tail -f file`          | Follow a growing file in real time                 |
| `tail -F file`          | Follow and survive log rotation                    |
| `nl file`               | Number non-blank lines                             |
| `tac file`              | Print file in reverse line order                   |
| `rev file`              | Reverse characters of each line                    |
| `od -c file`            | Octal/char dump for binary inspection              |
| `hexdump -C file`       | Canonical hex+ASCII dump                           |
| `xxd file`              | Hex dump (also reverse with `-r`)                  |
| `strings binary`        | Extract printable strings from a binary            |
| `column -t`             | Align input into neat columns                      |
| `vi` / `vim file`       | Modal text editor                                  |
| `nano file`             | Simple, beginner-friendly editor                   |
| `emacs file`            | Extensible editor                                  |
| `ed file`               | Classic line editor (scriptable)                   |
| `sed -i 's/a/b/g' file` | Edit in place (see Text Processing)                |
| `dos2unix file`         | Convert CRLF → LF line endings                     |
| `unix2dos file`         | Convert LF → CRLF line endings                     |

## 3. Text Processing

| Command                        | Description                                     |
| ------------------------------ | ----------------------------------------------- |
| `sed 's/old/new/' file`        | Stream editor; replace first match per line     |
| `sed 's/old/new/g' file`       | Replace all matches per line                    |
| `sed -n '5,10p' file`          | Print lines 5–10                                |
| `sed -i`                       | Edit files in place (GNU)                       |
| `awk '{print $1}' file`        | Field-based extraction; `$1` = first column     |
| `awk -F: '{print $1}'`         | Use `:` as field separator                      |
| `awk '$3>100 {print}'`         | Filter rows by a numeric condition              |
| `cut -d: -f1,3 file`           | Select fields by delimiter                      |
| `paste f1 f2`                  | Merge files side by side                        |
| `join f1 f2`                   | Relational join on a common field               |
| `sort file`                    | Sort lines alphabetically                       |
| `sort -n`                      | Numeric sort                                    |
| `sort -h`                      | Human-readable size sort (1K, 2M…)              |
| `sort -u`                      | Sort and deduplicate                            |
| `sort -k2 -t:`                 | Sort by field 2 using `:` separator             |
| `uniq`                         | Collapse adjacent duplicate lines               |
| `uniq -c`                      | Prefix lines with occurrence counts             |
| `wc -l file`                   | Count lines                                     |
| `wc -w` / `wc -c`              | Count words / bytes                             |
| `tr 'a-z' 'A-Z'`               | Translate or delete characters                  |
| `tr -d '\r'`                   | Delete carriage returns                         |
| `grep pat file`                | Print matching lines                            |
| `grep -r pat dir`              | Recursive search                                |
| `grep -i` / `-v`               | Case-insensitive / invert match                 |
| `grep -E`                      | Extended regex (egrep)                          |
| `grep -o`                      | Print only the matched part                     |
| `grep -c`                      | Count matching lines                            |
| `grep -A3 -B3`                 | Show 3 lines of context after/before            |
| `fgrep`                        | Fixed-string (no regex) grep                    |
| `diff a b`                     | Show line differences between files             |
| `diff -u a b`                  | Unified diff (patch format)                     |
| `sdiff a b`                    | Side-by-side differences                        |
| `comm a b`                     | Common/unique lines of two sorted files         |
| `patch < p.diff`               | Apply a diff                                    |
| `cmp a b`                      | Byte-level comparison                           |
| `md5sum file`                  | MD5 checksum                                    |
| `sha256sum file`               | SHA-256 checksum                                |
| `sha256sum -c sums.txt`        | Verify checksums from a file                    |
| `cksum file`                   | CRC checksum and byte count                     |
| `shuf file`                    | Randomly shuffle lines                          |
| `fold -w 80`                   | Wrap lines to 80 columns                        |
| `fmt`                          | Reformat/rewrap paragraphs                      |
| `expand` / `unexpand`          | Tabs ↔ spaces                                   |
| `xargs cmd`                    | Build and run command lines from stdin          |
| `xargs -0`                     | NUL-delimited input (pairs with `find -print0`) |
| `tee file`                     | Write stdout to file **and** terminal           |
| `split -l 1000 f part_`        | Split a file into chunks                        |
| `csplit`                       | Split by pattern                                |
| `aspell check file`            | Spell checker                                   |
| `iconv -f UTF-8 -t ISO-8859-1` | Convert text encodings                          |

## 4. Searching & Finding

| Command                        | Description                               |
| ------------------------------ | ----------------------------------------- |
| `find dir -name "*.log"`       | Find by filename pattern                  |
| `find dir -iname`              | Case-insensitive name match               |
| `find dir -type f` / `-type d` | Restrict to files / directories           |
| `find dir -size +100M`         | Find files larger than 100 MB             |
| `find dir -mtime -7`           | Modified within the last 7 days           |
| `find dir -perm -4000`         | Find SUID executables                     |
| `find dir -user em`            | Owned by a given user                     |
| `find dir -empty`              | Empty files/directories                   |
| `find dir -delete`             | Delete matches (dangerous)                |
| `find dir -exec cmd {} \;`     | Run a command per result                  |
| `find dir -exec cmd {} +`      | Batch results into one invocation         |
| `locate pat`                   | Fast filename lookup via `updatedb` index |
| `updatedb`                     | Rebuild the locate database               |
| `which cmd`                    | Show the resolved path of a command       |
| `type cmd`                     | Show how the shell resolves a name        |
| `whereis cmd`                  | Locate binary, source, and man page       |
| `command -v cmd`               | POSIX-portable "is this runnable?"        |
| `grep -r` / `rg`               | Content search (ripgrep is much faster)   |
| `fd pat`                       | Fast, user-friendly alternative to `find` |
| `fzf`                          | Fuzzy finder for interactive filtering    |

## 5. File Permissions & Ownership

| Command                     | Description                                 |
| --------------------------- | ------------------------------------------- |
| `chmod 755 file`            | Set mode in octal (`rwxr-xr-x`)             |
| `chmod u+x file`            | Add execute for owner (symbolic)            |
| `chmod -R g-w dir`          | Recursively remove group write              |
| `chown user:group file`     | Change owner and group                      |
| `chown -R em:em dir`        | Recursive ownership change                  |
| `chgrp group file`          | Change group only                           |
| `umask 022`                 | Set default permission mask for new files   |
| `getfacl file`              | Show POSIX ACLs                             |
| `setfacl -m u:bob:rw file`  | Grant per-user ACL entry                    |
| `lsattr` / `chattr +i file` | Show/set ext4 attributes (`+i` = immutable) |
| `chattr +a file`            | Append-only attribute                       |
| `sudo -l`                   | List your sudo privileges                   |
| `sudo -u user cmd`          | Run a command as another user               |
| `su - user`                 | Switch user with a login shell              |
| `visudo`                    | Safely edit `/etc/sudoers`                  |
| `groups` / `groups user`    | List group memberships                      |
| `newgrp group`              | Start a shell in a new primary group        |

## 6. Archiving & Compression

| Command                        | Description                               |
| ------------------------------ | ----------------------------------------- |
| `tar -cvf a.tar dir`           | Create a tar archive, verbose             |
| `tar -xvf a.tar`               | Extract a tar archive                     |
| `tar -tvf a.tar`               | List contents without extracting          |
| `tar -czf a.tar.gz dir`        | Create gzip-compressed tar (`.tgz`)       |
| `tar -xzf a.tar.gz`            | Extract a gzip tar                        |
| `tar -cjf a.tar.bz2 dir`       | bzip2-compressed tar                      |
| `tar -cJf a.tar.xz dir`        | xz-compressed tar (best ratio, slow)      |
| `tar -xzf a.tgz -C /dst`       | Extract into a target directory           |
| `gzip file` / `gunzip file.gz` | Compress / decompress                     |
| `bzip2` / `bunzip2`            | Block-sorting compressor                  |
| `xz` / `unxz`                  | High-ratio compressor                     |
| `zstd file` / `unzstd`         | Fast modern compressor                    |
| `lz4`                          | Extremely fast compressor                 |
| `zip -r a.zip dir`             | Create a zip archive                      |
| `unzip a.zip`                  | Extract a zip                             |
| `unzip -l a.zip`               | List zip contents                         |
| `7z a a.7z dir`                | Create a 7-Zip archive (`p7zip`)          |
| `7z x a.7z`                    | Extract with full paths                   |
| `zcat file.gz`                 | Print decompressed content to stdout      |
| `zless` / `zmore`              | Page through a gzip file                  |
| `cpio`                         | Copy files to/from archives (initramfs)   |
| `ar`                           | Create/manipulate static libraries (`.a`) |
| `gpg -c file`                  | Symmetric-encrypt a file                  |
| `openssl enc -aes-256-cbc`     | OpenSSL file encryption                   |

## 7. Process Management

| Command                      | Description                                   |
| ---------------------------- | --------------------------------------------- |
| `ps aux`                     | Snapshot of all running processes (BSD style) |
| `ps -ef`                     | Snapshot, full-format (SysV style)            |
| `ps -ef --forest`            | Show the process tree                         |
| `top`                        | Live process and resource monitor             |
| `htop`                       | Friendlier interactive process viewer         |
| `btop`                       | Modern resource monitor                       |
| `atop`                       | Historical resource accounting                |
| `pgrep pat`                  | Find PIDs by name/pattern                     |
| `pkill pat`                  | Signal processes by name                      |
| `pkill -f "cmd args"`        | Match against the full command line           |
| `kill pid`                   | Send SIGTERM (polite terminate)               |
| `kill -9 pid`                | Send SIGKILL (cannot be caught)               |
| `killall name`               | Signal all processes by exact name            |
| `nice -n 10 cmd`             | Start with lower CPU priority                 |
| `renice -n -5 -p pid`        | Change priority of a running process          |
| `nohup cmd &`                | Run immune to hangup, keep after logout       |
| `cmd &`                      | Run in background                             |
| `jobs` / `fg` / `bg`         | List / foreground / background shell jobs     |
| `Ctrl-Z` then `bg`           | Suspend then resume in background             |
| `disown`                     | Detach a job from the shell                   |
| `wait`                       | Wait for background jobs to finish            |
| `time cmd`                   | Measure real/user/sys execution time          |
| `watch -n2 cmd`              | Re-run a command every 2 s                    |
| `strace -p pid`              | Trace syscalls of a process                   |
| `lsof -p pid`                | List files/sockets a process holds            |
| `fuser -k dir`               | Kill processes using a directory              |
| `pmap pid`                   | Show a process's memory map                   |
| `ionice`                     | Set I/O scheduling class/priority             |
| `taskset -c 0,1 cmd`         | Pin a process to CPU cores                    |
| `systemd-run --scope cmd`    | Run a command in a transient unit             |
| `setsid cmd`                 | Run in a new session (detach from tty)        |
| `ulimit -a`                  | Show resource limits for the shell            |
| `cgroups`/`systemctl status` | Inspect cgroup resource control               |

## 8. System Information & Monitoring

| Command               | Description                           |
| --------------------- | ------------------------------------- |
| `uname -a`            | Kernel name, version, architecture    |
| `hostnamectl`         | Hostname, OS, kernel, virtualization  |
| `uptime`              | Uptime and load averages              |
| `whoami` / `id`       | Current user / UID, GID, groups       |
| `who` / `w`           | Logged-in users / with activity       |
| `last` / `lastb`      | Login history / failed logins         |
| `date`                | Show or set the system date/time      |
| `timedatectl`         | Timezone, NTP sync status             |
| `cal`                 | Calendar                              |
| `free -h`             | Memory and swap usage                 |
| `vmstat 1`            | Virtual memory, CPU, I/O stats        |
| `iostat -x 1`         | Per-device I/O statistics (`sysstat`) |
| `mpstat -P ALL 1`     | Per-CPU utilisation                   |
| `sar`                 | Historical system activity (sysstat)  |
| `pidstat`             | Per-process CPU/memory/I/O stats      |
| `lscpu`               | CPU topology and features             |
| `lspci`               | List PCI devices                      |
| `lsusb`               | List USB devices                      |
| `lsblk`               | Block devices in a tree               |
| `lshw`                | Detailed hardware inventory           |
| `dmidecode`           | BIOS/SMBIOS hardware info (root)      |
| `sensors`             | CPU/board temperatures (`lm-sensors`) |
| `nvidia-smi`          | GPU status and utilisation            |
| `dmesg`               | Kernel ring buffer messages           |
| `journalctl`          | systemd journal log viewer            |
| `auditctl`/`ausearch` | Kernel audit subsystem                |
| `env` / `printenv`    | Show environment variables            |
| `sysctl -a`           | Show all kernel parameters            |
| `ulimit`              | Per-process resource limits           |
| `procinfo` / `/proc`  | Kernel-exposed process/system data    |
| `fastfetch`           | Aesthetic system summary               |

## 9. User & Group Management

| Command                 | Description                         |
| ----------------------- | ----------------------------------- |
| `useradd -m user`       | Create a user with a home directory |
| `adduser user`          | Interactive Debian/Ubuntu wrapper   |
| `usermod -aG sudo user` | Add user to supplementary group     |
| `usermod -L user`       | Lock an account                     |
| `userdel -r user`       | Delete user and home directory      |
| `passwd user`           | Set/change a password               |
| `passwd -l user`        | Lock a password                     |
| `chage -l user`         | Show password aging policy          |
| `groupadd grp`          | Create a group                      |
| `groupdel grp`          | Delete a group                      |
| `gpasswd -a user grp`   | Add a user to a group               |
| `getent passwd`         | Query NSS user database             |
| `getent group`          | Query NSS group database            |
| `vipw` / `vigr`         | Safely edit passwd / group files    |
| `pwck` / `grpck`        | Verify passwd / group integrity     |
| `chsh -s /bin/zsh`      | Change login shell                  |
| `su - user`             | Switch user session                 |
| `sudo -i`               | Root login shell                    |
| `loginctl`              | Manage sessions and seats           |
| `pam_tally2`/`faillock` | Failed-login counters               |

## 10. Package Management

### Debian / Ubuntu (APT)

| Command                    | Description                          |
| -------------------------- | ------------------------------------ |
| `apt update`               | Refresh package indexes              |
| `apt upgrade`              | Upgrade installed packages           |
| `apt full-upgrade`         | Upgrade, allowing dependency changes |
| `apt install pkg`          | Install a package                    |
| `apt remove pkg`           | Remove a package                     |
| `apt purge pkg`            | Remove including config files        |
| `apt autoremove`           | Remove unneeded dependencies         |
| `apt search pat`           | Search the index                     |
| `apt show pkg`             | Package metadata                     |
| `apt list --installed`     | List installed packages              |
| `dpkg -i pkg.deb`          | Install a local `.deb`               |
| `dpkg -l`                  | List installed packages              |
| `apt-file search file`     | Which package ships a file           |
| `add-apt-repository ppa:x` | Add a PPA                            |

### Fedora / RHEL (DNF / YUM)

| Command                        | Description                   |
| ------------------------------ | ----------------------------- |
| `dnf install pkg`              | Install a package             |
| `dnf remove pkg`               | Remove a package              |
| `dnf upgrade`                  | Upgrade all packages          |
| `dnf search pat`               | Search repositories           |
| `dnf info pkg`                 | Package metadata              |
| `dnf provides file`            | Which package provides a file |
| `rpm -ivh pkg.rpm`             | Install an RPM                |
| `rpm -qa`                      | List installed RPMs           |
| `rpm -qf file`                 | Which RPM owns a file         |
| `dnf groupinstall "Dev Tools"` | Install a package group       |

### Arch (Pacman)

| Command                  | Description                   |
| ------------------------ | ----------------------------- |
| `pacman -S pkg`          | Install                       |
| `pacman -Rns pkg`        | Remove with deps and config   |
| `pacman -Syu`            | Sync and upgrade the system   |
| `pacman -Ss pat`         | Search                        |
| `pacman -Q` / `-Qo file` | Query installed / owning file |
| `yay` / `paru`           | AUR helper                    |

### Other

| Command               | Description                        |
| --------------------- | ---------------------------------- |
| `snap install pkg`    | Install a snap                     |
| `flatpak install pkg` | Install a Flatpak                  |
| `pip install pkg`     | Install a Python package           |
| `pipx install app`    | Install an isolated Python CLI app |
| `npm install -g pkg`  | Install a global Node package      |
| `cargo install pkg`   | Install a Rust binary              |
| `go install pkg`      | Install a Go program               |
| `gem install pkg`     | Install a Ruby gem                 |

## 11. Networking

| Command                           | Description                          |
| --------------------------------- | ------------------------------------ |
| `ip addr`                         | Show IP addresses on interfaces      |
| `ip link`                         | Show link-layer interfaces and state |
| `ip route`                        | Show the routing table               |
| `ip neigh`                        | Show the ARP/neighbour table         |
| `ip -br a`                        | Compact interface summary            |
| `ifconfig`                        | Legacy interface configuration       |
| `route -n`                        | Legacy routing table                 |
| `ss -tulpn`                       | TCP/UDP listeners with PIDs          |
| `ss -s`                           | Socket summary statistics            |
| `netstat -tulpn`                  | Legacy socket listing                |
| `ping host`                       | ICMP reachability test               |
| `ping -c4 host`                   | Send exactly 4 pings                 |
| `traceroute host`                 | Path trace to a host                 |
| `tracepath host`                  | Path trace without root              |
| `mtr host`                        | Continuous traceroute + ping stats   |
| `dig domain`                      | DNS query (record details)           |
| `dig +short domain`               | Concise DNS answer                   |
| `dig -x ip`                       | Reverse DNS lookup                   |
| `nslookup domain`                 | One-shot DNS query (bare = interactive) |
| `host domain`                     | Simple DNS lookup                    |
| `resolvectl status`               | systemd-resolved DNS config          |
| `curl url`                        | Transfer data from a URL             |
| `curl -O url`                     | Download, keeping the remote name    |
| `curl -L -o f url`                | Follow redirects, save to `f`        |
| `curl -I url`                     | Fetch headers only                   |
| `curl -X POST -d @f`              | POST a body                          |
| `wget url`                        | Download files recursively/simply    |
| `wget -c url`                     | Resume an interrupted download       |
| `aria2c url`                      | Multi-connection downloader          |
| `nc -l 8080`                      | Netcat listener                      |
| `nc host 8080`                    | Netcat client (port check)           |
| `socat TCP-LISTEN:1,fork -`       | Multiplexed socket relay             |
| `telnet host port`                | Raw TCP connectivity test            |
| `ssh user@host`                   | Secure shell session                 |
| `scp file user@host:/dst`         | Copy over SSH                        |
| `sftp user@host`                  | Interactive SSH file transfer        |
| `rsync -av src dst`               | Efficient incremental sync           |
| `rsync -av --delete a/ b/`        | Mirror with deletions                |
| `iptables -L -n -v`               | List firewall rules (legacy)         |
| `nft list ruleset`                | List nftables ruleset                |
| `ufw status`                      | UFW firewall status                  |
| `firewall-cmd --list-all`         | firewalld configuration              |
| `tcpdump -i any -nn`              | Capture packets                      |
| `tcpdump -w cap.pcap`             | Write a capture file                 |
| `tshark`                          | Wireshark's CLI dissector            |
| `nmap host`                       | Port scan and host discovery         |
| `nmap -sV -p- host`               | Full TCP scan with versions          |
| `arp -a`                          | Show the ARP cache                   |
| `ethtool eth0`                    | NIC link settings and stats          |
| `iwconfig` / `nmcli`              | Wireless config / NetworkManager CLI |
| `nmcli device status`             | NetworkManager device state          |
| `hostname -I`                     | Show all local IP addresses          |
| `whois domain`                    | Domain registration lookup           |
| `openssl s_client -connect h:443` | TLS handshake inspection             |
| `speedtest-cli`                   | Bandwidth test                       |
| `dhclient -v eth0`                | Request a DHCP lease                 |

## 12. Disk & Filesystem

| Command                                  | Description                        |
| ---------------------------------------- | ---------------------------------- |
| `df -h`                                  | Filesystem usage, human-readable   |
| `df -i`                                  | Inode usage                        |
| `du -sh dir`                             | Total size of a directory          |
| `du -h --max-depth=1`                    | Per-subdirectory sizes             |
| `ncdu dir`                               | Interactive disk-usage browser     |
| `lsblk -f`                               | Block devices with filesystem/UUID |
| `blkid`                                  | Device UUIDs and filesystems       |
| `fdisk -l`                               | Partition tables (MBR/GPT)         |
| `parted -l`                              | Partition layout (alternative)     |
| `gdisk -l`                               | GPT-aware partition tool           |
| `mkfs.ext4 dev`                          | Create an ext4 filesystem          |
| `mkfs.xfs` / `mkfs.vfat`                 | Create XFS / FAT filesystems       |
| `mkswap dev`                             | Initialise a swap area             |
| `swapon` / `swapoff`                     | Enable / disable swap              |
| `mount dev dir`                          | Mount a filesystem                 |
| `mount -o remount,rw /`                  | Remount read-write                 |
| `umount dir`                             | Unmount (use `-l` for lazy)        |
| `findmnt`                                | Show mounted filesystems as a tree |
| `mount.cifs`/`mount.nfs`                 | Mount SMB/CIFS / NFS shares        |
| `fsck dev`                               | Check and repair a filesystem      |
| `e2fsck -f dev`                          | Force ext2/3/4 check               |
| `tune2fs -l dev`                         | Show ext filesystem parameters     |
| `resize2fs dev`                          | Grow/shrink an ext filesystem      |
| `xfs_growfs dir`                         | Grow an XFS filesystem             |
| `dd if=src of=dst bs=4M`                 | Byte-level copy / image            |
| `dd if=/dev/zero of=f bs=1M count=100`   | Write 100 MB of zeros              |
| `dd if=/dev/urandom of=f bs=1M count=10` | Write random data                  |
| `sync`                                   | Flush filesystem buffers to disk   |
| `hdparm -tT dev`                         | Disk read speed test               |
| `smartctl -a dev`                        | SMART health and attributes        |
| `badblocks -sv dev`                      | Scan for bad blocks                |
| `lvm` / `pvdisplay` / `vgdisplay`        | LVM physical/volume-group tools    |
| `mdadm --detail /dev/md0`                | Software RAID status               |
| `quota` / `repquota`                     | User/group disk quotas             |
| `ionice -c3 cmd`                         | Run with idle I/O priority         |
| `inotifywait -m dir`                     | Watch a directory for changes      |
| `lsof +D dir`                            | Processes holding files in a dir   |
| `fallocate -l 1G file`                   | Preallocate a file's blocks        |
| `chroot dir`                             | Run a command with a changed root  |

## 13. systemd, Services & Boot

| Command                                  | Description                         |
| ---------------------------------------- | ----------------------------------- |
| `systemctl status unit`                  | Show a unit's state and recent logs |
| `systemctl start/stop unit`              | Start / stop a service              |
| `systemctl restart unit`                 | Restart a service                   |
| `systemctl reload unit`                  | Reload config without restart       |
| `systemctl enable unit`                  | Enable at boot                      |
| `systemctl disable unit`                 | Disable at boot                     |
| `systemctl --failed`                     | List failed units                   |
| `systemctl list-units --type=service`    | List service units                  |
| `systemctl list-timers`                  | List scheduled timer units          |
| `systemctl list-dependencies unit`       | Show a unit's dependency tree       |
| `systemctl daemon-reload`                | Re-read unit files after edits      |
| `systemctl reset-failed unit`            | Clear a latched failed state        |
| `systemctl cat unit`                     | Show a unit's effective definition  |
| `systemctl edit unit`                    | Create a drop-in override           |
| `systemctl isolate multi-user.target`    | Switch run target                   |
| `systemd-analyze blame`                  | Boot time per unit                  |
| `systemd-analyze critical-chain`         | Boot critical path                  |
| `systemctl get-default`                  | Current default target              |
| `systemctl set-default graphical.target` | Change default target               |
| `journalctl -u unit`                     | Logs for one unit                   |
| `journalctl -b`                          | Logs from the current boot          |
| `journalctl -b -1`                       | Logs from the previous boot         |
| `journalctl -f`                          | Follow the journal live             |
| `journalctl -p err`                      | Only error-level entries            |
| `journalctl --since "1 hour ago"`        | Time-bounded logs                   |
| `journalctl --disk-usage`                | Journal disk consumption            |
| `journalctl --vacuum-time=7d`            | Trim the journal                    |
| `loginctl list-sessions`                 | Active login sessions               |
| `hostnamectl set-hostname x`             | Set the hostname                    |
| `shutdown -h now`                        | Halt immediately                    |
| `shutdown -r +5`                         | Reboot in 5 minutes                 |
| `reboot` / `poweroff`                    | Restart / power off                 |
| `init 3` / `init 5`                      | Switch to text / graphical mode     |

## 14. Scheduling & Automation

| Command                           | Description                                |
| --------------------------------- | ------------------------------------------ |
| `crontab -l`                      | List your cron jobs                        |
| `crontab -e`                      | Edit your crontab                          |
| `crontab -r`                      | Remove your crontab                        |
| `cron` spec `m h dom mon dow cmd` | Standard cron schedule fields              |
| `/etc/cron.d/`                    | System cron drop-in files                  |
| `systemd timer` + `.service`      | Modern replacement for cron                |
| `at 15:00`                        | Schedule a one-off command                 |
| `atq` / `atrm n`                  | List / remove `at` jobs                    |
| `batch`                           | Run when system load permits               |
| `anacron`                         | Catch up missed periodic jobs              |
| `run-parts dir`                   | Run all scripts in a directory             |
| `flock -n f cmd`                  | Mutually exclusive execution via lock file |
| `timeout 10 cmd`                  | Kill a command after 10 s                  |
| `watch -n60 cmd`                  | Periodic re-execution in a terminal        |
| `systemd-run --on-calendar`       | Transient scheduled unit                   |
| `sleep 5`                         | Delay execution                            |
| `until`/`wait`                    | Shell-level sequencing primitives          |

## 15. Shell, Environment & Scripting

| Command                        | Description                            |
| ------------------------------ | -------------------------------------- |
| `bash` / `sh` / `zsh` / `fish` | Shell interpreters                     |
| `echo text`                    | Print text (use `-e` for escapes)      |
| `printf fmt args`              | Formatted output                       |
| `read var`                     | Read a line from stdin into a variable |
| `export VAR=val`               | Set an environment variable            |
| `unset VAR`                    | Remove a variable                      |
| `set -euo pipefail`            | Strict-mode script preamble            |
| `source file` / `. file`       | Execute in the current shell           |
| `alias ll='ls -la'`            | Define a command alias                 |
| `unalias ll`                   | Remove an alias                        |
| `history`                      | Show command history                   |
| `history -c`                   | Clear in-memory history                |
| `Ctrl-R`                       | Reverse-search history interactively   |
| `!!`                           | Repeat the last command                |
| `!$`                           | Last argument of the previous command  |
| `sudo !!`                      | Re-run the previous command as root    |
| `test` / `[ ]`                 | Condition evaluation                   |
| `[[ ]]`                        | Bash extended conditions               |
| `$?`                           | Exit status of the last command        |
| `$$` / `$!`                    | Current PID / last background PID      |
| `$#` / `$@` / `$1`             | Arg count / all args / first arg       |
| `$(...)`                       | Command substitution                   |
| `$(( ))`                       | Arithmetic expansion                   |
| `for` / `while` / `until`      | Loop constructs                        |
| `case`                         | Multi-branch matching                  |
| `trap`                         | Handle signals inside a script         |
| `readonly VAR`                 | Mark a variable immutable              |
| `local`                        | Function-scoped variable               |
| `shift`                        | Drop the first positional argument     |
| `getopts`                      | Parse short option flags               |
| `declare -i/-a/-A`             | Typed / array / associative variables  |
| `eval`                         | Re-parse and execute a string          |
| `exec cmd`                     | Replace the shell process              |
| `command cmd`                  | Bypass functions/aliases               |
| `builtin cmd`                  | Invoke a shell builtin directly        |
| `bash -n script`               | Syntax-check without running           |
| `bash -x script`               | Trace execution                        |
| `shellcheck script`            | Static analysis for shell bugs         |
| `shfmt -w script`              | Format shell scripts                   |
| `dialog` / `whiptail`          | Text-mode UI dialogs                   |
| `jq '.key' file.json`          | JSON processor                         |
| `yq`                           | YAML processor                         |
| `bc`                           | Arbitrary-precision calculator         |
| `expr`                         | Evaluate expressions (legacy)          |
| `seq 1 10`                     | Generate number sequences              |
| `yes`                          | Repeat a string (often to auto-answer) |

## 16. Security, Encryption & Auditing

| Command                             | Description                          |
| ----------------------------------- | ------------------------------------ |
| `gpg --gen-key`                     | Generate a GPG key pair              |
| `gpg -e -r user file`               | Encrypt to a recipient               |
| `gpg -d file.gpg`                   | Decrypt                              |
| `gpg --sign file`                   | Create a detached/attached signature |
| `gpg --verify sig file`             | Verify a signature                   |
| `openssl genrsa 4096`               | Generate an RSA private key          |
| `openssl req -new -x509`            | Self-signed certificate              |
| `openssl x509 -text -noout`         | Inspect a certificate                |
| `openssl dgst -sha256 file`         | Hash a file                          |
| `ssh-keygen -t ed25519`             | Generate an SSH key pair             |
| `ssh-copy-id user@host`             | Install your public key remotely     |
| `ssh-agent` / `ssh-add key`         | Agent-based key management           |
| `ssh -L 8080:host:80 user@jump`     | Local port forward                   |
| `ssh -R 9000:localhost:3000 user@h` | Remote port forward                  |
| `ssh -D 1080 user@host`             | Dynamic SOCKS proxy                  |
| `sshfs user@host:/dir mnt`          | Mount a remote directory over SSH    |
| `gpg-zip` / `tar c f \| gpg`       | Encrypted tar archives                |
| `cryptsetup luksFormat dev`         | Create a LUKS encrypted volume       |
| `cryptsetup luksOpen dev name`      | Unlock a LUKS volume                 |
| `cryptsetup close name`             | Lock a LUKS volume                   |
| `eCryptfs`                          | Per-directory encryption             |
| `pass`                              | Password-store CLI over GPG/Git      |
| `keepassxc-cli`                     | KeePass database CLI                 |
| `vault`                             | HashiCorp Vault secrets client       |
| `age`                               | Modern simple file encryption        |
| `sqlmap`                            | Automated SQL-injection tester       |
| `nikto`                             | Web server misconfiguration scanner  |
| `lynis`                             | System hardening audit               |
| `chkrootkit` / `rkhunter`           | Rootkit detection                    |
| `clamscan -r dir`                   | ClamAV malware scan                  |
| `aide --check`                      | File-integrity verification          |
| `seinfo` / `getenforce`             | SELinux status                       |
| `semanage` / `restorecon`           | SELinux policy labels                |
| `apparmor_status`                   | AppArmor profile status              |
| `auditctl -l`                       | List kernel audit rules              |
| `tripwire`                          | Host-based IDS / integrity           |
| `openssl rand -hex 32`              | Generate random bytes                |
| `shred -n3 -z file`                 | Multi-pass overwrite                 |
| `tomb`                              | Encrypted-volume manager             |

## 17. Logs & Diagnostics

| Command                       | Description                         |
| ----------------------------- | ----------------------------------- |
| `journalctl`                  | systemd journal                     |
| `dmesg -T`                    | Kernel log with timestamps          |
| `dmesg -w`                    | Follow kernel messages              |
| `tail -f /var/log/syslog`     | Follow the Debian/Ubuntu system log |
| `tail -f /var/log/messages`   | Follow the RHEL-style system log    |
| `cat /var/log/auth.log`       | Authentication events (Debian)      |
| `cat /var/log/secure`         | Authentication events (RHEL)        |
| `zgrep pat file.gz`           | grep inside rotated logs            |
| `logrotate -d conf`           | Dry-run log rotation                |
| `last -x`                     | Shutdown/runlevel history           |
| `journalctl -k -b -1`         | Previous boot's kernel log          |
| `abrt-cli` / `apport-cli`     | Crash reporting                     |
| `coredumpctl`                 | Inspect systemd-coredump entries    |
| `systemd-analyze verify unit` | Validate a unit file                |
| `perf stat cmd`               | Hardware performance counters       |
| `bpftrace` / `bpftool`        | eBPF tracing                        |
| `ltrace cmd`                  | Library call tracing                |
| `gdb --args prog`             | Debugger                            |
| `valgrind ./prog`             | Memory error and leak detector      |
| `sysdig`                      | Container/system tracing            |
| `powertop`                    | Power-consumption analysis          |
| `iotop`                       | Per-process I/O monitor             |

## 18. Terminals, Sessions & Remote

| Command               | Description                          |
| --------------------- | ------------------------------------ |
| `tty`                 | Print the current terminal device    |
| `stty -a`             | Show terminal line settings          |
| `reset`               | Restore a garbled terminal           |
| `clear`               | Clear the screen                     |
| `tput cols`           | Query terminal capabilities/width    |
| `script file`         | Record a terminal session            |
| `screen`              | Detachable terminal multiplexer      |
| `tmux`                | Modern terminal multiplexer          |
| `tmux new -s name`    | New named session                    |
| `tmux attach -t name` | Reattach to a session                |
| `tmux ls`             | List sessions                        |
| `Ctrl-b d`            | Detach from tmux                     |
| `byobu`               | Wrapper over tmux/screen             |
| `wall msg`            | Broadcast a message to all terminals |
| `write user`          | Send a message to one user           |
| `mesg`                | Control write permission to your tty |
| `talk`                | Interactive chat with another user   |
| `who` / `w`           | Who is logged in                     |
| `ssh -t host cmd`     | Run a remote command with a tty      |
| `mosh user@host`      | Roaming-tolerant remote shell        |
| `autossh`             | Self-healing SSH tunnel              |
| `dtach`               | Minimal detach/reattach              |
| `abduco`              | Session management with dvtm         |
| `dvtm`                | Dynamic virtual terminal manager     |
| `expect`              | Script interactive programs          |
| `ansi2html`           | Convert ANSI output to HTML          |

## 19. File Transfer & Sync

| Command                     | Description                          |
| --------------------------- | ------------------------------------ |
| `scp file user@host:/dst`   | Copy over SSH                        |
| `scp -r dir user@host:/dst` | Recursive copy                       |
| `sftp user@host`            | Interactive transfer session         |
| `rsync -avz src/ dst/`      | Compressed incremental sync          |
| `rsync -e ssh ...`          | rsync over SSH                       |
| `rsync --exclude pat`       | Skip matching files                  |
| `rsync -a --delete`         | Mirror, removing extras              |
| `rcp` / `rlogin`            | Legacy insecure remote tools         |
| `ftp` / `lftp`              | FTP clients                          |
| `curl -T file url`          | Upload via HTTP PUT                  |
| `curl -F file=@f url`       | Multipart upload                     |
| `wget --mirror url`         | Recursive site download              |
| `rclone copy src dst`       | Cloud-storage sync (S3, Drive, etc.) |
| `rclone config`             | Configure rclone remotes             |
| `syncthing`                 | Peer-to-peer folder sync             |
| `unison`                    | Bidirectional file sync              |
| `aws s3 cp/sync`            | AWS S3 transfer (CLI)                |
| `gcloud storage cp/sync`    | Google Cloud Storage transfer        |
| `az storage copy`           | Azure storage transfer               |
| `nc -w3 host port < file`   | Netcat push                          |
| `croc send file`            | Simple encrypted peer transfer       |
| `magic-wormhole send f`     | Short-code file transfer             |
| `nfs`/`smbclient`           | Share access clients                 |

## 20. Multimedia & Documents

| Command                               | Description                       |
| ------------------------------------- | --------------------------------- |
| `ffmpeg -i in out`                    | Convert/transcode audio and video |
| `ffprobe file`                        | Inspect media streams             |
| `avconv`                              | ffmpeg fork (legacy)              |
| `sox in.wav out.mp3`                  | Audio conversion and effects      |
| `lame` / `flac` / `oggenc`            | MP3 / FLAC / Ogg encoders         |
| `mediainfo file`                      | Detailed media metadata           |
| `exiftool image`                      | Read/write EXIF metadata          |
| `identify image`                      | ImageMagick image info            |
| `magick in.png out.jpg`               | ImageMagick conversion            |
| `mogrify -resize 50% *.jpg`           | In-place batch image edit         |
| `montage`                             | Compose image tiling              |
| `optipng` / `jpegoptim`               | Lossless image optimisation       |
| `pdftk`                               | PDF merge/split/rotate            |
| `pdfinfo` / `pdftotext`               | PDF metadata / text extraction    |
| `qpdf --decrypt`                      | PDF structural tooling            |
| `gs -sDEVICE=...`                     | Ghostscript PDF/PS processing     |
| `enscript` / `a2ps`                   | Pretty-print text to PostScript   |
| `libreoffice --headless --convert-to` | Office document conversion        |
| `pandoc in.md -o out.pdf`             | Universal document converter      |
| `tesseract img out`                   | OCR text extraction               |
| `scrot` / `grim` / `import`           | Screenshot capture (X11/Wayland)  |
| `xclip` / `wl-copy`                   | Clipboard from the terminal       |
| `xdotool`                             | Automate X11 window/input         |
| `mplayer` / `mpv` / `vlc`             | Media players                     |
| `cvlc --intf dummy`                   | Headless VLC                      |
| `yt-dlp url`                          | Download from video sites         |
| `asunder` / `abcde`                   | CD ripping                        |
| `cdrecord`/`wodim`                    | Optical disc burning              |
| `dd if=img.iso of=dev`                | Write a raw ISO to a device       |

## 21. Printing, Hardware & Peripherals

| Command                    | Description                   |
| -------------------------- | ----------------------------- |
| `lp file`                  | Print a file                  |
| `lpstat -p`                | Printer status                |
| `lpq` / `lprm`             | Show / remove print jobs      |
| `cancel -a`                | Cancel all jobs for a printer |
| `cupsd` / `cupsctl`        | CUPS scheduler control        |
| `lsusb -t`                 | USB device tree               |
| `usbguard`                 | USB device authorisation      |
| `udevadm info --query=all` | Device metadata               |
| `udevadm monitor`          | Live udev events              |
| `bluetoothctl`             | Bluetooth device management   |
| `rfkill list`              | Wireless kill-switch state    |
| `xinput`                   | X11 input device control      |
| `evtest`                   | Raw input event viewer        |
| `setleds`                  | Keyboard LED control          |
| `alsamixer` / `amixer`     | ALSA audio controls           |
| `pactl` / `wpctl`          | PulseAudio / PipeWire control |
| `pavucontrol`              | GUI audio mixer               |
| `aplay` / `paplay`         | Play an audio file            |
| `arecord -f cd out.wav`    | Record audio                  |
| `speaker-test`             | Channel/speaker test          |
| `xrandr`                   | Display resolution and layout |
| `wlr-randr`                | Wayland display config        |
| `brightnessctl`            | Backlight control             |
| `xbacklight`               | X11 backlight control         |
| `acpi -V`                  | Battery and thermal status    |
| `upower -d`                | Power device details          |
| `tlp-stat`                 | Laptop power tuning status    |
| `powertop --auto-tune`     | Apply power-saving tunables   |

## 22. Kernel & Boot

| Command                         | Description                       |
| ------------------------------- | --------------------------------- |
| `uname -r`                      | Running kernel release            |
| `lsmod`                         | List loaded kernel modules        |
| `modprobe name`                 | Load a module (with deps)         |
| `modprobe -r name`              | Unload a module                   |
| `insmod file.ko`                | Insert a module directly          |
| `rmmod name`                    | Remove a module                   |
| `modinfo name`                  | Module metadata and parameters    |
| `sysctl -w key=val`             | Set a kernel parameter at runtime |
| `dmesg \| grep -i`               | Filter kernel log messages            |
| `update-initramfs -u`           | Rebuild the initramfs (Debian)    |
| `mkinitcpio -P`                 | Rebuild initramfs (Arch)          |
| `dracut -f`                     | Rebuild initramfs (RHEL/Fedora)   |
| `update-grub` / `grub-mkconfig` | Regenerate the GRUB config        |
| `grub-install dev`              | Install the bootloader            |
| `efibootmgr -v`                 | UEFI boot entries                 |
| `bootctl status`                | systemd-boot status               |
| `kexec -l vmlinuz`              | Load a kernel for fast reboot     |
| `kmod`                          | Manage module loading rules       |
| `perf top`                      | Live kernel profile               |
| `trace-cmd`                     | Ftrace front-end                  |
| `crash`                         | Kernel crash-dump analysis        |
| `kpatch`/`livepatch`            | Live kernel patching              |

## 23. Containers & Virtualisation

| Command                      | Description                       |
| ---------------------------- | --------------------------------- |
| `docker ps -a`               | List containers                   |
| `docker run -it img sh`      | Run an interactive container      |
| `docker exec -it c sh`       | Shell into a running container    |
| `docker logs -f c`           | Follow container logs             |
| `docker images`              | List local images                 |
| `docker build -t t .`        | Build an image from a Dockerfile  |
| `docker pull` / `push`       | Fetch / publish an image          |
| `docker compose up -d`       | Start a Compose stack             |
| `docker stats`               | Live container resource usage     |
| `docker system prune -a`     | Remove unused images (add `--volumes` for volumes) |
| `podman`                     | Daemonless container engine       |
| `buildah`                    | OCI image builder                 |
| `skopeo copy`                | Move images between registries    |
| `nerdctl`                    | containerd CLI, Docker-compatible |
| `ctr`                        | containerd low-level CLI          |
| `kubectl get pods`           | Kubernetes resource queries       |
| `kubectl apply -f m.yaml`    | Apply a manifest                  |
| `kubectl logs -f pod`        | Pod logs                          |
| `kubectl exec -it pod -- sh` | Exec into a pod                   |
| `helm install n chart`       | Install a Helm release            |
| `minikube start`             | Local Kubernetes cluster          |
| `kind create cluster`        | Kubernetes-in-Docker              |
| `k9s`                        | Terminal Kubernetes UI            |
| `virsh list --all`           | libvirt VM management             |
| `virt-install`               | Create a VM                       |
| `qemu-system-x86_64`         | Full-system emulation             |
| `lxc` / `lxd`                | System containers                 |
| `vagrant up`                 | Reproducible VM environments      |
| `distrobox create`           | Container-backed dev environments |
| `toolbox`                    | Fedora container toolboxes        |
| `distrobox-export`           | Expose container apps to the host |

## 24. Development & Build Tools

| Command                         | Description                      |
| ------------------------------- | -------------------------------- |
| `gcc` / `clang`                 | C/C++ compilers                  |
| `g++`                           | C++ front-end                    |
| `make` / `make -j8`             | Build automation / parallel jobs |
| `cmake`                         | Cross-platform build generator   |
| `meson` / `ninja`               | Modern build system / backend    |
| `pkg-config`                    | Library compile/link flags       |
| `ldd binary`                    | Shared-library dependencies      |
| `nm` / `objdump`                | Inspect object/symbol tables     |
| `readelf -h file`               | ELF header inspection            |
| `strip file`                    | Remove debug/symbol info         |
| `ar rcs lib.a objs`             | Build a static library           |
| `pkg-config --libs`             | Emit linker flags                |
| `astyle` / `clang-format`       | Code formatters (C/C++)          |
| `gdb` / `lldb`                  | Debuggers                        |
| `lldb-server`                   | Remote LLDB                      |
| `valgrind` / `heaptrack`        | Memory profiling                 |
| `perf` / `gprof`                | Profiling                        |
| `gcov` / `lcov`                 | Code coverage                    |
| `gcov` / `lcov`                 | Code coverage                    |
| `ctags` / `cscope`              | Code indexing for editors        |
| `git`                           | Distributed version control      |
| `svn` / `hg`                    | Centralised / Mercurial VCS      |
| `gh`                            | GitHub CLI                       |
| `glab`                          | GitLab CLI                       |
| `python3` / `pip` / `venv`      | Python runtime and packaging     |
| `node` / `npm` / `pnpm` / `bun` | JavaScript runtimes              |
| `go build` / `go test`          | Go toolchain                     |
| `cargo build/test`              | Rust toolchain                   |
| `mvn` / `gradle`                | JVM build tools                  |
| `dotnet`                        | .NET CLI                         |
| `ruby` / `gem` / `bundler`      | Ruby toolchain                   |
| `php` / `composer`              | PHP toolchain                    |
| `lua` / `luarocks`              | Lua runtime                      |
| `julia`                         | Numerical computing              |
| `R` / `Rscript`                 | Statistical computing            |
| `sqlite3`                       | SQLite CLI                       |
| `psql` / `mysql` / `mongosh`    | Database clients                 |
| `redis-cli`                     | Redis client                     |
| `jq` / `yq` / `dasel`           | Structured-data processors       |

## 25. Miscellaneous Utilities

| Command                   | Description                        |
| ------------------------- | ---------------------------------- |
| `man cmd`                 | Manual page                        |
| `man -k keyword`          | Search man page descriptions       |
| `info cmd`                | GNU info documentation             |
| `cmd --help`              | Quick usage summary                |
| `tldr cmd`                | Community example-based help       |
| `apropos word`            | Search the manual index            |
| `whatis cmd`              | One-line description               |
| `help builtin`            | Bash builtin help                  |
| `echo $PATH`              | Show the executable search path    |
| `hash -r`                 | Clear the shell's command cache    |
| `compgen -c`              | List all available commands        |
| `compgen -b`              | List shell builtins                |
| `readlink -f`             | Canonical path resolution          |
| `bc -l`                   | Interactive calculation            |
| `dc`                      | Reverse-polish calculator          |
| `units`                   | Unit conversion                    |
| `qalc`                    | Advanced CLI calculator            |
| `date -d @1699999999`     | Convert an epoch timestamp         |
| `date +%s`                | Current epoch seconds              |
| `ncal` / `ncal -w`        | Vertical calendar / with week numbers |
| `watch`                   | Periodic execution (see Processes) |
| `parallel`                | GNU parallel job runner            |
| `entr`                    | Re-run a command on file change    |
| `inotifywait`             | Wait for filesystem events         |
| `fswatch`                 | Cross-platform file watching       |
| `uuidgen`                 | Generate a UUID                    |
| `base64` / `base32`       | Encode/decode to base-N            |
| `xxd -r`                  | Reverse a hex dump                 |
| `openssl rand -base64 24` | Random base64 string               |
| `pwgen` / `apg`           | Generate passwords                 |
| `passwd -S`               | Password status for a user         |
| `htpasswd`                | Manage Apache basic-auth files     |
| `mkpasswd`                | Generate crypt-style hashes        |
| `crontab -n`              | Dry-run validation (Debian cron)   |
| `screenkey`               | On-screen keystroke display        |
| `figlet` / `toilet`       | ASCII-art banners                  |
| `lolcat`                  | Rainbow-coloured output            |
| `cowsay` / `fortune`      | Novelty output                     |
| `pv file`                 | Pipe progress meter                |
| `progress`                | Show progress of running coreutils |
| `watch -d`                | Highlight changing output          |
| `notify-send msg`         | Desktop notification               |
| `zenity` / `yad`          | GUI dialogs from scripts           |
| `xdg-open file`           | Open with the default app          |
| `xdg-mime`                | Query/set file associations        |
| `xdg-user-dir`            | Resolve XDG user directories       |
| `dbus-send`               | Send a D-Bus message               |
| `busctl`                  | Introspect systemd D-Bus services  |
| `gdbus`                   | D-Bus CLI                          |
| `strace -c cmd`           | Summarise syscall counts           |
| `blktrace`                | Block-layer I/O tracing            |

---

## Handy One-Liners

```bash
# Find the 10 largest files under a directory
find . -type f -printf '%s %p\n' | sort -nr | head -10

# Recursively replace a string across text files in place
find . -path ./.git -prune -o -type f -exec grep -lZ 'old' {} + \
  | xargs -0 sed -i 's/old/new/g'

# Tail every matching log file at once
tail -F /var/log/nginx/*.log

# Total size of files by extension
find . -type f -printf '%f\t%s\n' | awk -F'\t' '{e=$1; sub(/.*\./, "", e); s[e]+=$2} END {for (e in s) print s[e], e}' \
  | sort -nr | head

# Watch disk usage every 5 seconds
watch -n5 'df -h | grep -v tmpfs'

# Kill whatever is listening on a port
fuser -k 8080/tcp

# Copy a directory tree preserving everything
rsync -aHAX --info=progress2 src/ dst/

# Compress a directory with the best ratio
tar -cJf backup.tar.xz dir/

# Check certificates expiring within 30 days
echo | openssl s_client -connect host:443 -servername host 2>/dev/null \
  | openssl x509 -noout -checkend 2592000 -dates

# Show top memory consumers
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head

# Sequential rename with zero-padding
i=1; for f in *.jpg; do mv "$f" "$(printf '%04d.jpg' $i)"; i=$((i+1)); done
```

---

## Signal Reference

| Signal      | Number | Meaning                        |
| ----------- | ------ | ------------------------------ |
| `SIGHUP`    | 1      | Hangup / reload config         |
| `SIGINT`    | 2      | Interrupt (`Ctrl-C`)           |
| `SIGQUIT`   | 3      | Quit with core dump (`Ctrl-\`) |
| `SIGKILL`   | 9      | Forced kill (uncatchable)      |
| `SIGSEGV`   | 11     | Segmentation fault             |
| `SIGTERM`   | 15     | Polite terminate (default)     |
| `SIGSTOP`   | 19     | Pause (uncatchable)            |
| `SIGTSTP`   | 20     | Terminal stop (`Ctrl-Z`)       |
| `SIGCONT`   | 18     | Resume after stop              |
| `SIGUSR1/2` | 10/12  | Application-defined            |

## Exit Status Reference

| Code    | Meaning                                      |
| ------- | -------------------------------------------- |
| `0`     | Success                                      |
| `1`     | General error / false condition              |
| `2`     | Shell misuse / bad option                    |
| `126`   | Command found but not executable             |
| `127`   | Command not found                            |
| `128+n` | Killed by signal `n` (e.g. `130` = `Ctrl-C`) |

## Permission Bit Quick Table

| Octal | Symbolic | Meaning                |
| ----- | -------- | ---------------------- |
| `0`   | `---`    | No permissions         |
| `1`   | `--x`    | Execute only           |
| `2`   | `-w-`    | Write only             |
| `3`   | `-wx`    | Write + execute        |
| `4`   | `r--`    | Read only              |
| `5`   | `r-x`    | Read + execute         |
| `6`   | `rw-`    | Read + write           |
| `7`   | `rwx`    | Read + write + execute |

Special bits: `4xxx` SUID, `2xxx` SGID, `1xxx` sticky (e.g. `4755`, `1777`).

## Useful Paths

| Path                            | Contents                              |
| ------------------------------- | ------------------------------------- |
| `/etc/`                         | System-wide configuration             |
| `/var/log/`                     | Log files                             |
| `/proc/`                        | Kernel/process pseudo-files           |
| `/sys/`                         | Kernel/device pseudo-files            |
| `/dev/`                         | Device nodes                          |
| `/tmp/`                         | World-writable temp (cleared on boot) |
| `/usr/bin/`, `/usr/local/bin/`  | Executables                           |
| `/etc/fstab`                    | Filesystem mount table                |
| `/etc/crontab`, `/etc/cron.*/`  | System cron                           |
| `/etc/systemd/system/`          | Local systemd units                   |
| `/etc/hosts`                    | Static hostname mapping               |
| `/etc/resolv.conf`              | DNS resolver config                   |
| `/etc/passwd`, `/etc/group`     | User/group databases                  |
| `/etc/sudoers.d/`               | Sudo drop-in rules                    |
| `~/.config/`, `~/.local/share/` | XDG user config/data                  |
| `~/.bashrc`, `~/.profile`       | Shell startup files                   |
| `~/.ssh/`                       | SSH keys and known_hosts              |
