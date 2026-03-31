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
- [5. Linux Directory Structure](#5-linux-directory-structure)
- [6. Essential Commands](#6-essential-commands)
  - [Listing & Navigation](#61-listing--navigation)
  - [File & Directory Management](#62-file--directory-management)
  - [Getting Help](#63-getting-help)
  - [File Creation & Editors](#64-file-creation--text-editors)
  - [Viewing Files](#65-viewing-file-contents)
  - [Text Processing](#66-text-processing--filtering)

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

## 5. Linux Directory Structure

Linux uses a single hierarchical tree rooted at `/` (root). Everything — files, devices, processes — lives somewhere in this tree.

```
/
├── root/       ← home of the superuser
├── home/       ← home dirs for regular users
├── boot/       ← kernel and bootloader files
├── etc/        ← system configuration files
├── usr/        ← user utilities and programs
├── opt/        ← optional / third-party software
├── bin/        ← essential binaries (ls, cp, …)
├── sbin/       ← system/admin binaries (fdisk, …)
├── dev/        ← device files (/dev/sda, …)
├── proc/       ← virtual FS exposing kernel info
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
| `/etc` | System-wide configuration files |
| `/usr` | User utilities, libraries, and installed programs |
| `/opt` | Optional / third-party software installations |
| `/bin` | Essential binaries available to all users |
| `/sbin` | System binaries (admin/root commands) |
| `/dev` | Device files — represent hardware |
| `/proc` | Virtual filesystem exposing kernel & process info |
| `/var` | Variable data — logs, caches, spool files |
| `/tmp` | Temporary files (cleared on reboot) |
| `/mnt` | Temporary mount point for manually mounted filesystems |
| `/media` | Auto-mount point for removable media (USB, CD) |
| `/lib` | Shared libraries needed by `/bin` and `/sbin` |

---

## 6. Essential Commands

### 6.1 Listing & Navigation

#### `ls` — List Directory Contents

```bash
ls                  # list files in current directory
ls -l               # long format: permissions, owner, size, date
ls -a               # show hidden files (starting with .)
ls -la              # long format including hidden files
ls -lh              # long format with human-readable sizes (KB, MB)
ls -R               # recursively list all subdirectories
```

#### `cd` — Change Directory

```bash
cd /etc             # navigate to a specific directory
cd ..               # go up one level to the parent directory
cd ~                # go to your home directory
cd -                # return to the previous directory
cd /                # go to the root directory
```

#### `pwd` — Print Working Directory

```bash
pwd                 # print the full path of the current directory
```

---

### 6.2 File & Directory Management

#### `mkdir` / `rmdir` — Create / Remove Directories

```bash
mkdir projects          # create a new directory
mkdir -p work/logs/2024 # create nested directories in one command
rmdir old_dir           # remove an empty directory
```

#### `rm` — Remove Files or Directories

```bash
rm notes.txt            # delete a file
rm -i notes.txt         # delete with interactive confirmation
rm -r old_project/      # recursively delete a directory and its contents
rm -f temp.log          # force delete without prompting
rm -rf /tmp/cache/      # force recursive delete — use with extreme caution ⚠️
```

#### `cp` — Copy Files or Directories

```bash
cp file.txt /tmp/       # copy a file to a destination
cp -r docs/ backup/     # recursively copy a directory
cp -p script.sh /opt/   # preserve permissions, timestamps, ownership
cp -i file.txt /tmp/    # prompt before overwriting an existing file
```

#### `mv` — Move / Rename Files

```bash
mv file.txt /tmp/       # move a file to a new location
mv report.txt final.txt # rename a file
mv -i data.csv /opt/    # prompt before overwriting
```

#### `find` — Search for Files

```bash
find /var -name '*.log'         # find all .log files under /var
find / -type d -name bin        # find directories named 'bin'
find /home -mtime -7            # find files modified in the last 7 days
find / -size +100M              # find files larger than 100 MB
```

#### `stat` — File Status

```bash
stat /etc/hosts     # show detailed info: size, permissions, timestamps, inode
```

---

### 6.3 Getting Help

```bash
man ls              # open the manual page (most detailed)
ls --help           # quick help summary with common flags
info bash           # more detailed GNU-style documentation
```

---

### 6.4 File Creation & Text Editors

#### `touch` — Create Empty Files / Update Timestamps

```bash
touch notes.txt             # create a new empty file (or update timestamp)
touch a.txt b.txt c.txt     # create multiple files at once
```

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

### 6.5 Viewing File Contents

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

### 6.6 Text Processing & Filtering

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

---

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
