# Chapter 1 — Core Foundations

---

## Table of Contents

- [1.0 Introduction to Linux](#10-introduction-to-linux)
  - [Computer Basics](#computer-basics)
  - [Key Definitions](#key-definitions)
  - [What is Linux](#what-is-linux)
  - [Linux Distributions](#linux-distributions-distros)
  - [Virtual File System (VFS)](#virtual-file-system-vfs)
- [1.1 Linux Architecture — Kernel, Shell & System Calls](#11-linux-architecture--kernel-shell--system-calls)
- [1.2 File Hierarchy Standard (FHS)](#12-file-hierarchy-standard-fhs)
- [1.3 File Permissions](#13-file-permissions)
- [1.4 Hard Links vs Symbolic Links](#14-hard-links-vs-symbolic-links)
- [1.5 Package Management](#15-package-management)
- [1.6 CLI Text Processing — grep, sed, awk](#16-cli-text-processing--grep-sed-awk)
- [1.7 Essential Commands](#17-essential-commands)

---

## 1.0 Introduction to Linux

### Computer Basics

A computer is made up of two fundamental components:

| Component | Examples |
|-----------|----------|
| **Hardware** | CPU, Memory (RAM), Hard Disk, Peripheral Devices, Networking Devices |
| **Software** | Programs, Databases, Operating System, Application Software |

---

### Key Definitions

#### Program

A **program** is a *passive file* stored on disk — a set of instructions that tells a computer what to do. It does nothing until it is executed.

#### Operating System (OS)

An **OS** is a collection of system software that manages a computer's hardware resources and provides services to applications running on top of it.

#### Kernel

The **kernel** is the core foundational component of the OS. It boots the OS and acts as the primary bridge between hardware and software applications.

#### Shell

The **shell** is a user-facing interface used to interact with the kernel and OS. It accepts commands and passes them to the kernel for execution.

Examples: `bash`, `zsh`, `sh`

#### Process

A **process** is a program that is actively executing — it has been given CPU time, memory, and system resources.

> **Program vs Process**
>
> | | |
> |---|---|
> | **Program** | Passive file sitting on disk |
> | **Process** | Program actively executing with CPU & memory assigned |
>
> Simply loading a program into RAM alone does **not** make it a process — it must be *actively executing*.

---

### What is Linux

**Linux** is an open-source, Unix-like OS kernel. While technically just the kernel, the term "Linux" commonly refers to a full **Linux distribution** — the kernel bundled with system software, tools, and package managers.

Linux is widely used in:
- Servers and data centres
- Supercomputers *(over 90% of the world's top 500 run Linux)*
- Embedded systems (routers, IoT devices)
- Mobile devices (Android is built on the Linux kernel)

---

### Linux Distributions (Distros)

A Linux **distro** is a complete OS built around the Linux kernel. Distros differ in the programs, tools, package managers, and configuration they bundle.

| Distribution | Type | Package Manager |
|---|---|---|
| **Debian** | Community / Free | `apt` |
| **Ubuntu** | Community / Free (based on Debian) | `apt` |
| **CentOS Stream** | Community / Free (upstream preview of RHEL) | `dnf` / `yum` |
| **RHEL** | Enterprise / Paid (Red Hat) | `dnf` (modern), `yum` (legacy) |
| **Android** | Mobile (based on Linux kernel) | — |

> - **RHEL family** (RHEL, CentOS Stream) → `dnf` / `yum` — enterprise-focused
> - **Debian family** (Debian, Ubuntu) → `apt` — community-focused
> - `dnf` is the modern replacement for `yum`

---

### Virtual File System (VFS)

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

## 1.1 Linux Architecture — Kernel, Shell & System Calls

### The Three Layers

```
+-------------------------------------------------------+
|  User Space                                           |
|    [ User Apps: Chrome, VS Code, Database, CLI Tools] |
|                           |                           |
|                    (Shell / GUI)                      |
|                           v                           |
|                  ====================                 |
|                   SYSTEM CALL INTERFACE               |
|                  ====================                 |
+-------------------------------------------------------+
|  Kernel Space             |                           |
|                           v                           |
|         [ Kernel: Process, Memory, Device Drivers ]   |
+-------------------------------------------------------+
|  Hardware                                             |
|         [ Physical: CPU, RAM, SSD/HDD, NIC ]         |
+-------------------------------------------------------+
```

| Layer | Execution Mode | What lives here |
|-------|---------------|-----------------|
| **Hardware** | — | CPU, RAM, SSD, NIC — raw physical components |
| **Kernel Space** | Ring 0 (unrestricted) | Kernel, device drivers, memory manager |
| **User Space** | Ring 3 (restricted) | Shell, apps, everything you interact with |

> **Why the split?** If your browser crashes in User Space, the CPU blocks it from touching hardware. It cannot wipe your disk or crash the CPU. Kernel crash = entire machine dies (Kernel Panic).

---

### The Shell

- A User Space application that reads commands and translates them into OS instructions
- Does **not** execute commands itself — it finds the binary and asks the system to run it
- `mkdir SecretFolder` → shell parses text → finds `mkdir` binary → triggers `execve()` syscall

| Shell | Notes |
|-------|-------|
| `bash` | Bourne Again Shell — standard default |
| `zsh` | Highly customizable, modern Mac/Linux |
| `sh` | POSIX-compliant minimal shell |

---

### The Kernel

The absolute manager of the machine. Has direct, unrestricted hardware access.

| Responsibility | What it does |
|----------------|-------------|
| **Process Management** | Decides which app gets CPU time and for how long |
| **Memory Management** | Allocates RAM blocks — ensures App A cannot read App B's memory |
| **Device Drivers** | Translates OS calls to hardware-specific signals (GPU, NIC, etc.) |
| **File System Management** | Organizes how data is structured, saved, read from disks |

---

### System Calls (Syscalls)

A syscall is a **programmatic request** from User Space that switches the CPU into Kernel Mode so the kernel can perform a privileged task on behalf of the application.

**Lifecycle of a syscall — e.g. saving a file:**

1. You click Save in a text editor
2. Application triggers the `write()` system call
3. CPU pauses the app, switches **User Mode → Kernel Mode**
4. Kernel validates the request (disk space, write permission)
5. Kernel talks to disk driver, physically writes bytes to SSD
6. Kernel switches CPU back to **User Mode**, returns success code to app

**Core syscalls:**

| Category | Syscall | Description |
|----------|---------|-------------|
| **File** | `open()` | Opens a file, returns a file descriptor |
| **File** | `read()` | Reads data from a file descriptor |
| **File** | `write()` | Writes data to a file descriptor |
| **File** | `close()` | Closes a file descriptor |
| **File** | `chmod()` | Changes file permissions |
| **Process** | `fork()` | Creates a child process (copy of parent) |
| **Process** | `execve()` | Replaces current process with a new program (used by shell for every command) |
| **Process** | `exit()` | Terminates the current process |
| **Process** | `wait()` | Parent waits for child to finish |
| **Network** | `socket()` | Creates a network socket |
| **Network** | `connect()` | Connects socket to a remote address |
| **Network** | `send()` | Sends data over socket |

> **Note on memory calls:** `malloc()`, `mmap()`, `brk()` are often listed as syscalls but are technically C library (libc) wrappers that eventually call kernel syscalls underneath.

---

### Monolithic Kernel & Loadable Kernel Modules (LKMs)

**Monolithic** = all core OS services (process manager, memory manager, VFS, all device drivers) run together inside a **single address space in Kernel Mode**.

| | Monolithic (Linux) | Microkernel (QNX, Mach) |
|---|---|---|
| **Performance** | High — components talk in same memory space, no mode switching | Slower — components separated into user-space servers, requires IPC messaging |
| **Stability** | A buggy driver in kernel space can crash the whole system | A driver crash only kills that isolated process; kernel keeps running |

**Loadable Kernel Modules (LKMs)** — the modern hybrid solution:
- Linux kernel is monolithic in *execution* but modular in *design*
- Drivers and features (`.ko` files) can be loaded/unloaded **on the fly, without reboot**

```bash
insmod  module.ko       # insert/load a kernel module
modprobe module_name    # load module + auto-resolve dependencies
rmmod   module_name     # remove/unload a kernel module
lsmod                   # list all currently loaded modules
```

---

### Key Distinctions

| Term | One-line definition |
|------|-------------------|
| **Program** | Passive file sitting on disk — does nothing until executed |
| **Process** | Program actively executing with CPU time and memory assigned |
| **Shell** | Translator between user and OS |
| **Kernel** | Manager of CPU, memory, and hardware |
| **Syscall** | The explicit, secure door from User Space into Kernel Space |
| **Monolithic** | Everything fast in one kernel space, made flexible via LKMs |

---

## 1.2 File Hierarchy Standard (FHS)

Linux uses a single hierarchical tree rooted at `/`. Everything — files, devices, processes — lives somewhere in this tree.

```
/
├── root/       ← home of the superuser
├── home/       ← home dirs for regular users
├── boot/       ← kernel and bootloader files
├── etc/        ← system configuration files
├── usr/
│   ├── bin/    ← most regular programs (python3, vim, …)
│   ├── sbin/   ← extra admin tools (sshd, …)
│   └── lib/    ← helper files for /usr/bin programs
├── opt/        ← optional / third-party software
├── bin/        ← essential binaries [symlink → /usr/bin on modern systems]
├── sbin/       ← system/admin binaries (reboot, fdisk, …)
├── dev/        ← device files (/dev/sda, …)
├── proc/       ← virtual FS exposing kernel/process info
├── sys/        ← live info about connected devices
├── var/        ← logs, caches, spool files
├── tmp/        ← temporary files (cleared on reboot)
├── mnt/        ← manual mount point
├── media/      ← auto-mount for USB, CD
└── lib/        ← shared libraries for /bin & /sbin
```

| Directory | Purpose |
|-----------|---------|
| `/` | Root of the entire filesystem |
| `/root` | Home directory for the root (superuser) account |
| `/home` | Home directories for regular users (`/home/alice`) |
| `/boot` | Bootloader files and the Linux kernel image |
| `/etc` | System-wide configuration files |
| `/usr/bin` | Most regular programs (`python3`, `vim`) |
| `/usr/sbin` | Extra admin tools (`sshd`) |
| `/usr/lib` | Helper libraries for `/usr/bin` programs |
| `/opt` | Optional / third-party software (`/opt/google/chrome`) |
| `/bin` | Essential binaries available to all users |
| `/sbin` | System binaries for managing/fixing the system |
| `/dev` | Device files — represent hardware (`/dev/sda`) |
| `/proc` | Virtual FS exposing kernel & process info (`/proc/cpuinfo`) |
| `/sys` | Live info about connected devices (`/sys/class/net/`) |
| `/var` | Variable data — logs, caches, spool files |
| `/tmp` | Temporary files (cleared on reboot) |
| `/mnt` | Temporary mount point for manually mounted filesystems |
| `/media` | Auto-mount point for removable media (USB, CD) |
| `/lib` | Shared libraries needed by `/bin` and `/sbin` |

> **Why do `/bin` and `/usr/bin` both exist?** Legacy: `/bin` held tools needed immediately at early boot; `/usr/bin` had the rest. On modern systems, `/bin` is typically just a symlink to `/usr/bin`.

---

### Critical Sub-Paths for Troubleshooting

**Inside `/etc`:**

| File | Purpose |
|------|---------|
| `/etc/passwd` | User account info (usernames, UIDs, home dirs, default shells). Stores `x` placeholder — **NOT the actual password** |
| `/etc/shadow` | Actual encrypted passwords. **Root-readable only** |
| `/etc/group` | Group definitions and memberships |
| `/etc/fstab` | File System Table — what partitions/drives mount automatically at boot |
| `/etc/hosts` | Static hostname-to-IP mappings |
| `/etc/ssh/sshd_config` | SSH daemon configuration |

**Inside `/var`:**

| Path | Purpose |
|------|---------|
| `/var/log/` | The primary troubleshooting directory |
| `/var/log/syslog` | General system logs (Debian/Ubuntu) |
| `/var/log/messages` | General system logs (RHEL/CentOS) |
| `/var/log/auth.log` | Authentication logs — failed SSH, sudo usage (Debian/Ubuntu) |
| `/var/log/secure` | Authentication logs (RHEL/CentOS) |

---

### `/proc` vs `/sys` vs `/dev` — Don't Mix These Up

| | `/dev` | `/proc` | `/sys` |
|---|---|---|---|
| **What it is** | Pointers to physical hardware devices | Virtual FS — kernel creates it in RAM dynamically | Virtual FS for live device attributes |
| **Lives on disk?** | Yes (device nodes) | No — exists only in memory | No — exists only in memory |
| **Example** | `/dev/sda` = your hard drive. Write to it directly = raw disk blocks | `cat /proc/cpuinfo` = live CPU specs. `/proc/1234/` = info on PID 1234 | `/sys/class/net/` = NIC states. Can alter device settings on the fly |

```bash
cat /proc/cpuinfo       # live CPU information
cat /proc/meminfo       # live RAM statistics
ls /proc/1234/          # everything about process with PID 1234
```

---

### FHS Design Philosophy (Static vs Dynamic, Shareable vs Unshareable)

| | **Shareable** (can be network-hosted, multi-machine) | **Unshareable** (unique to this machine) |
|---|---|---|
| **Static** (doesn't change without admin action) | `/usr/bin`, `/usr/share` | `/etc`, `/boot` |
| **Variable** (changes automatically at runtime) | `/home`, `/var/mail` | `/var/log`, `/var/run` |

---

## 1.3 File Permissions

### Permission String Anatomy

```
 -  rwx  r-x  r--
 ┬  ─┬─  ─┬─  ─┬─
 │   │    │    └── Others (everyone else)
 │   │    └─────── Group
 │   └──────────── User (owner)
 └──────────────── File type: - = file, d = directory, l = symlink
```

### Permission Behavior: File vs Directory

| Permission | On a **File** | On a **Directory** |
|-----------|--------------|-------------------|
| `r` (4) | View file contents (`cat`) | List contents (`ls`) |
| `w` (2) | Modify/write to file | Create, delete, rename files inside |
| `x` (1) | Execute as a program | `cd` into the directory |

> ⚠️ **Classic trap:** `w` on a *directory* lets you delete files inside it even if you have `000` permissions on the individual files themselves.

---

### Octal (Absolute) Notation

Each permission maps to a numeric weight: `r=4`, `w=2`, `x=1`, `-=0`. Add them per entity:

```
rwx = 4+2+1 = 7
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4

chmod 754 file.txt   → owner: rwx, group: r-x, others: r--
```

| Octal | Symbolic | Typical Use |
|-------|----------|-------------|
| `755` | `rwxr-xr-x` | Executables, directories |
| `644` | `rw-r--r--` | Regular files, configs |
| `600` | `rw-------` | Private files (SSH keys) |
| `700` | `rwx------` | Private scripts |
| `777` | `rwxrwxrwx` | ⚠️ Avoid in production |

---

### Symbolic Notation

Syntax: `[entity][operator][permission]`

| Entity | Operator | Permission |
|--------|----------|-----------|
| `u` = owner | `+` = add | `r` = read |
| `g` = group | `-` = remove | `w` = write |
| `o` = others | `=` = set exactly | `x` = execute |
| `a` = all | | |

```bash
chmod u+x script.sh             # add execute for owner
chmod go-w draft.txt            # remove write from group and others
chmod g=r config.json           # set group to read-only exactly
chmod u=rwx,g=rx,o=r file.txt   # set all three at once
chmod -R 755 /var/www/          # apply recursively
```

> Use symbolic when you want to change *one bit* without recalculating the whole string.

---

### Changing Ownership

```bash
chown alice file.txt                    # change owner
chown alice:devops file.txt             # change owner AND group
chown :devops file.txt                  # change group only
chgrp devops file.txt                   # change group only (explicit)
chown -R webadmin:www-data /var/www/    # recursive ownership change
```

---

### Special Permissions: SUID, SGID, Sticky Bit

#### SUID — Set User ID

- When an executable with SUID is run, the **process runs as the file's owner** (not the user who ran it)
- Classic example: `passwd` command — regular users need to write to `/etc/shadow` (root-only), SUID lets them do it temporarily as root
- Shown as `s` in the owner execute position

```
-rwsr-xr-x  1 root root  /usr/bin/passwd
```

```bash
chmod u+s filename      # set SUID (symbolic)
chmod 4755 filename     # set SUID (octal — the 4 prefix)
```

#### SGID — Set Group ID

- **On a file:** Process runs with the file's group permissions
- **On a directory (most common use):** New files/subdirectories created inside automatically **inherit the directory's group**, not the creator's primary group

```
drwxrwsr-x  2 root marketing 4096 /opt/marketing
```

Real-world: Alice (`primary group: alice`) creates a file in `/opt/marketing` with SGID set → file belongs to `marketing` group, not `alice` → Bob (also in `marketing`) can edit it.

```bash
chmod g+s /dir          # set SGID (symbolic)
chmod 2775 /dir         # set SGID (octal — the 2 prefix)
```

#### Sticky Bit

- Applied to a **directory**: only the **file owner**, **directory owner**, or **root** can delete/rename files inside it — even if others have write permission on the directory
- Classic example: `/tmp` — everyone can write there, but no one can delete another user's files

```
drwxrwxrwt 15 root root 4096 /tmp
```

```bash
chmod +t /dir           # set sticky bit (symbolic)
chmod 1777 /dir         # set sticky bit (octal — the 1 prefix)
```

#### Special Permission Quick Reference

| | Prefix | Appears as | Applied to |
|---|---|---|---|
| **SUID** | `4` | `s` at owner execute | Files |
| **SGID** | `2` | `s` at group execute | Files & Directories |
| **Sticky** | `1` | `t` at others execute | Directories |

> ⚠️ **Capital `S` or `T`** means the special bit is set but the underlying execute bit (`x`) is **not** set — the special permission exists but can never fire. Almost always a misconfiguration.

---

### Special File Attributes (`chattr`)

Beyond standard permissions — low-level attributes the kernel enforces even against root.

```bash
chattr +i file.txt              # immutable — cannot be modified, deleted, or renamed (even by root)
chattr +a /var/log/app.log      # append-only — can only add to file, not overwrite
chattr -i file.txt              # remove immutable attribute
lsattr file.txt                 # view current attributes
```

---

### Access Control Lists (`setfacl`) — Per-User/Per-Group Fine-Grained Permissions

Standard permissions only allow one owner + one group. ACLs allow per-user or per-group rules on top.

```bash
setfacl -m u:john:r file.txt            # give john read access
setfacl -m g:developers:rw project.txt  # give developers group read+write
setfacl -d -m g:webadmins:rwx /var/www/ # set default ACL (inherited by new files)
setfacl -x u:john file.txt              # remove john's ACL entry
getfacl file.txt                         # view all ACL entries on a file
```

---

## 1.4 Hard Links vs Symbolic Links

### What a File Actually Is

A file in Linux has two distinct parts:

- **Inode (Index Node):** A hidden database record storing all *metadata* — size, permissions, timestamps, ownership, and physical disk block pointers. **The inode does NOT store the filename.**
- **Directory Entry:** A filename is just a pointer in a directory map that points to an inode number.

---

### Hard Links

An **additional directory entry** pointing to the **exact same inode** as the original.

- Both names share identical permissions, size, and data blocks — they are the same file with two names
- The kernel tracks an **Inode Link Count** — increments by 1 per hard link created
- Deleting one name reduces the link count. **Data is only erased when link count hits 0**
- Cannot cross filesystem/partition boundaries (inodes are unique per partition)
- Cannot link to directories (prevents infinite loops)

> Think of printing multiple identical copies of a book. Each copy is complete and independent. Destroying one copy leaves all others intact.

```bash
ln original.txt hard_pointer.txt        # create a hard link
```

### Symbolic (Soft) Links

An **entirely separate file** with its own unique inode, whose data block contains nothing but the **pathname** of the target.

- Acts like a desktop shortcut — redirects you to the target path
- If the original is moved or deleted → **dangling/broken link** — the path it stores no longer exists
- Can cross filesystem boundaries
- Can link to directories

> Think of a bookmark in a book. The bookmark isn't the book itself — it just tells you where to find it. If someone moves the book, the bookmark becomes useless.

```bash
ln -s /path/to/original.txt soft_link.txt   # create a symbolic link (-s flag required)
```

---

### Inspecting Links

```bash
ls -li          # -i shows inode numbers alongside filenames
```

```
3492834 -rw-r--r--  2 user team  12 Jun 16 18:00 original.txt
3492834 -rw-r--r--  2 user team  12 Jun 16 18:00 hard_pointer.txt
8593022 lrwxrwxrwx  1 user team  12 Jun 16 18:01 soft_link.txt -> original.txt
```

- `original.txt` and `hard_pointer.txt` share inode `3492834`, link count = `2`
- `soft_link.txt` has its own inode `8593022`, link count = `1`, shows `->` target path

---

### What Happens When You Delete the Source?

| | Hard Link | Symbolic Link |
|---|---|---|
| Source deleted | Link count drops by 1. If count > 0, **data stays intact** | Link still exists but becomes a **dangling/broken link** |
| Access still works? | ✅ Yes — data survives until all hard links gone | ❌ No — `cat soft_link.txt` → "No such file or directory" |

---

### Comparison Table

| Feature | Hard Link | Symbolic Link |
|---------|-----------|---------------|
| Points to | Inode (data on disk) | File path (text string) |
| Own inode? | ❌ Shares original inode | ✅ Has its own inode |
| Survives source deletion? | ✅ Yes (until link count = 0) | ❌ No (becomes dangling) |
| Cross-filesystem? | ❌ No | ✅ Yes |
| Link to directories? | ❌ No | ✅ Yes |
| Detectable as a link? | Hard — same inode | Easy — `ls -la` shows `->` |
| Created with | `ln target link` | `ln -s target link` |

> **Diagnostic:** If you `chmod 400 hard_pointer.txt` — the permissions on `original.txt` **also change** because they share the same inode (one permission record). If you `chmod` a symlink — it affects the **target file**, not the symlink itself (`lrwxrwxrwx` permissions on symlinks are cosmetic and ignored by the kernel).

---

## 1.5 Package Management

### The Problem: Dependency Hell

Early Linux required downloading raw source code, compiling with `make`, and manually tracking every `.so` library needed. One missing library = failed install. Modern package managers solve this with a centralized ecosystem.

---

### Three-Pillar Architecture

```
+-------------------------------------------------------------+
| 1. Repository (Remote Server)                               |
|    Stores thousands of .deb / .rpm files + metadata index   |
+-------------------------------------------------------------+
                              |
                    apt update / dnf check-update
                              v
+-------------------------------------------------------------+
| 2. Local Cache / Index (On Your Machine)                    |
|    Local DB matching package names to versions & deps        |
+-------------------------------------------------------------+
                              |
                    apt install / dnf install
                              v
+-------------------------------------------------------------+
| 3. Package Manager (The Engine)                             |
|    Downloads, resolves dependencies, installs               |
+-------------------------------------------------------------+
```

- **Package:** Compressed archive containing pre-compiled binaries, config files, man pages, and a dependency manifest
- **Repository:** Remote server hosted by the distro — stores verified packages + an index file tracking versions/dependencies
- **Package Manager:** CLI tool that reads the local index, fetches packages over HTTPS, and hands off to the low-level installer

---

### APT vs YUM/DNF

| | Debian / Ubuntu | Red Hat / RHEL / Fedora |
|---|---|---|
| **Low-level tool** | `dpkg` | `rpm` |
| **High-level manager** | `apt` | `yum` (legacy) / `dnf` (modern) |
| **Package format** | `.deb` | `.rpm` |
| **Repo config location** | `/etc/apt/sources.list` & `/etc/apt/sources.list.d/` | `/etc/yum.repos.d/` |

> `dnf` is the modern drop-in replacement for `yum`. On current RHEL/CentOS/Fedora systems, prefer `dnf`.

---

### Command Cross-Reference

#### Sync Repository Index — **always run this first on a new/fresh instance**

```bash
sudo apt update                 # Debian/Ubuntu
sudo dnf check-update           # RHEL/Fedora
```

> ⚠️ `apt update` does **not** upgrade software. It only refreshes the local list of what's available.

#### Install a Package

```bash
sudo apt install nginx          # Debian/Ubuntu
sudo dnf install nginx          # RHEL/Fedora
```

Auto-resolves and installs all dependencies first.

#### Upgrade Installed Software

```bash
sudo apt upgrade                # Debian/Ubuntu
sudo dnf upgrade                # RHEL/Fedora
```

#### Remove Software

```bash
# Debian/Ubuntu
sudo apt remove nginx           # remove package, keep config files
sudo apt purge nginx            # remove package AND config files
sudo apt autoremove             # clean up orphaned dependencies

# RHEL/Fedora
sudo dnf remove nginx           # removes package + unneeded dependencies automatically
```

#### Other Useful Commands

```bash
apt list --installed            # list all installed packages (apt)
apt search nginx                # search repository for packages
apt show nginx                  # show package details, dependencies
dpkg -l                         # list all installed packages (low-level)
rpm -qa                         # list all installed packages (rpm)
```

---

### How Dependency Resolution Works

Example: `sudo apt install graphics-tool`

1. `apt` checks local cache for `graphics-tool`
2. Reads package metadata: `Depends: libpng16, libjpeg`
3. Checks if those are already installed; if not, adds them to download queue
4. Recursively checks if `libpng16` has its own dependencies
5. Once full dependency tree is mapped, downloads all `.deb` files to `/var/cache/apt/archives/`
6. Hands files to `dpkg`, which extracts them and places binaries into `/usr/bin/`, `/usr/sbin/`, etc.

---

### Common Troubleshooting Trap

**Scenario:** Fresh Ubuntu cloud instance, run `sudo apt install htop` → `E: Unable to locate package htop`

**Cause:** Fresh instance has an empty/outdated local cache. The system has no index of what packages exist yet.

**Fix:** Run `sudo apt update` first to populate `/var/lib/apt/lists/`.

---

## 1.6 CLI Text Processing — grep, sed, awk

In production, you never open a 2GB log file in an editor — that can exhaust memory and crash your terminal. You stream and slice data directly on the CLI.

**Role of each tool:**
- `grep` — line-oriented: finds lines matching a pattern
- `sed` — modification-oriented: replaces, deletes, or transforms text strings
- `awk` — column/field-oriented: treats lines as data rows, allows math, logic, reports

---

### `grep` — The Pattern Matcher

```bash
grep [options] "pattern" filename
```

#### Key Flags

| Flag | Purpose |
|------|---------|
| `-i` | Case-insensitive match |
| `-v` | Invert match (show lines that do NOT match) |
| `-c` | Count matching lines |
| `-n` | Show line numbers |
| `-r` | Recursive search through directories |
| `-l` | Show only filenames with matches (not the lines) |
| `-E` | Extended regex (enables `|` OR logic, `+`, `?`) |
| `-B N` | Show N lines **before** each match |
| `-A N` | Show N lines **after** each match |

#### Production One-Liners

```bash
# Find all errors (case-insensitive)
grep -i "error" /var/log/syslog

# Count how many times an IP hit your server
grep -c "192.168.1.50" /var/log/nginx/access.log

# Everything except successful 200 responses
grep -v "HTTP/1.1 200" /var/log/nginx/access.log

# See what happened 2 lines before and 5 lines after a crash event
grep -B 2 -A 5 "CRITICAL_EXCEPTION" /var/log/myapp.log

# Find which config files mention PermitRootLogin (filename only)
grep -rl "PermitRootLogin" /etc/ssh/

# Find multiple log levels at once (OR logic)
grep -E "CRITICAL|FATAL|EMERGENCY" /var/log/syslog
```

---

### `sed` — The Stream Editor

```bash
sed 's/old_text/new_text/flags' filename
```

#### Key Syntax

| Pattern | Meaning |
|---------|---------|
| `s/old/new/` | Replace first occurrence per line |
| `s/old/new/g` | Replace all occurrences per line (`g` = global) |
| `-i` | Edit file in-place (modifies the actual file) |
| `/pattern/d` | Delete lines matching pattern |
| `-n 'N,Mp'` | Print only lines N through M |
| `^#` | Matches lines starting with `#` |
| `^$` | Matches empty lines |

#### Production One-Liners

```bash
# Replace deprecated DB host across a config file
sed 's/db-test.local/db-prod.internal/g' config.yaml

# In-place edit — directly modify the file
sed -i 's/DEBUG/INFO/g' /opt/app/env.conf

# Extract lines 500–550 from a log (precise time slice)
sed -n '500,550p' /var/log/nginx/access.log

# Strip comments and blank lines — see only active config
sed '/^#/d; /^$/d' /etc/nginx/nginx.conf

# Use custom delimiter | to avoid backslash hell with file paths
sed 's|/var/www/html|/opt/nginx/dist|g' config.json
```

---

### `awk` — The Data Processor

```bash
awk 'condition { action }' filename
```

`awk` auto-splits every line into column variables: `$1` = first column, `$2` = second, `$0` = entire line. Default delimiter is whitespace; use `-F` for custom.

#### Key Patterns

| Pattern | Meaning |
|---------|---------|
| `{ print $1 }` | Print column 1 of every line |
| `$3 > 100 { print }` | Print lines where column 3 > 100 |
| `-F:` | Use `:` as field delimiter |
| `END { print sum }` | Run action after all lines processed |
| `NR==3` | Match only line number 3 |

#### Production One-Liners

```bash
# Extract IPs (col 1) and URLs (col 7) from nginx access log
awk '{ print $1, $7 }' /var/log/nginx/access.log

# Find all requests that returned HTTP 500
awk '$9 == 500 { print $1, $7, $9 }' /var/log/nginx/access.log

# Parse /etc/passwd with colon delimiter, print usernames
awk -F: '{ print $1 }' /etc/passwd

# Calculate total bandwidth from nginx log (col 10 = bytes)
awk '{ sum += $10 } END { print "Total:", sum / 1024 / 1024, "MB" }' /var/log/nginx/access.log

# Find processes using more than 10% memory
ps aux | awk '$4 > 10.0 { print $2, $11, $4 }'
# col 4 = %MEM, col 2 = PID, col 11 = command
```

---

### `cut` — Extract Columns by Position

```bash
cut -d: -f1 /etc/passwd         # field 1 using ':' as delimiter
cut -c1-10 names.txt            # extract characters 1–10 from each line
cut -d, -f2,3 data.csv          # extract columns 2 and 3 from a CSV
```

---

### When to Use Which Tool

| Tool | Use it for |
|------|-----------|
| `grep` | Finding lines that match a pattern |
| `cut` | Extracting specific columns by fixed position |
| `sed` | Find-and-replace / deleting lines / line range extraction |
| `awk` | Processing structured/columnar data, math, conditional filtering |

---

### Power Combos — Chained Pipelines

#### Find the Top 5 IPs Hammering Your Server

```bash
cat /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 5
```

| Step | Command | What it does |
|------|---------|-------------|
| 1 | `awk '{print $1}'` | Extract column 1 — the client IP |
| 2 | `sort` | Sort IPs alphabetically so duplicates sit together |
| 3 | `uniq -c` | Count consecutive duplicates → `[count] [IP]` |
| 4 | `sort -nr` | Sort numerically, reverse — highest count first |
| 5 | `head -n 5` | Show only top 5 results |

#### Extract Failed SSH Attempts and Format for Firewall Script

```bash
grep "Failed password for root" /var/log/auth.log | awk '{print $11}' | sed 's/^/block_this_ip=/'
```

| Step | Command | What it does |
|------|---------|-------------|
| 1 | `grep` | Filter lines recording failed root auth attempts |
| 2 | `awk '{print $11}'` | Column 11 = the attacking IP address |
| 3 | `sed 's/^/block_this_ip=/'` | Prepend `block_this_ip=` to each IP line |

---

## 1.7 Essential Commands

### Navigation

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
ls -lS              # sort by size (largest first)
ls -R               # recursively list all subdirectories

# Find the largest files (disk space debugging)
ls -laSh | head -10

# Find recently modified files (troubleshooting)
ls -lat | head -10

# List only directories
ls -la | grep ^d
```

**Reading `ls -l` output:**

```
-rw-r--r-- 1 user group 1234 Nov 24 10:30 config.py
│││││││││ │ │     │     │    │             │
│││││││││ │ │     │     │    │             └── filename
│││││││││ │ │     │     │    └── modification time
│││││││││ │ │     │     └── size in bytes
│││││││││ │ │     └── group owner
│││││││││ │ └── user owner
│││││││││ └── number of hard links
│└┴┴┴┴┴┴┴┴── file permissions
```

#### `cd` — Change Directory

```bash
cd /etc                       # absolute path (starts from /)
cd projects/web-app           # relative path (from current location)
cd ..                         # go up one level to parent directory
cd ../another-project         # go up one level, then into another directory
cd ../../back-two-levels      # go up two levels
cd ~                          # go to your home directory
cd -                          # return to the previous directory
cd /                          # go to root directory
```

---

### File & Directory Management

#### `mkdir` / `rmdir` — Create / Remove Directories

```bash
mkdir projects              # create a new directory
mkdir -p work/logs/2024     # create nested directories in one command
rmdir old_dir               # remove an empty directory
```

#### `touch` — Create Empty Files

```bash
touch newfile.txt                       # create a new empty file (or update timestamp)
touch file1.txt file2.txt file3.txt     # create multiple files at once
```

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
cp file.txt /path/to/dir/       # copy a file to a destination directory
cp -r docs/ backup/             # recursively copy a directory
cp -p script.sh /opt/           # preserve permissions, timestamps, ownership
cp -i file.txt /tmp/            # prompt before overwriting
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
find /home -mtime -7                        # files modified in the last 7 days
find / -size +100M                          # files larger than 100 MB
```

#### `stat` — File Status

```bash
stat /etc/hosts     # show detailed info: size, permissions, timestamps, inode
```

---

### Getting Help

```bash
man ls              # open the manual page (most detailed)
ls --help           # quick help summary with common flags
info bash           # more detailed GNU-style documentation
```

---

### Text Editors

| Editor | Description |
|--------|-------------|
| `vi` / `vim` | Powerful modal editor. `i` = Insert mode, `Esc` = Normal mode, `:wq` = save and quit, `:q!` = quit without saving |
| `nano` | Beginner-friendly. `Ctrl+O` = save, `Ctrl+X` = exit |

```bash
vim file.txt        # open file in vim
nano file.txt       # open file in nano
```

---

### Viewing File Contents

```bash
cat /etc/hosts          # print entire file (best for short files)
tac log.txt             # print file in reverse (last line first)
head -n 20 log.txt      # show the first 20 lines
tail -n 20 log.txt      # show the last 20 lines
tail -f /var/log/syslog # follow file in real time (great for live logs)
less big_file.txt       # paginated viewer — scroll up/down, q to quit
more big_file.txt       # older pager — forward only, q to quit
```

> Prefer `less` over `more` — it supports both forward and backward scrolling.

---

### Sorting & Deduplication

```bash
sort names.txt          # sort lines alphabetically
sort -n scores.txt      # sort numerically
sort -r names.txt       # reverse order
sort -u list.txt        # sort and remove duplicate lines
uniq -c sorted.txt      # count consecutive duplicate lines
```

---

> **Quick Reference — Chapter 1 Commands**
>
> | Category | Commands |
> |----------|----------|
> | **Kernel Modules** | `insmod`, `modprobe`, `rmmod`, `lsmod` |
> | **Navigation** | `pwd`, `ls`, `cd`, `find`, `stat` |
> | **File Management** | `touch`, `mkdir`, `rm`, `cp`, `mv`, `rmdir` |
> | **Viewing Files** | `cat`, `tac`, `head`, `tail`, `less`, `more` |
> | **Permissions** | `chmod`, `chown`, `chgrp`, `chattr`, `lsattr` |
> | **ACLs** | `setfacl`, `getfacl` |
> | **Links** | `ln`, `ln -s`, `ls -li` |
> | **Package (Debian)** | `apt update`, `apt install`, `apt upgrade`, `apt remove`, `apt purge`, `apt autoremove` |
> | **Package (RHEL)** | `dnf install`, `dnf upgrade`, `dnf remove`, `dnf check-update` |
> | **Text Processing** | `grep`, `sed`, `awk`, `cut`, `sort`, `uniq` |
> | **Help** | `man`, `--help`, `info` |
