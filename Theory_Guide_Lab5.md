# Theory Guide - Lab 5: Process Management

## Table of Contents
1. [Introduction to Processes](#introduction-to-processes)
2. [Process States](#process-states)
3. [Job Control](#job-control)
4. [Process Information](#process-information)
5. [Signals](#signals)
6. [Process Priority](#process-priority)
7. [System Resources](#system-resources)
8. [Command Reference](#command-reference)

---

## Introduction to Processes

### What is a Process?

A **process** is a running instance of a program.

**Key characteristics:**
- Has unique **Process ID (PID)**
- Owns system resources (memory, CPU time, file handles)
- Has a **state** (running, stopped, sleeping, etc.)
- Has a **parent process** (PPID)
- Can spawn **child processes**

### Process vs Program

| Program | Process |
|---------|---------|
| Executable file on disk | Running instance in memory |
| Static code | Dynamic execution |
| One program | Multiple processes |
| `/usr/bin/bash` | PID 1234, PID 5678 |

---

## Process States

### Common Process States

| State | Symbol | Description |
|-------|--------|-------------|
| **Running** | R | Actively executing on CPU |
| **Sleeping** | S | Waiting for event (interruptible) |
| **Stopped** | T | Suspended (Ctrl+Z) |
| **Zombie** | Z | Terminated, waiting for parent cleanup |
| **Uninterruptible Sleep** | D | Waiting for I/O (can't be interrupted) |

### Process Lifecycle

```
Created → Running ⇄ Sleeping
            ↓
         Stopped (T)
            ↓
         Terminated → Zombie → Reaped
```

**Transitions:**
- **Running** ↔ **Sleeping:** Normal execution cycle
- **Running** → **Stopped:** Signal (SIGSTOP, Ctrl+Z)
- **Stopped** → **Running:** Signal (SIGCONT, fg, bg)
- **Running** → **Zombie:** Process exits, waits for parent
- **Zombie** → **Reaped:** Parent collects exit status

---

## Job Control

### Background vs Foreground

**Foreground process:**
- Occupies the terminal
- Receives keyboard input
- Prevents new commands until finished

**Background process:**
- Runs without occupying terminal
- Cannot receive keyboard input
- Terminal remains free for other commands

### Starting Processes

**Foreground (default):**
```bash
command
```

**Background (append `&`):**
```bash
command &
```

**Example:**
```bash
# Foreground - terminal blocked
sleep 100

# Background - terminal free
sleep 100 &
```

### The `jobs` Command

**Purpose:** List jobs in current shell session.

**Syntax:**
```bash
jobs [OPTIONS]
```

**Common options:**
| Option | Description |
|--------|-------------|
| `-l` | Include PID |
| `-p` | PID only |
| `-r` | Running jobs only |
| `-s` | Stopped jobs only |

**Output format:**
```bash
[1]+ Running    sleep 100 &
[2]- Stopped    vim file.txt
```

**Explanation:**
- `[1]`, `[2]` - Job numbers
- `+` - Current job (default for fg/bg)
- `-` - Previous job
- **Running/Stopped** - Job state
- Command and arguments

### The `fg` Command

**Purpose:** Bring background/stopped job to foreground.

**Syntax:**
```bash
fg [%JOB]
```

**Usage:**
```bash
fg          # Bring most recent job to foreground
fg %1       # Bring job 1 to foreground
fg %2       # Bring job 2 to foreground
```

**Example:**
```bash
sleep 100 &
# [1] 12345
jobs
# [1]+ Running    sleep 100 &
fg %1
# (sleep command now in foreground)
```

### The `bg` Command

**Purpose:** Resume stopped job in background.

**Syntax:**
```bash
bg [%JOB]
```

**Usage:**
```bash
bg          # Resume most recent stopped job
bg %1       # Resume job 1 in background
```

**Typical workflow:**
```bash
vim file.txt         # Start vim
Ctrl+Z               # Suspend vim
# [1]+ Stopped    vim file.txt
bg                   # Try to resume in background (won't work for vim)
fg                   # Bring vim back to foreground
```

### Key Combinations

| Key | Signal | Effect |
|-----|--------|--------|
| **Ctrl+C** | SIGINT | Terminate process |
| **Ctrl+Z** | SIGTSTP | Suspend (stop) process |
| **Ctrl+D** | EOF | End of input (not a signal) |
| **Ctrl+\\** | SIGQUIT | Quit with core dump |

**Ctrl+Z Example:**
```bash
vim file.txt         # Start editing
Ctrl+Z               # Suspend vim
# [1]+ Stopped    vim file.txt
ls                   # Run other commands
fg                   # Resume vim
```

### Job Specifications

**Referring to jobs:**

| Spec | Meaning |
|------|---------|
| `%1` | Job 1 |
| `%2` | Job 2 |
| `%%` | Current job (same as `%+`) |
| `%+` | Current job |
| `%-` | Previous job |
| `%str` | Job starting with "str" |
| `%?str` | Job containing "str" |

**Examples:**
```bash
fg %1           # Job 1 to foreground
bg %2           # Resume job 2 in background
kill %vim       # Kill job starting with "vim"
```

---

## Process Information

### The `ps` Command

**Purpose:** Display process status snapshot.

**Basic syntax:**
```bash
ps [OPTIONS]
```

**Common option sets:**

**BSD style (no dash):**
```bash
ps aux          # All processes, detailed
ps aux | grep pattern
```

**Unix style (with dash):**
```bash
ps -ef          # All processes, full format
ps -u username  # Processes by user
```

**Essential columns:**

| Column | Meaning |
|--------|---------|
| **USER** | Process owner |
| **PID** | Process ID |
| **%CPU** | CPU usage percentage |
| **%MEM** | Memory usage percentage |
| **VSZ** | Virtual memory size (KB) |
| **RSS** | Resident set size (physical RAM, KB) |
| **TTY** | Controlling terminal |
| **STAT** | Process state |
| **START** | Start time |
| **TIME** | CPU time consumed |
| **COMMAND** | Command with arguments |

**Common ps commands:**

```bash
# All processes
ps aux

# My processes
ps -u $USER

# Processes without terminal (daemons)
ps aux | grep "?"

# Tree view (process hierarchy)
ps auxf        # BSD
ps -ejH        # Unix

# Specific process
ps -p 1234

# Watch processes (updates)
watch -n 1 'ps aux | head -20'
```

### The `pstree` Command

**Purpose:** Display process tree (hierarchy).

**Syntax:**
```bash
pstree [OPTIONS] [PID|USER]
```

**Common options:**
| Option | Description |
|--------|-------------|
| `-p` | Show PIDs |
| `-a` | Show command line arguments |
| `-u` | Show user changes |
| `-h` | Highlight current process |

**Example output:**
```
systemd─┬─sshd───sshd───bash───vim
        ├─cron
        └─apache2─┬─apache2
                  ├─apache2
                  └─apache2
```

### The `top` Command

**Purpose:** Real-time process monitoring.

**Basic usage:**
```bash
top
```

**Interactive commands (while top is running):**

| Key | Action |
|-----|--------|
| `q` | Quit |
| `h` | Help |
| `k` | Kill process (prompts for PID) |
| `r` | Renice (change priority) |
| `M` | Sort by memory |
| `P` | Sort by CPU |
| `u` | Filter by user |
| `1` | Show all CPUs |
| `c` | Show full command |

**Command line options:**
```bash
top -u username     # Show only user's processes
top -p PID          # Monitor specific process
top -b -n 1         # Batch mode, one iteration
```

**Understanding top output:**

**Header:**
- **Load average:** System load (1, 5, 15 minute averages)
- **Tasks:** Total, running, sleeping, stopped, zombie
- **%Cpu(s):** CPU usage breakdown
- **MiB Mem:** Memory usage
- **MiB Swap:** Swap usage

**Load average interpretation:**
- < 1.0 per CPU: System fine
- = 1.0 per CPU: System at capacity
- \> 1.0 per CPU: System overloaded

### The `htop` Command

**Purpose:** Enhanced, user-friendly version of top.

**Features:**
- Color-coded display
- Mouse support
- Tree view
- Easy filtering and sorting
- Visual CPU/memory bars

**Usage:**
```bash
htop
```

**Advantages over top:**
- More intuitive interface
- Easier process management
- Better visualization
- Horizontal/vertical scrolling

**Note:** May need installation: `sudo apt install htop`

### The `pidof` Command

**Purpose:** Find PID of running program.

**Syntax:**
```bash
pidof program_name
```

**Examples:**
```bash
pidof bash
# Output: 1234 5678 9012

pidof sshd
# Output: 987

pidof nonexistent
# No output (exit code 1)
```

**Use in scripts:**
```bash
if pidof apache2 > /dev/null; then
    echo "Apache is running"
fi
```

### The `uptime` Command

**Purpose:** Show system uptime and load average.

**Syntax:**
```bash
uptime
```

**Example output:**
```
14:23:45 up 10 days, 3:42, 5 users, load average: 0.15, 0.18, 0.20
```

**Interpretation:**
- **Current time:** 14:23:45
- **Uptime:** 10 days, 3 hours, 42 minutes
- **Users:** 5 logged in
- **Load average:** 1min, 5min, 15min

---

## Signals

### What are Signals?

**Signals** are software interrupts that notify processes of events.

**Purpose:**
- Terminate processes
- Suspend/resume execution
- Force process to respond
- Communicate between processes

### Common Signals

| Signal | Number | Description | Default Action |
|--------|--------|-------------|----------------|
| **SIGHUP** | 1 | Hangup (terminal closed) | Terminate |
| **SIGINT** | 2 | Interrupt (Ctrl+C) | Terminate |
| **SIGQUIT** | 3 | Quit (Ctrl+\\) | Terminate + core dump |
| **SIGKILL** | 9 | Kill (cannot be caught) | Terminate immediately |
| **SIGTERM** | 15 | Terminate (default kill) | Terminate gracefully |
| **SIGSTOP** | 19 | Stop (cannot be caught) | Suspend |
| **SIGTSTP** | 20 | Terminal stop (Ctrl+Z) | Suspend |
| **SIGCONT** | 18 | Continue | Resume |

### The `kill` Command

**Purpose:** Send signals to processes.

**Common misconception:** "kill" doesn't always terminate - it sends signals.

**Syntax:**
```bash
kill [OPTIONS] PID
kill -SIGNAL PID
kill -s SIGNAL PID
```

**Examples:**

**Terminate gracefully (default):**
```bash
kill 1234              # Sends SIGTERM (15)
```

**Terminate forcefully:**
```bash
kill -9 1234           # Sends SIGKILL
kill -KILL 1234        # Same
kill -SIGKILL 1234     # Same
```

**Stop process:**
```bash
kill -STOP 1234        # Suspend
```

**Continue process:**
```bash
kill -CONT 1234        # Resume
```

**List signals:**
```bash
kill -l                # List all signals
```

### The `killall` Command

**Purpose:** Kill processes by name.

**Syntax:**
```bash
killall [OPTIONS] process_name
```

**Examples:**
```bash
killall firefox        # Kill all firefox processes
killall -9 python      # Force kill all python processes
killall -STOP prog     # Stop all instances of prog
killall -CONT prog     # Resume all instances
```

**⚠️ Warning:** Affects ALL processes with that name!

### Signal Handling

**Catchable signals** (process can handle):
- SIGTERM (15)
- SIGINT (2)
- SIGUSR1 (10)
- SIGUSR2 (12)

**Uncatchable signals** (cannot be blocked/handled):
- SIGKILL (9)
- SIGSTOP (19)

**Why SIGKILL vs SIGTERM?**

**SIGTERM (15):**
- **Graceful shutdown**
- Process can clean up (save files, close connections)
- Process can refuse (if programmed to)
- **Default kill signal**

**SIGKILL (9):**
- **Immediate termination**
- No cleanup possible
- Cannot be blocked
- **Last resort only**

**Best practice:**
```bash
# Try graceful first
kill PID
sleep 2

# If still running, force
kill -9 PID
```

---

## Process Priority

### Understanding Priority

**Priority** determines CPU scheduling preference.

**Nice values:**
- Range: **-20** (highest priority) to **+19** (lowest priority)
- Default: **0**
- **Lower nice value = higher priority**

**Who can set what:**
- Regular users: Can only increase nice (lower priority)
- Root: Can set any nice value

### The `nice` Command

**Purpose:** Start process with modified priority.

**Syntax:**
```bash
nice [OPTIONS] COMMAND
```

**Examples:**
```bash
# Default nice (10)
nice command

# Specific nice value
nice -n 15 command      # Lower priority
nice -n -10 command     # Higher priority (requires root)

# Background with low priority
nice -n 19 long_task &
```

**Practical uses:**
```bash
# CPU-intensive task, low priority
nice -n 19 ffmpeg -i input.mp4 output.avi

# Compile with normal priority
nice -n 0 make
```

### The `renice` Command

**Purpose:** Change priority of running process.

**Syntax:**
```bash
renice PRIORITY [OPTIONS] PID
```

**Examples:**
```bash
# Set PID 1234 to nice 10
renice 10 1234

# Set to highest priority (root only)
renice -20 1234

# By user
renice 15 -u username

# By process group
renice 10 -g 5678
```

**Verification:**
```bash
ps -o pid,ni,comm -p 1234
# Shows PID, nice value, command
```

---

## System Resources

### The `free` Command

**Purpose:** Display memory usage.

**Syntax:**
```bash
free [OPTIONS]
```

**Common options:**
| Option | Description |
|--------|-------------|
| `-h` | Human-readable (GB, MB) |
| `-m` | Show in megabytes |
| `-g` | Show in gigabytes |
| `-s N` | Update every N seconds |

**Example output:**
```bash
free -m
              total    used    free  shared  buff/cache  available
Mem:           7982    3421    1234     567        3327       3876
Swap:          2047       0    2047
```

**Understanding columns:**
- **total:** Total installed RAM
- **used:** Used by processes
- **free:** Completely unused
- **shared:** Shared memory
- **buff/cache:** Buffers and cache
- **available:** Available for new processes

**Important:** Don't panic if "free" is low - Linux uses free RAM for caching.

**Continuous monitoring:**
```bash
watch -n 1 free -m     # Update every second
free -s 2              # Update every 2 seconds
```

### Memory Types

**Physical Memory (RAM):**
- **RSS (Resident Set Size):** Actual physical memory used
- **Fast** but limited

**Virtual Memory (VSZ):**
- **Total memory** including swap
- Includes memory mapped to disk

**Swap:**
- **Disk space** used as extra RAM
- **Slow** but provides buffer

### Process Resource Usage

**Check specific process:**
```bash
ps -o pid,rss,vsz,comm -p PID

# Example output:
#  PID   RSS   VSZ COMMAND
# 1234 12345 23456 firefox
```

**Top consumers:**
```bash
# By memory
ps aux --sort=-%mem | head -10

# By CPU
ps aux --sort=-%cpu | head -10
```

---

## Command Reference

### Job Control

```bash
command &           # Start in background
jobs                # List jobs
jobs -l             # List with PIDs
fg                  # Foreground most recent
fg %1               # Foreground job 1
bg                  # Background most recent
bg %1               # Background job 1
Ctrl+Z              # Suspend foreground
Ctrl+C              # Terminate foreground
```

### Process Information

```bash
ps aux              # All processes
ps -u user          # User's processes
ps -p PID           # Specific process
pstree              # Process tree
top                 # Real-time monitor
htop                # Enhanced monitor
pidof program       # Find PID
uptime              # System uptime
```

### Signals

```bash
kill PID            # Terminate (SIGTERM)
kill -9 PID         # Force kill (SIGKILL)
kill -STOP PID      # Suspend
kill -CONT PID      # Resume
killall prog        # Kill by name
kill -l             # List signals
```

### Priority

```bash
nice -n 10 cmd      # Start with priority
renice 10 PID       # Change priority
```

### Resources

```bash
free -m             # Memory usage (MB)
free -h             # Memory usage (human)
```

---

## Best Practices

### 1. Graceful Termination

```bash
# Good: Try graceful first
kill PID
sleep 2
# If needed:
kill -9 PID

# Bad: Immediate force
kill -9 PID
```

### 2. Background Long Tasks

```bash
# Good: Free terminal
long_task &

# Bad: Block terminal
long_task
```

### 3. Monitor Resource Usage

```bash
# Check before and during intensive tasks
top
htop
free -m
```

### 4. Use Appropriate Signals

| Scenario | Signal |
|----------|--------|
| Normal termination | SIGTERM (15) |
| Frozen process | SIGKILL (9) |
| Suspend temporarily | SIGSTOP (19) |
| Resume suspended | SIGCONT (18) |

### 5. Priority for Batch Jobs

```bash
# CPU-intensive, non-urgent
nice -n 19 batch_process &
```

---

## Common Scenarios

### Scenario 1: Process Won't Stop

```bash
# Try graceful
kill PID

# Wait a moment
sleep 3

# Check if still running
ps -p PID

# Force if necessary
kill -9 PID
```

### Scenario 2: Finding Resource Hogs

```bash
# CPU hog
ps aux --sort=-%cpu | head -5

# Memory hog
ps aux --sort=-%mem | head -5

# In top: Press M (memory) or P (CPU)
```

### Scenario 3: Background Task Management

```bash
# Start task
long_process &

# Check status
jobs

# Bring to foreground to see output
fg

# Suspend and resume in background
Ctrl+Z
bg
```

### Scenario 4: Terminal Disconnection

```bash
# Problem: Process dies when terminal closes

# Solution 1: nohup (no hangup)
nohup command &

# Solution 2: disown
command &
disown

# Solution 3: tmux/screen
tmux
command
# Detach: Ctrl+b, d
```

---

## Summary

Lab 5 covers essential process management:

**Job Control:**
- `jobs` - List jobs
- `fg` - Foreground job
- `bg` - Background job
- `Ctrl+Z` - Suspend
- `Ctrl+C` - Terminate

**Process Info:**
- `ps` - Process status
- `top`/`htop` - Monitor
- `pstree` - Hierarchy
- `pidof` - Find PID
- `uptime` - System stats

**Signals:**
- `kill` - Send signal to PID
- `killall` - Send signal to name
- Common signals: SIGTERM (15), SIGKILL (9), SIGSTOP, SIGCONT

**Priority:**
- `nice` - Start with priority
- `renice` - Change priority
- Range: -20 (high) to +19 (low)

**Resources:**
- `free` - Memory usage
- RSS vs VSZ

**Key Principle:** Processes are manageable resources - control their lifecycle, monitor their usage, and manage their priority.
