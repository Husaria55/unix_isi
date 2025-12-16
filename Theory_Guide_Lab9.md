# Theory Guide - Lab 9: Compression, Archives, Scheduling, and Advanced Tools

## Table of Contents
1. [File Compression](#file-compression)
2. [Archives with tar](#archives-with-tar)
3. [Resource Limits](#resource-limits)
4. [Terminal Multiplexing](#terminal-multiplexing)
5. [Scheduled Tasks](#scheduled-tasks)
6. [Command Reference](#command-reference)

---

## File Compression

### gzip/gunzip

**Purpose:** Compress/decompress files using Lempel-Ziv coding.

**Compress file:**
```bash
gzip file.txt
# Creates: file.txt.gz
# Removes: file.txt
```

**Decompress:**
```bash
gunzip file.txt.gz
# Creates: file.txt
# Removes: file.txt.gz
```

**Keep original:**
```bash
gzip -c file.txt > file.txt.gz
gunzip -c file.txt.gz > file.txt
```

**Compression levels:**
```bash
gzip -1 file.txt  # Fastest (less compression)
gzip -9 file.txt  # Best (slower)
gzip -6 file.txt  # Default
```

**View compressed file:**
```bash
zcat file.txt.gz        # View contents
zgrep pattern file.gz   # Search in compressed
zless file.gz           # Page through compressed
```

---

## Archives with tar

### What is tar?

**tar** - Tape Archive - bundles multiple files into one archive.

**Common operations:**
- Create archive
- Extract archive
- List contents
- Append files

### Creating Archives

**Basic archive:**
```bash
tar -cf archive.tar file1 file2 dir/
```

**Options:**
- `c` - Create
- `f` - File (specify archive name)

**With compression:**
```bash
tar -czf archive.tar.gz files/   # gzip compression
tar -cjf archive.tar.bz2 files/  # bzip2 compression
tar -cJf archive.tar.xz files/   # xz compression
```

**Options:**
- `z` - gzip
- `j` - bzip2
- `J` - xz

**Verbose output:**
```bash
tar -cvzf archive.tar.gz files/
```
- `v` - Verbose (show files being archived)

### Extracting Archives

**Extract:**
```bash
tar -xf archive.tar
```

**Extract compressed:**
```bash
tar -xzf archive.tar.gz   # gzip
tar -xjf archive.tar.bz2  # bzip2
tar -xJf archive.tar.xz   # xz
```

**Options:**
- `x` - Extract

**Extract to specific directory:**
```bash
tar -xzf archive.tar.gz -C /path/to/dir
```

**Extract specific files:**
```bash
tar -xzf archive.tar.gz file.txt dir/
```

### Listing Contents

**List files in archive:**
```bash
tar -tzf archive.tar.gz
```
- `t` - List contents

**Detailed list:**
```bash
tar -tvzf archive.tar.gz
```

### Common tar Patterns

**Backup directory:**
```bash
tar -czf backup-$(date +%Y%m%d).tar.gz ~/important/
```

**Exclude files:**
```bash
tar -czf archive.tar.gz --exclude='*.log' --exclude='tmp/' project/
```

**Preserve permissions:**
```bash
tar -cpzf archive.tar.gz files/
```
- `p` - Preserve permissions

---

## Resource Limits

### ulimit Command

**Purpose:** Control per-user resource limits.

**View limits:**
```bash
ulimit -a  # All limits
```

**Common limits:**

| Option | Limit |
|--------|-------|
| `-c` | Core file size |
| `-d` | Data segment size |
| `-f` | File size |
| `-n` | Open files |
| `-u` | User processes |
| `-v` | Virtual memory |

**View specific limit:**
```bash
ulimit -u  # Max user processes
ulimit -n  # Max open files
```

**Set limits:**

**Soft limit (can be increased up to hard limit):**
```bash
ulimit -S -u 100  # Max 100 processes
```

**Hard limit (cannot exceed):**
```bash
ulimit -H -u 200
```

**Both:**
```bash
ulimit -u 150
```

**Example:**
```bash
# Check current process limit
ulimit -u
# Output: 15000

# Set to 50
ulimit -u 50

# Try exceeding
ls | wc -l      # Works (3 processes)
ls | cat | wc -l  # Works (4 processes)
# Keep adding pipes until error
```

---

## Terminal Multiplexing

### screen Command

**Purpose:** Multiple virtual terminals in one session.

**Benefits:**
- Multiple windows in one terminal
- Detach and reattach sessions
- Persist after disconnect

### Basic Usage

**Start screen:**
```bash
screen
```

**Commands (prefix: `Ctrl+a`):**

| Keys | Action |
|------|--------|
| `Ctrl+a c` | Create new window |
| `Ctrl+a n` | Next window |
| `Ctrl+a p` | Previous window |
| `Ctrl+a 0-9` | Switch to window 0-9 |
| `Ctrl+a "` | List all windows |
| `Ctrl+a A` | Rename window |
| `Ctrl+a d` | Detach session |
| `Ctrl+a k` | Kill window |
| `Ctrl+a [` | Copy mode (scroll) |

### Sessions

**Detach session:**
```bash
Ctrl+a d
```

**List sessions:**
```bash
screen -ls
```

**Reattach:**
```bash
screen -r         # Reattach to session
screen -r 12345   # Reattach to specific PID
```

**Create named session:**
```bash
screen -S mysession
```

**Reattach by name:**
```bash
screen -r mysession
```

### Configuration

**Status line (.screenrc):**
```bash
echo 'hardstatus alwayslastline' >> ~/.screenrc
echo 'hardstatus string "%{= kw}%-w%{= BW}%n %t%{-}%+w"' >> ~/.screenrc
```

Shows window list at bottom.

---

## Scheduled Tasks

### at Command

**Purpose:** Run command once at specified time.

**Schedule task:**
```bash
at now + 5 minutes
at> echo "Hello" > /tmp/output.txt
at> <Ctrl+D>
```

**Time specifications:**
```bash
at 15:30              # Today at 3:30 PM
at 3:30 PM tomorrow   # Tomorrow at 3:30 PM
at now + 1 hour       # One hour from now
at noon               # Today at noon
at midnight           # Tonight at midnight
at 10:00 AM 12/25/2024  # Specific date/time
```

**From file:**
```bash
at -f script.sh now + 10 minutes
```

**List scheduled jobs:**
```bash
atq
```

**Remove job:**
```bash
atrm JOB_NUMBER
```

### cron/crontab

**Purpose:** Run commands periodically.

**Edit crontab:**
```bash
crontab -e
```

**Crontab format:**
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, 0 or 7 = Sunday)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

**Examples:**
```bash
# Every minute
* * * * * /path/to/script.sh

# Every hour at minute 0
0 * * * * /path/to/script.sh

# Every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Every Monday at 8:00 AM
0 8 * * 1 /path/to/script.sh

# Every 15 minutes
*/15 * * * * /path/to/script.sh

# Twice daily (8 AM and 8 PM)
0 8,20 * * * /path/to/script.sh

# Weekdays at noon
0 12 * * 1-5 /path/to/script.sh
```

**Special strings:**
```bash
@reboot   /path/to/script.sh  # At boot
@hourly   /path/to/script.sh  # Every hour
@daily    /path/to/script.sh  # Every day at midnight
@weekly   /path/to/script.sh  # Every week
@monthly  /path/to/script.sh  # Every month
@yearly   /path/to/script.sh  # Every year
```

**View crontab:**
```bash
crontab -l
```

**Remove crontab:**
```bash
crontab -r
```

**Logging:**
```bash
# Redirect output
* * * * * /path/to/script.sh >> /path/to/log.txt 2>&1

# Or use logger
* * * * * /path/to/script.sh && logger "Script succeeded"
```

### watch Command

**Purpose:** Execute command repeatedly, display output.

**Basic usage:**
```bash
watch command
```

**Update interval:**
```bash
watch -n 5 command  # Every 5 seconds
watch -n 0.5 date   # Every 0.5 seconds
```

**Highlight differences:**
```bash
watch -d command
```

**Examples:**
```bash
watch -n 1 'ps aux | grep myprocess'
watch -n 2 'df -h'
watch -n 5 'tail -n 20 /var/log/syslog'
```

---

## Additional Tools

### umask (Covered in Lab 8, recap)

**Default permissions for new files:**
```bash
umask         # View current
umask 0077    # Set restrictive
umask 0022    # Set default
```

**How it works:**
- Files: 666 - umask
- Directories: 777 - umask

**Permanent:**
```bash
echo "umask 0077" >> ~/.bashrc
```

---

## Command Reference

### Compression

```bash
gzip file              # Compress
gunzip file.gz         # Decompress
gzip -c file > file.gz # Keep original
zcat file.gz           # View compressed
zgrep pattern file.gz  # Search compressed
```

### Archives

```bash
tar -czf archive.tar.gz dir/   # Create compressed
tar -xzf archive.tar.gz        # Extract
tar -tzf archive.tar.gz        # List contents
tar -czf backup.tar.gz --exclude='*.log' dir/  # Exclude files
```

### Resource Limits

```bash
ulimit -a              # View all limits
ulimit -u              # Process limit
ulimit -u 100          # Set limit
```

### Terminal Multiplexing

```bash
screen                 # Start screen
Ctrl+a c               # New window
Ctrl+a d               # Detach
screen -ls             # List sessions
screen -r              # Reattach
```

### Scheduling

```bash
at now + 5 minutes     # Schedule once
atq                    # List at jobs
atrm JOB_ID            # Remove at job
crontab -e             # Edit cron jobs
crontab -l             # List cron jobs
watch -n 1 command     # Repeat command
```

---

## Best Practices

### 1. Compress Before Archiving

```bash
# Good (one step)
tar -czf archive.tar.gz files/

# Unnecessary (two steps)
tar -cf archive.tar files/
gzip archive.tar
```

### 2. Verify Archives

```bash
# Create with verify
tar -cvzf archive.tar.gz files/

# List contents after creation
tar -tzf archive.tar.gz
```

### 3. Backup Strategies

```bash
# Date-stamped backups
tar -czf backup-$(date +%Y%m%d-%H%M%S).tar.gz ~/data/

# Keep multiple versions
find ~/backups -name "backup-*.tar.gz" -mtime +7 -delete
```

### 4. Cron Best Practices

```bash
# Use absolute paths
30 2 * * * /usr/bin/python3 /home/user/script.py

# Log output
0 * * * * /path/to/script.sh >> /var/log/myscript.log 2>&1

# Set environment
PATH=/usr/local/bin:/usr/bin:/bin
0 * * * * my_command
```

### 5. Screen for Long Tasks

```bash
# Start screen
screen -S longtask

# Run long process
./long_process.sh

# Detach: Ctrl+a d
# Logout
# Login later
screen -r longtask  # Resume
```

---

## Common Patterns

### Automated Backup Script

```bash
#!/bin/bash

BACKUP_DIR=~/backups
SOURCE=~/important_data
DATE=$(date +%Y%m%d)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/backup-$DATE.tar.gz" "$SOURCE"

# Keep only last 7 days
find "$BACKUP_DIR" -name "backup-*.tar.gz" -mtime +7 -delete
```

**Schedule daily:**
```bash
crontab -e
# Add:
0 2 * * * /home/user/backup_script.sh
```

### Monitor Directory Size

```bash
watch -n 60 'du -sh ~/project/*'
```

### Compress Old Logs

```bash
#!/bin/bash

LOG_DIR=/var/log/myapp

find "$LOG_DIR" -name "*.log" -mtime +1 -exec gzip {} \;
```

---

## Summary

Lab 9 covers advanced system management:

**Compression:**
- `gzip`/`gunzip` - Compress/decompress files
- `zcat`, `zgrep` - Work with compressed files

**Archives:**
- `tar -czf` - Create compressed archive
- `tar -xzf` - Extract
- `tar -tzf` - List contents

**Resource Limits:**
- `ulimit` - Control resource usage
- Process, file, memory limits

**Screen:**
- Multiple terminals in one
- Detach/reattach sessions
- Persist after logout

**Scheduling:**
- `at` - One-time tasks
- `cron` - Recurring tasks
- `watch` - Repeated execution

**Key Principle:** Automation and resource management enable efficient system administration.
