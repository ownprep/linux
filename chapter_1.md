# Chapter 1 — Linux Introduction & Fundamentals

---

## Table of Contents

- [1. Computer Basics](#1-computer-basics)
- [2. Key Definitions](#2-key-definitions)
  - [Program](#21-program)
  - [Operating System (OS)](#22-operating-system-os)
  - [Kernel](#23-kernel)
  - [Shell](#24-shell)
  - [Process](#25-process)
- [3. Linux](#3-linux)
  - [Linux Distributions](#31-linux-distributions-distros)
- [4. Basic System Calls](#4-basic-system-calls)
- [5. Linux Directory Structure (FHS)](#5-linux-directory-structure-fhs)
- [6. Virtual File System (VFS)](#6-virtual-file-system-vfs)
- [7. Essential Commands](#7-essential-commands)
  - [Listing & Navigation](#71-listing--navigation)
  - [File & Directory Management](#72-file--directory-management)
  - [Getting Help](#73-getting-help)
  - [File Creation & Editors](#74-file-creation--text-editors)
  - [Viewing Files](#75-viewing-file-contents)
  - [Text Processing](#76-text-processing--filtering)
- [8. Access Management](#8-access-management)
  - [File Permissions](#81-file-permissions)
  - [chmod — Change Permissions](#82-chmod--change-permissions)
  - [chown — Change Ownership](#83-chown--change-ownership)
  - [chattr — Change File Attributes](#84-chattr--change-file-attributes)
  - [setfacl — Access Control Lists](#85-setfacl--access-control-lists)
- [9. Hard Links vs Symbolic Links](#9-hard-links-vs-symbolic-links)

---

## 1. Computer Basics

A computer is made up of two fundamental components:

| Component | Examples |
|-----------|----------|
| **Hardware** | CPU, Memory (RAM), Hard Disk, Peripheral Devices, Networking Devices |
| **Software** | Programs, Databases, Operating System, Application Software |

---

## 2. Key Definitions

### 2.1 Program

A **program** is a *passive file* stored on disk — a set of instructions that tells a computer what to do. It does nothing until it is executed.

### 2.2 Operating System (OS)

An **OS** is a collection of system software that manages a computer's hardware resources and provides services to applications running on top of it.

### 2.3 Kernel

The **kernel** is the core foundational component of the OS. It boots the OS and acts as the primary bridge between hardware and software applications.

The kernel handles:

- **Process management** — creating, scheduling, and terminating processes
- **Memory management** — allocating and managing RAM
- **Device drivers** — communicating with hardware devices
- **System calls** — providing an interface for programs to request OS services

The "API" of a kernel is technically the System Call Interface. It is a predefined list of functions that the kernel agrees to perform for applications.

- **File Management:** open(), read(), write(), close()
- **Process Management:** fork() (create a process), exec() (run a program), exit()
- **Networking:** socket(), connect(), send()

### 2.4 Shell

The **shell** is a user-facing interface used to interact with the kernel and OS. It accepts commands and passes them to the kernel for execution.

Examples: `bash`, `zsh`, `sh`

### 2.5 Process

A **process** is a program that is actively executing — it has been given CPU time, memory, and system resources.

> [!NOTE]
> **Program vs Process**
> | | |
> |---|---|
> | **Program** | Passive file sitting on disk |
> | **Process** | Program actively executing with CPU & memory assigned |
>
> Simply loading a program into RAM alone does **not** make it a process — it must be *actively executing*.

---

## 3. Linux

**Linux** is an open-source, Unix-like OS kernel. While technically just the kernel, the term "Linux" commonly refers to a full **Linux distribution** — the kernel bundled with system software, tools, and package managers.

Linux is widely used in:
- Servers and data centres
- Supercomputers *(over 90% of the world's top 500 run Linux)*
- Embedded systems (routers, IoT devices)
- Mobile devices (Android is built on the Linux kernel)

### 3.1 Linux Distributions (Distros)

A Linux **distro** is a complete OS built around the Linux kernel. Distros differ in the programs, tools, package managers, and configuration they bundle.

| Distribution | Type | Package Manager |
|---|---|---|
| **Debian** | Community / Free | `apt` |
| **Ubuntu** | Community / Free (based on Debian) | `apt` |
| **CentOS Stream** | Community / Free (upstream preview of RHEL) | `dnf` / `yum` |
| **RHEL** | Enterprise / Paid (Red Hat) | `dnf` (modern), `yum` (legacy) |
| **Android** | Mobile (based on Linux kernel) | — |

> [!TIP]
> - **RHEL family** (RHEL, CentOS Stream) → `dnf` / `yum` — enterprise-focused
> - **Debian family** (Debian, Ubuntu) → `apt` — community-focused
> - `dnf` is the modern replacement for `yum`

---

## 4. Basic System Calls

System calls are the interface between **user-space programs** and the **kernel**. When a program needs a hardware resource, it makes a system call.

### File Operations

| Call | Description | Example |
|------|-------------|---------|
| `open()` | Opens a file and returns a file descriptor | `fd = open("file.txt", O_RDONLY)` |
| `read()` | Reads data from an open file descriptor | `read(fd, buffer, 1024)` |
| `write()` | Writes data to an open file descriptor | `write(fd, "hello", 5)` |
| `close()` | Closes an open file descriptor | `close(fd)` |

### Process Operations

| Call | Description | Example |
|------|-------------|---------|
| `fork()` | Creates a child process (copy of parent) | `pid = fork()` |
| `exec()` | Replaces current process with a new program | `execl("/bin/ls", "ls", NULL)` |
| `exit()` | Terminates the current process | `exit(0)` |
| `wait()` | Parent waits for child process to finish | `wait(&status)` |

### Memory Operations

| Call | Description | Example |
|------|-------------|---------|
| `malloc()` | Allocates a block of heap memory | `ptr = malloc(1024)` |
| `mmap()` | Maps files or devices into memory | `mmap(NULL, size, PROT_READ, ...)` |
| `brk()` | Adjusts the program's data segment size (low-level) | `brk(new_end)` |

---

## 5. Linux Directory Structure (FHS)

Linux uses a single hierarchical tree rooted at `/` (root). Everything — files, devices, processes — lives somewhere in this tree. This layout is standardised by the **Filesystem Hierarchy Standard (FHS)**.

```
/
├── root/       ← home of the superuser
├── home/       ← home dirs for regular users
├── boot/       ← kernel and bootloader files
├── etc/        ← system configuration files
├── usr/        ← user utilities and programs
│   ├── bin/    ← most regular programs (python3, vim, …)
│   ├── sbin/   ← extra admin tools (sshd, …)
│   └── lib/    ← helper files for /usr/bin programs
├── opt/        ← optional / third-party software
├── bin/        ← essential binaries (ls, cp, …)  [symlink → /usr/bin on modern systems]
├── sbin/       ← system/admin binaries (reboot, fdisk, …)
├── dev/        ← device files (/dev/sda, …)
├── proc/       ← virtual FS exposing kernel/process info
├── sys/        ← live info about connected devices
├── var/        ← logs, caches, spool files
├── tmp/        ← temporary files (cleared on reboot)
├── mnt/        ← manual mount point
├── media/      ← auto-mount for USB, CD, …
└── lib/        ← shared libraries for /bin & /sbin
```

| Directory | Purpose |
|-----------|---------|
| `/` | Root of the entire filesystem |
| `/root` | Home directory for the root (superuser) account |
| `/home` | Home directories for regular users (e.g. `/home/alice`) |
| `/boot` | Bootloader files and the Linux kernel image |
| `/etc` | System-wide configuration files (e.g. `/etc/passwd`) |
| `/usr` | Where most software and shared files are stored |
| `/usr/bin` | Most regular programs (`python3`, `vim`, …) |
| `/usr/sbin` | Extra admin tools (`sshd`, …) |
| `/usr/lib` | Helper libraries for programs in `/usr/bin` |
| `/opt` | Optional / third-party software (e.g. `/opt/google/chrome`) |
| `/bin` | Essential binaries available to all users |
| `/sbin` | System binaries — commands for managing/fixing the system |
| `/dev` | Device files — represent hardware (e.g. `/dev/sda`) |
| `/proc` | Virtual filesystem exposing kernel & process info (e.g. `/proc/cpuinfo`) |
| `/sys` | Live info about connected devices (e.g. `/sys/class/net/`) |
| `/var` | Variable data — logs (`/var/log/syslog`), caches, spool files |
| `/tmp` | Temporary files (cleared on reboot) |
| `/mnt` | Temporary mount point for manually mounted filesystems |
| `/media` | Auto-mount point for removable media (USB, CD) |
| `/lib` | Shared libraries needed by `/bin` and `/sbin` |

> [!NOTE]
> **Why do `/bin` and `/usr/bin` both exist?**
> Long ago, computers booted in small steps — `/bin` held the tools needed immediately at startup, and `/usr/bin` had the rest. On modern systems, everything loads at once, so `/bin` is typically just a symlink pointing to `/usr/bin`.

---

## 6. Virtual File System (VFS)

In Linux, **everything is treated as a file** — regular files, directories, hardware devices, and even live kernel data.

```bash
vim somefile.txt       # editing a regular file
cat somefile.txt       # reading a regular file
cat /proc/cpuinfo      # reading live CPU info — also just a file!
```

The **Virtual File System (VFS)** is the kernel abstraction layer that makes this possible. It provides a unified interface so that the same `open()`, `read()`, `write()` system calls work regardless of the underlying filesystem type.

```
cat /proc/cpuinfo
        |
    Kernel ──────────────► Virtual File System (VFS)
                                    |
                    ┌───────────────┼───────────────┐
                  Ext4            proc             XFS
               (disk FS)      (kernel info)    (disk FS)
```

| Filesystem | Role |
|-----------|------|
| `Ext4` | Standard Linux disk filesystem |
| `proc` | Virtual FS exposing live kernel/process data |
| `XFS` | High-performance disk filesystem (common in RHEL) |

---

## 7. Essential Commands

### 7.1 Listing & Navigation

#### `pwd` — Print Working Directory

```bash
pwd                 # print the full path of the current directory
```

#### `ls` — List Directory Contents

```bash
ls                  # list files in current directory
ls -l               # long format: permissions, owner, size, date
ls -a               # show hidden files (starting with .)
ls -la              # long format including hidden files
ls -lh              # long format with human-readable sizes (KB, MB)
ls -lt              # sort by time (newest first)
ls -ls              # sort by size (largest first)
ls -R               # recursively list all subdirectories

# Find the largest files (disk space debugging)
ls -laSh | head -10

# Find recently modified files (troubleshooting)
ls -lat | head -10

# Check file permissions quickly
ls -la | grep -E '^-rwx'

# List only directories
ls -la | grep ^d
```

**Example output of `ls -l`:**

```
-rw-r--r-- 1 user group 1234 Nov 24 10:30 config.py
│││││││││ │ │     │     │    │             │
│││││││││ │ │     │     │    │             └── filename
│││││││││ │ │     │     │    └── modification time
│││││││││ │ │     │     └── size in bytes
│││││││││ │ │     └── group owner
│││││││││ │ └── user owner
│││││││││ └── number of hard links
│└┴┴┴┴┴┴┴┴── file permissions (see Access Management section)
```

#### `cd` — Change Directory

```bash
cd /etc                       # navigate using an absolute path (starts from /)
cd projects/web-app           # navigate using a relative path (from current location)
cd ../another-project         # go up one level, then into another directory
cd ../../back-two-levels      # go up two levels
cd ~                          # go to your home directory
cd ..                         # go up one level to the parent directory
cd -                          # return to the previous directory
cd /                          # go to the root directory
cd .                          # stay in current directory (useful in scripts)
```

---

### 7.2 File & Directory Management

#### `mkdir` / `rmdir` — Create / Remove Directories

```bash
mkdir projects          # create a new directory
mkdir -p work/logs/2024 # create nested directories in one command
rmdir old_dir           # remove an empty directory
```

#### `touch` — Create Empty Files

```bash
touch newfile.txt                       # create a new empty file (or update timestamp)
touch file1.txt file2.txt file3.txt     # create multiple files at once
```

> [!TIP]
> **`touch` vs `vim`:** `touch` creates an empty file instantly without opening an editor. `vim` creates (and opens) a file for editing — the file is only saved if you write it with `:w`.

#### `rm` — Remove Files or Directories

```bash
rm notes.txt            # delete a file
rm file1.txt file2.txt  # delete multiple files
rm -i notes.txt         # delete with interactive confirmation
rm -r old_project/      # recursively delete a directory and its contents
rm -f temp.log          # force delete without prompting
rm -rf /tmp/cache/      # force recursive delete — use with extreme caution ⚠️
```

#### `cp` — Copy Files or Directories

```bash
cp source.txt destination.txt   # copy file to a new name
cp file.txt /path/to/directory/ # copy a file to a destination directory
cp -r docs/ backup/             # recursively copy a directory
cp -p script.sh /opt/           # preserve permissions, timestamps, ownership
cp -i file.txt /tmp/            # prompt before overwriting an existing file
```

#### `mv` — Move / Rename Files

```bash
mv file.txt /new/location/  # move a file to a new location
mv report.txt final.txt     # rename a file
mv -i data.csv /opt/        # prompt before overwriting
```

#### `find` — Search for Files

```bash
find / -name "test.txt" 2>/dev/null         # find file by name (suppress errors)
find /etc -name "*.conf" 2>/dev/null        # find all .conf files under /etc
find /etc -name "*nginx*" -type f           # find nginx config files specifically
find / -type d -name bin                    # find directories named 'bin'
find /home -mtime -7                        # find files modified in the last 7 days
find / -size +100M                          # find files larger than 100 MB
```

#### `stat` — File Status

```bash
stat /etc/hosts     # show detailed info: size, permissions, timestamps, inode
```

---

### 7.3 Getting Help

```bash
man ls              # open the manual page (most detailed)
ls --help           # quick help summary with common flags
info bash           # more detailed GNU-style documentation
```

---

### 7.4 File Creation & Text Editors

#### Text Editors

| Editor | Description |
|--------|-------------|
| `vi` / `vim` | Powerful modal editor. Press `i` to enter Insert mode, `Esc` to return to Normal, `:wq` to save and quit, `:q!` to quit without saving. |
| `nano` | Beginner-friendly. Shortcuts shown at the bottom. `Ctrl+O` to save, `Ctrl+X` to exit. |

```bash
vim file.txt        # open file in vim
nano file.txt       # open file in nano
```

---

### 7.5 Viewing File Contents

```bash
cat /etc/hosts          # print entire file (best for short files)
tac log.txt             # print file in reverse (last line first)
head -n 20 log.txt      # show the first 20 lines
tail -n 20 log.txt      # show the last 20 lines
tail -f /var/log/syslog # follow file in real time (great for logs)
less big_file.txt       # paginated viewer — scroll up/down, q to quit
more big_file.txt       # older pager — forward only, q to quit
```

> [!TIP]
> Prefer `less` over `more` — it supports both forward and backward scrolling.

---

### 7.6 Text Processing & Filtering

#### `sort` — Sort Lines

```bash
sort names.txt          # sort lines alphabetically
sort -n scores.txt      # sort numerically
sort -r names.txt       # reverse order
sort -u list.txt        # sort and remove duplicate lines
```

#### `grep` — Search Text Patterns

One of the most-used commands in Linux — finds lines matching a pattern.

```bash
grep 'error' app.log        # find lines matching a pattern
grep -i 'ERROR' app.log     # case-insensitive search
grep -r 'TODO' ./src        # recursively search in all files in a directory
grep -n 'fail' log.txt      # show line numbers alongside matches
grep -v '#' config.txt      # show lines that do NOT match (inverse)
grep -c 'error' app.log     # count the number of matching lines
```

#### `cut` — Extract Columns / Fields

```bash
cut -d: -f1 /etc/passwd     # cut field 1 using ':' as delimiter
cut -c1-10 names.txt        # extract characters 1 through 10 from each line
cut -d, -f2,3 data.csv      # extract columns 2 and 3 from a CSV
```

#### `sed` — Stream Editor

Find-and-replace, deletion, and transformation of text.

```bash
sed 's/foo/bar/' file.txt       # replace first occurrence per line
sed 's/foo/bar/g' file.txt      # replace all occurrences (global)
sed -i 's/http/https/g' urls.txt# edit file in-place (modifies actual file)
sed '/^#/d' config.txt          # delete lines matching a pattern
sed -n '5,10p' log.txt          # print lines 5 to 10 only
```

#### `awk` — Pattern Scanning & Processing

A mini-language for processing structured/columnar text.

```bash
awk '{print $1}' log.txt                    # print the first column
awk -F: '{print $1}' /etc/passwd            # use ':' as delimiter, print field 1
awk '$3 > 100' data.txt                     # print lines where column 3 > 100
awk '{sum+=$1} END{print sum}' nums.txt     # sum all values in column 1
awk 'NR==3' file.txt                        # print only line number 3
```

> [!NOTE]
> **Quick reference — when to use which tool:**
>
> | Tool | Use it for |
> |------|-----------|
> | `grep` | Finding lines that match a pattern |
> | `cut` | Extracting specific columns by position |
> | `sed` | Find-and-replace / deleting lines |
> | `awk` | Processing and computing on structured/columnar data |
>
> These are commonly chained with pipes: `command1 | command2 | command3`

---

## 8. Access Management

### 8.1 File Permissions

Every file and directory in Linux has a **permission string** made up of 9 characters, split into three groups of three:

```
rwx rwx rwx
│   │   │
│   │   └── Others  (everyone else on the system)
│   └────── Group   (the file's assigned group)
└────────── User    (the file's owner/creator)
```

Each group uses three permission bits:

| Symbol | Name    | Value | Meaning on a file       | Meaning on a directory        |
|--------|---------|-------|-------------------------|-------------------------------|
| `r`    | read    | 4     | View file contents      | List directory contents       |
| `w`    | write   | 2     | Modify file contents    | Create/delete files inside    |
| `x`    | execute | 1     | Run file as a program   | Enter (`cd` into) directory   |
| `-`    | none    | 0     | Permission not granted  | Permission not granted        |

**Reading the full `ls -l` permission string:**

```
-rw-r--r-- 1 user group 1234 Nov 24 10:30 config.py
↑↑↑↑↑↑↑↑↑↑
│└──┴──┴──┴── permissions (10 chars)
│  │  │  │
│  │  │  └── Others: r-- = read only (4)
│  │  └───── Group:  r-- = read only (4)
│  └──────── User:   rw- = read + write (6)
└─────────── File type: '-' = regular file, 'd' = directory, 'l' = symlink
```

**Common permission presets (octal notation):**

| Octal | Symbolic    | Typical use                     |
|-------|-------------|---------------------------------|
| `755` | `rwxr-xr-x` | Executable files, directories   |
| `644` | `rw-r--r--` | Regular files (docs, configs)   |
| `600` | `rw-------` | Private files (SSH keys, etc.)  |
| `777` | `rwxrwxrwx` | ⚠️ Dangerous — avoid!           |

---

### 8.2 `chmod` — Change Permissions

```bash
# Standard file permissions
chmod 644 document.txt      # rw-r--r--

# Standard executable permissions
chmod 755 script.sh         # rwxr-xr-x

# Private file permissions
chmod 600 private_key.pem   # rw-------

# Directory permissions
chmod 755 /path/to/directory
```

---

### 8.3 `chown` — Change Ownership

```bash
# Change owner
chown newowner file.txt

# Change owner and group
chown newowner:newgroup file.txt

# Change only group
chown :newgroup file.txt
# or
chgrp newgroup file.txt
```

---

### 8.4 `chattr` — Change File Attributes

`chattr` sets special low-level attributes on files, beyond standard permissions.

```bash
# Make file immutable (cannot be deleted or modified — even by root)
chattr +i critical_config.txt

# Make file append-only (useful for log files)
chattr +a /var/log/application.log

# Remove immutable attribute
chattr -i critical_config.txt
```

---

### 8.5 `setfacl` — Access Control Lists

ACLs allow fine-grained per-user or per-group permissions beyond the standard owner/group/others model.

```bash
# Give a specific user read access
setfacl -m u:john:r file.txt

# Give a specific group write access
setfacl -m g:developers:rw project_file.txt

# Set default ACLs for a directory (inherited by new files)
setfacl -d -m g:webadmins:rwx /var/www/

# Remove an ACL entry
setfacl -x u:john file.txt
```

---

## 9. Hard Links vs Symbolic Links

Linux has two types of links — both let multiple names point to the same content, but they work very differently.

### Hard Links — Multiple Book Copies

> Think of printing multiple identical copies of a book. Each copy is complete and independent. Destroying one copy leaves all others intact. But if you want to update the content, you must update every copy manually.

- Both names point to the **same inode** (same data on disk)
- Deleting one link does **not** delete the data — data is only removed when all hard links are gone
- Cannot span across different filesystems
- Cannot link to directories

```bash
# Create a hard link
ln original_file.txt hard_link.txt

# Verify: both share the same inode number (first column)
ls -li original_file.txt hard_link.txt
```

---

### Symbolic (Soft) Links — Bookmarks

> Think of a bookmark in a book. The bookmark isn't the book itself — it just tells you where to find it. If someone moves the book, the bookmark becomes useless. But bookmarks are lightweight and can point to books in different buildings (even different filesystems).

- The symlink stores the **path** to the target, not the data itself
- If the original file is moved or deleted, the symlink **breaks** (becomes a dangling link)
- Can span across different filesystems
- Can link to directories

```bash
# Create a symbolic link
ln -s /path/to/original_file.txt symbolic_link.txt

# Inspect the link — note the '->' showing where it points
ls -la symbolic_link.txt
```

---

### Hard Link vs Symbolic Link — Comparison

| Feature | Hard Link | Symbolic Link |
|---------|-----------|---------------|
| Points to | Inode (data on disk) | File path (name) |
| Survives original deletion? | ✅ Yes | ❌ No (becomes dangling) |
| Cross-filesystem? | ❌ No | ✅ Yes |
| Link to directories? | ❌ No | ✅ Yes |
| Detectable as a link? | Hard (same inode) | Easy (`ls -la` shows `->`) |
| Created with | `ln target link` | `ln -s target link` |
