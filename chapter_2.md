# Chapter 2 — Advanced Linux Commands

---

## Table of Contents

- [1. Networking Commands](#1-networking-commands)
- [2. Permissions & User Management](#2-permissions--user-management)
  - [Understanding Permissions](#21-understanding-permissions)
  - [chmod](#22-chmod--change-file-permissions)
  - [chown](#23-chown--change-ownership)
  - [umask](#24-umask--default-permission-mask)
  - [User Management](#25-user-management)
  - [Group Management](#26-group-management)
- [3. System & Utility Commands](#3-system--utility-commands)
  - [Environment Variables](#31-environment-variables)
  - [Archives & Compression](#32-archives--compression)
  - [System Info & Monitoring](#33-system-info--monitoring)
  - [Scheduling with Cron](#34-scheduling-with-cron--crontab)
- [4. Process & Resource Management](#4-process--resource-management)
  - [top & htop](#41-top--htop--interactive-process-viewers)
  - [ps](#42-ps--process-snapshot)
  - [kill](#43-kill--terminate-processes)
  - [free](#44-free--memory-usage)
  - [df](#45-df--disk-space-usage)
  - [du](#46-du--directory-disk-usage)
  - [uptime](#47-uptime--system-uptime--load)
  - [nice & renice](#48-nice--renice--process-priority)

---

## 1. Networking Commands

### `ping` — Test Network Connectivity

Sends ICMP echo packets to a host to check if it's reachable and measure round-trip time.

```bash
ping google.com             # ping continuously until stopped (Ctrl+C)
ping -c 4 google.com        # send exactly 4 packets then stop
ping -i 2 google.com        # send a packet every 2 seconds
ping -s 1000 google.com     # send packets of 1000 bytes
```

---

### `curl` — Transfer Data via URLs

A powerful tool to make HTTP requests, download files, and interact with APIs — supports HTTP, HTTPS, FTP, and more.

```bash
curl https://example.com                        # fetch and print a webpage
curl -o file.html https://example.com           # save output to a file
curl -I https://example.com                     # show response headers only
curl -X POST -d '{"key":"val"}' \
     -H "Content-Type: application/json" \
     https://api.example.com/endpoint           # POST request with JSON body
curl -u user:pass https://example.com           # basic authentication
curl -L https://example.com                     # follow redirects
```

---

### `wget` — Download Files from the Web

A non-interactive downloader — great for scripting and resuming downloads.

```bash
wget https://example.com/file.zip               # download a file
wget -O output.zip https://example.com/file.zip # download and rename
wget -c https://example.com/file.zip            # resume an interrupted download
wget -r https://example.com                     # recursively download a website
wget -q https://example.com/file.zip            # quiet mode (no output)
```

> [!NOTE]
> **curl vs wget**
> | | `curl` | `wget` |
> |---|---|---|
> | Best for | API calls, sending data, headers | Downloading files, recursive downloads |
> | Output | stdout by default | saves to file by default |
> | Resume | with `-C -` | with `-c` |

---

### `netstat` — Network Statistics *(legacy)*

Displays active network connections, routing tables, and interface stats. Replaced by `ss` on modern systems, but still widely referenced.

```bash
netstat -tuln               # show all listening TCP/UDP ports (numeric)
netstat -an                 # show all connections (active + listening)
netstat -r                  # display the routing table
netstat -i                  # show network interface statistics
netstat -p                  # show PID/program name for each connection
```

> [!TIP]
> On modern Linux systems, `netstat` requires installing `net-tools`. Prefer `ss` instead.

---

### `ss` — Socket Statistics *(modern replacement for netstat)*

Faster and more detailed than `netstat`. Shows socket and connection information.

```bash
ss -tuln                    # show all listening TCP/UDP ports (numeric)
ss -ta                      # show all TCP connections
ss -s                       # summary of socket statistics
ss -p                       # show process using each socket
ss -tnp                     # TCP connections with process names
ss dst 192.168.1.1          # show connections to a specific IP
```

---

### `ip` — Show / Manage Network Interfaces & Routes

The modern replacement for `ifconfig`. Manages IP addresses, routes, and interfaces.

```bash
ip addr                     # show all IP addresses on all interfaces
ip addr show eth0           # show IP info for a specific interface
ip link show                # show link-layer (MAC) info for all interfaces
ip route                    # show the routing table
ip route add default via 192.168.1.1    # add a default gateway
ip link set eth0 up         # bring a network interface up
ip link set eth0 down       # bring a network interface down
```

---

### `hostname` — Show or Set the System Hostname

```bash
hostname                    # print the current hostname
hostname -I                 # print all IP addresses of the machine
hostname -f                 # print the fully qualified domain name (FQDN)
hostnamectl set-hostname newname   # permanently change the hostname
```

---

### `nslookup` — Query DNS Records *(basic)*

Queries DNS servers to resolve hostnames to IPs and vice versa.

```bash
nslookup google.com                     # basic DNS lookup
nslookup google.com 8.8.8.8            # query using a specific DNS server
nslookup -type=MX gmail.com            # look up mail exchange (MX) records
nslookup -type=NS google.com           # look up name server records
```

---

### `dig` — DNS Lookup *(advanced)*

More detailed and flexible than `nslookup`. The preferred tool for DNS troubleshooting.

```bash
dig google.com                          # full DNS query with all details
dig google.com +short                   # short output — IP only
dig google.com MX                       # query MX records
dig google.com NS                       # query name server records
dig @8.8.8.8 google.com                # query using Google's DNS server
dig -x 142.250.80.46                   # reverse DNS lookup (IP → hostname)
```

> [!NOTE]
> **nslookup vs dig**
> | | `nslookup` | `dig` |
> |---|---|---|
> | Output | Simple, human-friendly | Detailed, machine-parseable |
> | Best for | Quick lookups | Debugging, scripting, full DNS inspection |

---

### `vmstat` — Virtual Memory & System Statistics

Reports on memory, swap, I/O, and CPU activity. Useful for spotting performance bottlenecks.

```bash
vmstat                      # single snapshot of system stats
vmstat 2                    # report every 2 seconds continuously
vmstat 2 5                  # report every 2 seconds, 5 times total
vmstat -s                   # show memory stats as a summary table
vmstat -d                   # show disk I/O statistics
```

**Output columns explained:**

| Section | Column | Meaning |
|---------|--------|---------|
| **procs** | `r` | Processes waiting to run |
| **procs** | `b` | Processes in uninterruptible sleep |
| **memory** | `free` | Free RAM (KB) |
| **swap** | `si` / `so` | Swap in / out (KB/s) |
| **cpu** | `us` | % CPU time on user processes |
| **cpu** | `sy` | % CPU time on system (kernel) |
| **cpu** | `id` | % CPU idle |

---

## 2. Permissions & User Management

### 2.1 Understanding Permissions

Every file and directory in Linux has three permission sets and three permission types:

```
-  r w x   r w x   r w x
^  \_____/ \_____/ \_____/
|   owner   group   others
|
file type: - = file, d = directory, l = symlink
```

#### Permission Types

| Symbol | Octal | Meaning on File | Meaning on Directory |
|--------|-------|-----------------|----------------------|
| `r` | 4 | Read file contents | List directory contents |
| `w` | 2 | Modify / write to file | Create, delete files inside |
| `x` | 1 | Execute file as program | Enter (`cd`) the directory |
| `-` | 0 | No permission | No permission |

#### Common Permission Patterns

| Octal | Symbolic | Meaning |
|-------|----------|---------|
| `755` | `rwxr-xr-x` | Owner: full. Group & Others: read + execute. Typical for executables. |
| `644` | `rw-r--r--` | Owner: read + write. Group & Others: read only. Typical for files. |
| `700` | `rwx------` | Owner: full. No access for anyone else. |
| `600` | `rw-------` | Owner: read + write only. Private files (e.g. SSH keys). |
| `777` | `rwxrwxrwx` | Full access for everyone — avoid in production. |

> [!NOTE]
> **How to read octal — `755` example:**
> ```
> 7 = 4+2+1 = rwx  → owner can read, write, execute
> 5 = 4+0+1 = r-x  → group can read and execute (not write)
> 5 = 4+0+1 = r-x  → others can read and execute (not write)
> ```

---

### 2.2 `chmod` — Change File Permissions

```bash
# Octal (numeric) mode
chmod 755 script.sh             # rwxr-xr-x
chmod 644 file.txt              # rw-r--r--
chmod 700 private.sh            # rwx------
chmod -R 755 /var/www/          # apply recursively to a directory

# Symbolic mode
chmod u+x script.sh             # add execute for owner (u = user/owner)
chmod g-w file.txt              # remove write from group
chmod o-r secret.txt            # remove read from others
chmod a+r file.txt              # add read for all (a = all)
chmod u=rwx,g=rx,o=r file.txt   # set permissions explicitly
```

**Symbolic reference:**

| Who | Symbol |   | What | Symbol |
|-----|--------|---|------|--------|
| Owner | `u` | Add | `+` | Read | `r` |
| Group | `g` | Remove | `-` | Write | `w` |
| Others | `o` | Set exactly | `=` | Execute | `x` |
| All | `a` | | | | |

---

### 2.3 `chown` — Change Ownership

Changes the owner and/or group of a file or directory.

```bash
chown alice file.txt                # change owner to alice
chown alice:developers file.txt     # change owner to alice, group to developers
chown :developers file.txt          # change group only
chown -R alice:alice /home/alice/   # recursively change ownership
```

---

### 2.4 `umask` — Default Permission Mask

`umask` defines which permissions are *removed* by default when a new file or directory is created.

```bash
umask                   # show current umask (e.g. 0022)
umask 022               # set umask to 022 (for current session)
umask 027               # more restrictive: group gets no write, others get nothing
```

> [!NOTE]
> **How umask works:**
> ```
> Default max permissions:  777 (directories) / 666 (files)
> umask 022:               - 022
>                          = 755 (directories) / 644 (files)
> ```
> The umask *subtracts* from the default max — it does not add permissions.

---

### 2.5 User Management

#### `useradd` — Create a New User

```bash
useradd alice                           # create user alice (minimal setup)
useradd -m alice                        # create user with a home directory
useradd -m -s /bin/bash alice           # set bash as the default shell
useradd -m -G sudo,docker alice         # add to supplementary groups
useradd -m -c "Alice Smith" alice       # add a comment / full name
passwd alice                            # set a password for the user
```

#### `usermod` — Modify an Existing User

```bash
usermod -aG docker alice                # add alice to the docker group (-a = append)
usermod -s /bin/zsh alice               # change alice's default shell
usermod -l newname alice                # rename user alice to newname
usermod -L alice                        # lock alice's account (disable login)
usermod -U alice                        # unlock alice's account
usermod -d /new/home -m alice           # move alice's home directory
```

> [!TIP]
> Always use `-aG` (not just `-G`) when adding a user to a group — omitting `-a` **replaces** all existing groups.

#### `userdel` — Delete a User

```bash
userdel alice                   # delete user (keep home directory)
userdel -r alice                # delete user AND their home directory
```

---

### 2.6 Group Management

#### `groupadd` — Create a New Group

```bash
groupadd developers             # create a group called developers
groupadd -g 1050 developers     # create group with a specific GID
```

#### `groupdel` — Delete a Group

```bash
groupdel developers             # delete the group
```

#### Useful Commands for Inspecting Users & Groups

```bash
id alice                        # show alice's UID, GID, and group memberships
groups alice                    # list groups alice belongs to
cat /etc/passwd                 # list all users on the system
cat /etc/group                  # list all groups on the system
who                             # show who is currently logged in
```

---

## 3. System & Utility Commands

### 3.1 Environment Variables

#### `echo` — Print Text or Variables

```bash
echo "Hello, World"             # print a string
echo $HOME                      # print the value of an environment variable
echo $PATH                      # print the PATH variable
echo "User is: $USER"           # variable inside a string
echo -n "no newline"            # print without trailing newline
echo -e "line1\nline2"          # interpret escape sequences (\n, \t, etc.)
```

#### `env` — Display All Environment Variables

```bash
env                             # list all current environment variables
env | grep PATH                 # filter for a specific variable
env VAR=value command           # run a command with a temporary variable set
```

#### `printenv` — Print Specific Environment Variables

```bash
printenv                        # print all environment variables
printenv HOME                   # print the value of HOME
printenv PATH USER SHELL        # print multiple variables at once
```

> [!NOTE]
> **env vs printenv**
> - `env` — lists all variables AND can run commands with a modified environment
> - `printenv` — simpler, read-only, better for just checking variable values

#### `export` — Set Environment Variables

Makes a variable available to child processes and subshells.

```bash
export MY_VAR="hello"           # create and export a variable
export PATH=$PATH:/opt/myapp    # append to the existing PATH
export -p                       # list all currently exported variables
```

> [!TIP]
> Variables set with `export` in a terminal session are lost when the session ends.
> To make them permanent, add them to `~/.bashrc` or `~/.bash_profile`.

---

### 3.2 Archives & Compression

#### `tar` — Archive Files (Tape Archive)

`tar` bundles multiple files/directories into a single archive, optionally compressed.

```bash
# Create archives
tar -cvf archive.tar /path/         # create a .tar archive
tar -czvf archive.tar.gz /path/     # create a gzip-compressed archive
tar -cjvf archive.tar.bz2 /path/    # create a bzip2-compressed archive

# Extract archives
tar -xvf archive.tar                # extract a .tar archive
tar -xzvf archive.tar.gz            # extract a .tar.gz archive
tar -xjvf archive.tar.bz2           # extract a .tar.bz2 archive
tar -xvf archive.tar -C /opt/       # extract to a specific directory

# Inspect without extracting
tar -tvf archive.tar                # list contents of an archive
```

**Flag reference:**

| Flag | Meaning |
|------|---------|
| `c` | Create a new archive |
| `x` | Extract from archive |
| `t` | List contents |
| `v` | Verbose (show file names) |
| `f` | Specify the archive filename |
| `z` | Filter through gzip (`.tar.gz`) |
| `j` | Filter through bzip2 (`.tar.bz2`) |
| `C` | Change to directory before extracting |

#### `zip` / `unzip` — Zip Compression

```bash
zip archive.zip file1.txt file2.txt # zip specific files
zip -r archive.zip folder/          # zip a directory recursively
zip -e archive.zip file.txt         # create an encrypted zip (password prompt)

unzip archive.zip                   # extract to current directory
unzip archive.zip -d /opt/          # extract to a specific directory
unzip -l archive.zip                # list contents without extracting
unzip -o archive.zip                # overwrite existing files without asking
```

---

### 3.3 System Info & Monitoring

#### `date` — Display or Set the Date and Time

```bash
date                            # print current date and time
date +"%Y-%m-%d"                # print date in YYYY-MM-DD format
date +"%H:%M:%S"                # print time in HH:MM:SS format
date +"%d/%m/%Y %H:%M"          # custom format
date -d "2 days ago"            # print the date 2 days in the past
date -d "next Friday"           # print the date of next Friday
sudo date -s "2024-01-15 10:30" # set the system date and time
```

#### `whoami` — Print Current User

```bash
whoami                          # print the username of the current user
```

#### `uname` — Print System Information

```bash
uname                           # print the OS name (Linux)
uname -r                        # print the kernel release version
uname -a                        # print all system info (kernel, hostname, arch)
uname -m                        # print machine hardware architecture (x86_64)
uname -n                        # print the hostname
```

#### `history` — Command History

```bash
history                         # list all previously run commands
history 20                      # show the last 20 commands
history | grep ssh              # search history for a specific command
!42                             # re-run command number 42 from history
!!                              # re-run the last command
!ssh                            # re-run the most recent command starting with 'ssh'
history -c                      # clear all command history
```

> [!TIP]
> Press `Ctrl+R` in the terminal to interactively search through your command history.

#### `iostat` — CPU and I/O Statistics

Reports CPU utilization and disk I/O throughput. Useful for identifying disk bottlenecks.

```bash
iostat                          # single snapshot of CPU and disk stats
iostat 2                        # report every 2 seconds
iostat 2 5                      # report every 2 seconds, 5 times
iostat -x                       # extended disk stats (utilization %, await time)
iostat -d                       # disk stats only (no CPU)
iostat -c                       # CPU stats only (no disk)
```

**Key extended output columns (`-x`):**

| Column | Meaning |
|--------|---------|
| `%util` | % of time the disk was busy — near 100% = bottleneck |
| `await` | Average time (ms) for I/O requests to complete |
| `r/s` | Read requests per second |
| `w/s` | Write requests per second |
| `rkB/s` | Kilobytes read per second |
| `wkB/s` | Kilobytes written per second |

> [!NOTE]
> `iostat` is part of the `sysstat` package. Install with:
> ```bash
> sudo apt install sysstat     # Debian/Ubuntu
> sudo dnf install sysstat     # RHEL/CentOS
> ```

---

### 3.4 Scheduling with Cron / Crontab

#### What is a Cron Job?

A **cron job** is a scheduled task that Linux runs automatically at a defined time or interval. The **cron daemon** (`crond`) runs in the background and checks the **crontab** (cron table) file for scheduled tasks.

Common uses: automated backups, log rotation, sending reports, running scripts on a schedule.

#### Crontab Syntax

```
* * * * * /path/to/command
│ │ │ │ │
│ │ │ │ └── Day of week  (0–7, 0 and 7 = Sunday)
│ │ │ └──── Month        (1–12)
│ │ └────── Day of month (1–31)
│ └──────── Hour         (0–23)
└────────── Minute       (0–59)
```

#### Special Values

| Value | Meaning |
|-------|---------|
| `*` | Every (any value) |
| `*/n` | Every n units (e.g. `*/5` = every 5 minutes) |
| `n,m` | At n and m (e.g. `1,15` = 1st and 15th) |
| `n-m` | Range from n to m (e.g. `9-17` = 9am to 5pm) |

#### Crontab Examples

```bash
# ┌──── min  ┌──── hour  ┌──── day  ┌──── month  ┌──── weekday
# │          │           │          │             │
  0  2  *  *  *   /usr/bin/backup.sh          # daily at 2:00 AM
  0  9  *  *  1   /scripts/weekly_report.sh   # every Monday at 9:00 AM
  */5 *  *  *  *  /scripts/health_check.sh   # every 5 minutes
  0  0  1  *  *   /scripts/monthly.sh        # 1st of every month at midnight
  30 18  *  *  1-5 /scripts/end_of_day.sh    # weekdays at 6:30 PM
```

#### Crontab Commands

```bash
crontab -e              # open and edit your crontab (creates one if missing)
crontab -l              # list your current cron jobs
crontab -r              # remove (delete) all your cron jobs
sudo crontab -u alice -e # edit alice's crontab (as root)
```

#### Special Shorthand Strings

Instead of the 5 fields, you can use these shortcuts:

```bash
@reboot     /scripts/startup.sh    # run once at system boot
@daily      /scripts/backup.sh     # run once a day (midnight)
@weekly     /scripts/cleanup.sh    # run once a week (Sunday midnight)
@monthly    /scripts/report.sh     # run once a month
@hourly     /scripts/check.sh      # run every hour
```

> [!TIP]
> **How to add a cron job step by step:**
> 1. Run `crontab -e` — this opens your crontab in the default editor
> 2. Add a line with the schedule and command, e.g.:
>    ```
>    0 3 * * * /home/alice/backup.sh >> /var/log/backup.log 2>&1
>    ```
> 3. Save and exit — cron picks it up automatically, no restart needed
>
> The `>> /var/log/backup.log 2>&1` part redirects both stdout and errors to a log file — always useful for debugging.

---

## 4. Process & Resource Management

### 4.1 `top` / `htop` — Interactive Process Viewers

Both give a **live, real-time dashboard** of running processes, CPU, and memory usage. They refresh automatically every few seconds.

#### `top` — Built-in Process Monitor

```bash
top                         # launch the live process viewer
top -u alice                # show only processes owned by alice
top -p 1234                 # monitor a specific PID only
top -n 5                    # auto-exit after 5 refresh cycles (useful in scripts)
top -b -n 1 > snapshot.txt  # batch mode: capture one snapshot to a file
```

**Key interactive shortcuts inside `top`:**

| Key | Action |
|-----|--------|
| `k` | Kill a process (prompts for PID) |
| `r` | Renice a process (change priority) |
| `M` | Sort by memory usage |
| `P` | Sort by CPU usage (default) |
| `u` | Filter by username |
| `q` | Quit |
| `1` | Toggle per-CPU core breakdown |
| `h` | Help |

**Understanding the `top` header:**

```
top - 10:32:01 up 3 days,  2:14,  2 users,  load average: 0.12, 0.18, 0.20
Tasks: 210 total,   1 running, 209 sleeping
%Cpu(s):  2.1 us,  0.5 sy,  0.0 ni, 97.0 id,  0.3 wa
MiB Mem :  7849.0 total,   512.3 free,  4201.1 used,  3135.6 buff/cache
MiB Swap:  2048.0 total,  1980.2 free,    67.8 used.  3647.9 avail Mem
```

| Field | Meaning |
|-------|---------|
| `load average` | CPU load over 1, 5, 15 minutes. Values > number of CPU cores = overloaded |
| `us` | % CPU on user processes |
| `sy` | % CPU on kernel/system processes |
| `id` | % CPU idle (higher = healthier) |
| `wa` | % CPU waiting for I/O (high = disk bottleneck) |
| `buff/cache` | Memory used by kernel buffers — can be reclaimed if needed |

#### `htop` — Enhanced Interactive Viewer *(install separately)*

`htop` is a more user-friendly, colorful alternative to `top` with mouse support and easier process management.

```bash
htop                        # launch htop
htop -u alice               # show only alice's processes
htop -p 1234,5678           # monitor specific PIDs
htop -d 10                  # set refresh delay to 10 tenths of a second (1s)
```

**Why prefer `htop` over `top`:**
- Color-coded CPU/memory bars per core
- Mouse-clickable interface
- Easy sorting by clicking column headers
- Kill, renice, and search without remembering keyboard shortcuts
- Displays full command paths

> [!TIP]
> Install `htop` with:
> ```bash
> sudo apt install htop      # Debian/Ubuntu
> sudo dnf install htop      # RHEL/CentOS
> ```

> [!NOTE]
> **top vs htop**
> | | `top` | `htop` |
> |---|---|---|
> | Built-in | ✅ Always available | ❌ Must install |
> | Mouse support | ❌ | ✅ |
> | Per-core CPU bars | Toggle with `1` | Always visible |
> | Best for | Quick checks on any server | Daily monitoring on your own machine |

---

### 4.2 `ps` — Process Snapshot

While `top`/`htop` show live data, `ps` takes a **static snapshot** of currently running processes at the moment you run it. Essential for scripting and piping into `grep`.

```bash
ps                          # show processes in the current terminal session only
ps aux                      # show ALL processes from ALL users (most common)
ps -ef                      # same as aux but in standard (POSIX) format
ps -u alice                 # show all processes owned by alice
ps -p 1234                  # show info for a specific PID
ps --forest                 # show process tree (parent → child hierarchy)
ps aux | grep nginx         # find if a specific process is running
ps aux --sort=-%mem         # sort all processes by memory usage (highest first)
ps aux --sort=-%cpu         # sort all processes by CPU usage (highest first)
```

**`ps aux` vs `ps -ef` — what's the difference?**

| | `ps aux` | `ps -ef` |
|---|---|---|
| Style | BSD syntax | POSIX/System V syntax |
| Shows full command | ✅ | ✅ |
| Shows `%CPU`, `%MEM` | ✅ | ❌ |
| Shows parent PID (PPID) | ❌ | ✅ |
| Most common on | Linux | Linux & Unix |

**Key columns in `ps aux` output:**

| Column | Meaning |
|--------|---------|
| `USER` | Owner of the process |
| `PID` | Process ID — unique identifier |
| `%CPU` | CPU usage percentage |
| `%MEM` | RAM usage percentage |
| `VSZ` | Virtual memory size (KB) |
| `RSS` | Resident Set Size — actual RAM used (KB) |
| `STAT` | Process state (S=sleeping, R=running, Z=zombie, T=stopped) |
| `START` | When the process started |
| `COMMAND` | The command that launched the process |

> [!NOTE]
> **Process States (`STAT` column):**
> | Code | Meaning |
> |------|---------|
> | `R` | Running or runnable (on the CPU) |
> | `S` | Sleeping (waiting for an event) |
> | `D` | Uninterruptible sleep (usually waiting for I/O) |
> | `Z` | Zombie — process finished but not yet reaped by parent |
> | `T` | Stopped (e.g. by Ctrl+Z) |
> | `<` | High priority (negative nice value) |
> | `N` | Low priority (positive nice value) |

---

### 4.3 `kill` — Terminate Processes

`kill` sends a **signal** to a process. By default it sends `SIGTERM` (graceful shutdown request). You target processes by their **PID**, which you get from `ps` or `top`.

```bash
kill 1234                   # send SIGTERM (15) to PID 1234 — asks it to stop gracefully
kill -9 1234                # send SIGKILL — force-kill immediately, no cleanup
kill -15 1234               # explicitly send SIGTERM (same as default)
kill -1 1234                # send SIGHUP — reload config (used for daemons)
kill -STOP 1234             # pause/suspend a process
kill -CONT 1234             # resume a paused process
kill -l                     # list all available signal names and numbers
```

**Killing by name instead of PID:**

```bash
killall nginx               # kill all processes named "nginx" (by name)
killall -9 nginx            # force-kill all nginx processes
pkill nginx                 # similar to killall, supports pattern matching
pkill -u alice              # kill all processes owned by alice
pkill -9 -u alice           # force-kill all of alice's processes
```

**Common signals:**

| Signal | Number | Meaning |
|--------|--------|---------|
| `SIGHUP` | 1 | Hangup — reload config or restart daemon |
| `SIGINT` | 2 | Interrupt — same as pressing `Ctrl+C` |
| `SIGKILL` | 9 | Force kill — **cannot be caught or ignored** |
| `SIGTERM` | 15 | Graceful termination request (default) |
| `SIGSTOP` | 19 | Pause process (cannot be caught or ignored) |
| `SIGCONT` | 18 | Resume a paused process |

> [!TIP]
> **Always try `SIGTERM` (default) first.** Only use `kill -9` as a last resort — force-killing skips cleanup, which can leave corrupted files, locked resources, or orphaned child processes.

> [!WARNING]
> `kill -9` cannot be caught, blocked, or ignored by the process. The kernel forcibly removes it from memory immediately. Use it only when the process is completely unresponsive.

---

### 4.4 `free` — Memory Usage

Displays the total, used, and available **RAM and swap** space.

```bash
free                        # show memory in kilobytes (default)
free -m                     # show in megabytes (MB) — easier to read
free -h                     # show in human-readable units (GB/MB auto)
free -s 2                   # refresh every 2 seconds continuously
free -t                     # show a total row at the bottom
free -h --si                # use powers of 1000 (SI units) instead of 1024
```

**Example output (`free -h`):**

```
              total        used        free      shared  buff/cache   available
Mem:           7.7G        4.1G        512M        203M        3.1G        3.6G
Swap:          2.0G         66M        1.9G
```

**Understanding the columns:**

| Column | Meaning |
|--------|---------|
| `total` | Total installed RAM |
| `used` | RAM actively used by processes |
| `free` | RAM completely unused |
| `shared` | RAM shared between processes (tmpfs, etc.) |
| `buff/cache` | RAM used by kernel for disk caching — **can be freed if needed** |
| `available` | RAM available for new processes (free + reclaimable cache) — **use this, not `free`** |

> [!NOTE]
> **Don't panic if `free` is low.** Linux deliberately uses spare RAM for disk caching (`buff/cache`) to speed up I/O. This memory is released immediately when a process needs it. Always look at `available`, not `free`.

---

### 4.5 `df` — Disk Space Usage

Shows **disk space usage** for all mounted filesystems. `df` = **d**isk **f**ree.

```bash
df                          # show all filesystems in 1KB blocks
df -h                       # human-readable sizes (GB/MB)
df -H                       # same but using powers of 1000 (SI)
df -T                       # include filesystem type (ext4, tmpfs, etc.)
df -i                       # show inode usage instead of block usage
df /home                    # show disk info for the filesystem containing /home
df -h --total               # add a grand total row
```

**Example output (`df -h`):**

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   22G   26G  46% /
/dev/sdb1       200G  110G   82G  56% /data
tmpfs           3.9G  204K  3.9G   1% /run
```

> [!TIP]
> Watch out for **inode exhaustion** (`df -i`) — a disk can show free space but still fail to create new files if inodes are 100% used. This is common on filesystems with millions of tiny files.

---

### 4.6 `du` — Directory Disk Usage

Shows how much disk space a **specific file or directory** is using. `du` = **d**isk **u**sage.

```bash
du /var/log                 # show size of each subdirectory under /var/log
du -h /var/log              # human-readable sizes
du -sh /var/log             # summary only — single total for the whole directory
du -sh *                    # summary for each item in the current directory
du -ah /var/log             # show size of every file AND directory
du --max-depth=1 /var        # show only 1 level deep (direct children)
du -sh * | sort -h          # list items sorted by size (smallest to largest)
```

> [!NOTE]
> **df vs du**
> | | `df` | `du` |
> |---|---|---|
> | Scope | Entire filesystem / partition | Specific directory or file |
> | Best for | "Is my disk almost full?" | "What's eating up all my space?" |
> | Speed | Instant | Can be slow on large directories |

---

### 4.7 `uptime` — System Uptime & Load

Shows how long the system has been running, number of logged-in users, and **CPU load averages**.

```bash
uptime                      # show uptime and load averages
uptime -p                   # pretty format: "up 3 days, 2 hours, 14 minutes"
uptime -s                   # show the exact date/time the system was booted
```

**Example output:**

```
10:45:01 up 3 days, 2:17,  2 users,  load average: 0.15, 0.22, 0.18
```

| Field | Meaning |
|-------|---------|
| `up 3 days, 2:17` | System has been running for 3 days and 2 hours 17 minutes |
| `2 users` | Number of currently logged-in users |
| `load average: 0.15, 0.22, 0.18` | Average number of processes waiting for CPU over 1, 5, 15 minutes |

> [!NOTE]
> **How to interpret load average:**
> - Load average represents the average number of processes **waiting to run** or **waiting for I/O**.
> - A healthy load average is **≤ number of CPU cores**.
> - On a 4-core system: load of `4.0` = fully loaded, `8.0` = severely overloaded.
> - Check core count with: `nproc` or `grep -c ^processor /proc/cpuinfo`

---

### 4.8 `nice` / `renice` — Process Priority

Linux schedules CPU time among processes using a **priority value** called **niceness**. Lower nice = higher priority. Higher nice = more "polite" to other processes = lower priority.

```
Nice range: -20 (highest priority) to +19 (lowest priority)
Default:     0
```

#### `nice` — Launch a Process with a Set Priority

```bash
nice command                    # run command with default nice value (+10)
nice -n 10 ./backup.sh          # run backup script at low priority (nice=10)
nice -n -5 ./critical.sh        # run at higher priority (nice=-5, needs root)
sudo nice -n -20 ./urgent.sh    # run at maximum priority (root required for negative)
```

#### `renice` — Change Priority of a Running Process

```bash
renice 10 -p 1234               # set PID 1234 to nice value 10 (lower priority)
renice -5 -p 1234               # set to higher priority (requires root)
renice 15 -u alice              # lower priority of ALL of alice's processes
sudo renice -10 -p 1234         # requires sudo for negative (higher priority) values
```

> [!NOTE]
> **Niceness quick reference:**
> | Nice Value | Priority | Use Case |
> |------------|----------|----------|
> | `-20` | Highest | Real-time critical tasks (root only) |
> | `-5` to `-1` | High | Important background services (root only) |
> | `0` | Default | Normal processes |
> | `1` to `10` | Low | Background jobs that shouldn't disturb others |
> | `11` to `19` | Lowest | Long batch jobs, backups, compilations |

> [!TIP]
> **Rule of thumb:** Run long-running background tasks (backups, large file copies, video encoding) with `nice -n 19` so they don't steal CPU from interactive work. Regular users can only *increase* the nice value (lower priority). Only root can *decrease* it (raise priority).

---

> [!NOTE]
> **Quick Reference — Chapter 2 Commands**
>
> | Category | Commands |
> |----------|----------|
> | **Networking** | `ping`, `curl`, `wget`, `ss`, `netstat`, `ip`, `hostname`, `nslookup`, `dig`, `vmstat` |
> | **Permissions** | `chmod`, `chown`, `umask` |
> | **Users & Groups** | `useradd`, `usermod`, `userdel`, `groupadd`, `groupdel`, `id`, `groups` |
> | **Environment** | `echo`, `env`, `printenv`, `export` |
> | **Archives** | `tar`, `zip`, `unzip` |
> | **System Info** | `date`, `whoami`, `uname`, `history`, `iostat` |
> | **Scheduling** | `crontab` |
> | **Process Viewing** | `top`, `htop`, `ps`, `ps aux`, `ps -ef`, `ps -u` |
> | **Process Control** | `kill`, `kill -9`, `killall`, `pkill` |
> | **Resource Usage** | `free -m`, `free -h`, `df -h`, `du -sh`, `uptime` |
> | **Process Priority** | `nice`, `renice` |
