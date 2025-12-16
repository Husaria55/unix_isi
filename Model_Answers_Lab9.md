# Model Answers - Lab 9: System Management, Archives, and Scheduling

## Introduction Questions

**Q1: Co nam daje kompresja w czasach dużej pamięci?**

**Answer:** 
Even with large storage, compression remains valuable:
- **Network transfer** - Smaller files transfer faster
- **Backup efficiency** - Store more backups in the same space
- **Archival storage** - Long-term data storage costs
- **Disk I/O** - Reading smaller files is faster
- **Email attachments** - Size limits still exist
- **Cloud storage costs** - Pay per GB stored/transferred

**Q2: Co to za rodzaj archiwum "tar"?**

**Answer:** 
**tar** (Tape Archive) is an uncompressed archive format that bundles multiple files and directories into a single file while preserving:
- Directory structure
- File permissions
- Timestamps
- Ownership

Originally designed for tape backups, now commonly used with compression (gzip, bzip2, xz) for efficient file distribution and backups. Unlike ZIP, tar itself doesn't compress - it only bundles files.

**Q3: Czemu wprowadzać limity na procesy?**

**Answer:** 
Process limits prevent:
- **Fork bombs** - Malicious/buggy code creating infinite processes
- **Resource exhaustion** - One user consuming all system resources
- **System crashes** - Too many processes make system unresponsive
- **Fair sharing** - Ensure all users get reasonable access

Example: User accidentally runs `while true; do bash & done` - without limits, this crashes the server.

**Q4: Co nam daje narzędzie `screen`? Kiedy go używać?**

**Answer:** 
**screen** provides terminal persistence and multiplexing:

**Benefits:**
- Survive network disconnects
- Multiple terminals in one SSH connection
- Long-running tasks continue after logout
- Share sessions between users

**When to use:**
- Running long compilations/processes
- Server administration (keep terminal open)
- Unreliable network connections
- Need multiple windows but limited terminals

**Q5: Co tak właściwie daje nam cron? Kiedy jego użycie jest potrzebne?**

**Answer:** 
**cron** automates recurring tasks without manual intervention:

**Use cases:**
- Automated backups (daily, weekly)
- Log rotation and cleanup
- System maintenance (updates, cleanup)
- Report generation (daily statistics)
- Monitoring alerts
- Data synchronization

Essential when tasks must run:
- At specific times (night backups)
- Regularly without human intervention
- Even when user is not logged in

---

## Task Solutions

### Task 1: File Compression

**Requirement:** Test compression, compress files, compare sizes, search compressed content.

**Solution:**

```bash
# Create new directory
mkdir compression_test
cd compression_test

# Copy /etc/passwd
cp /etc/passwd .

# Create reversed file (dwssap)
tac passwd > dwssap

# Compress all files
for file in *; do
    gzip -c "$file" > "${file}.gz"
done

# Alternative one-liner
gzip -c passwd > passwd.gz
gzip -c dwssap > dwssap.gz

# Compare sizes
ls -lh passwd passwd.gz
# Example output:
# -rw-r--r-- 1 user user 2.5K passwd
# -rw-r--r-- 1 user user 1.1K passwd.gz

# Calculate compression ratio
original=$(stat -c%s passwd)
compressed=$(stat -c%s passwd.gz)
ratio=$(echo "scale=2; ($original - $compressed) / $original * 100" | bc)
echo "Compression ratio: ${ratio}%"

# Search for username in compressed files
zcat *.gz | grep "$USER"
# Or search each file
zgrep "$USER" *.gz
```

**Explanation:**

**Step 1: Directory and files**
- `tac` reverses line order (opposite of `cat`)
- Creates test files for compression

**Step 2: Compression**
- `gzip -c file` writes to stdout (preserves original)
- Without `-c`, gzip removes original file
- Loop compresses each file

**Step 3: Size comparison**
- `ls -lh` shows human-readable sizes
- Text files compress well (40-60% typically)
- `/etc/passwd` is repetitive text → good compression

**Step 4: Search compressed**
- `zcat` decompresses to stdout (like gunzip -c)
- `zgrep` searches compressed files directly
- No need to decompress permanently

**Verification:**
```bash
# Verify integrity
gunzip -t passwd.gz
echo $?  # Should be 0

# Decompress and compare
gunzip -c passwd.gz > passwd.restored
diff passwd passwd.restored  # Should be identical
```

---

### Task 2: tar Archives

**Requirement:** Create compressed tar archive, list contents, extract to new directory.

**Solution:**

```bash
# Assume we're in parent directory of compression_test/
# Create compressed tar archive
tar -czf compression_test.tar.gz compression_test/

# Alternative: verbose output
tar -cvzf compression_test.tar.gz compression_test/

# List contents
tar -tzf compression_test.tar.gz

# Detailed list (like ls -l)
tar -tvzf compression_test.tar.gz

# Extract to new directory
mkdir extraction_test
tar -xzf compression_test.tar.gz -C extraction_test/

# Verify extraction
diff -r compression_test/ extraction_test/compression_test/
```

**Explanation:**

**tar options breakdown:**
- `c` - Create archive
- `x` - Extract archive
- `t` - List (table of contents)
- `z` - gzip compression
- `v` - Verbose (show files)
- `f` - File name (must be last flag before filename)

**Archive creation:**
```bash
tar -czf archive.tar.gz directory/
```
- Creates single compressed archive
- Preserves directory structure
- Maintains permissions and timestamps

**Listing contents:**
```bash
tar -tzf archive.tar.gz
```
- Shows all files in archive
- Doesn't extract anything
- Useful before extraction to verify contents

**Extraction:**
```bash
tar -xzf archive.tar.gz -C /destination/
```
- Recreates original directory structure
- `-C` specifies extraction location
- Without `-C`, extracts to current directory

**Common patterns:**

```bash
# Backup with timestamp
tar -czf backup-$(date +%Y%m%d).tar.gz ~/important/

# Exclude files
tar -czf archive.tar.gz --exclude='*.log' --exclude='tmp/' project/

# Extract single file
tar -xzf archive.tar.gz path/to/specific/file.txt

# Append to existing (uncompressed only)
tar -rf archive.tar newfile.txt
```

**Verification:**
```bash
# Test archive integrity
tar -tzf compression_test.tar.gz > /dev/null
echo $?  # Should be 0 if OK

# Compare extracted with original
diff -r compression_test/ extraction_test/compression_test/
```

---

### Task 3: Process Limits (ulimit)

**Requirement:** Check process limit, set soft limit 2 above current, test exceeding.

**Solution:**

```bash
# Check current process usage (including threads)
current_processes=$(ps x -L | wc -l)
echo "Current processes (with threads): $current_processes"

# Subtract 1 for header line
actual_count=$((current_processes - 1))
echo "Actual LWP count: $actual_count"

# Check current limit
ulimit -u
# Example output: 15000

# Set soft limit 2 above current usage
new_limit=$((actual_count + 2))
ulimit -S -u $new_limit
echo "Set limit to: $new_limit"

# Test the limit - gradually increase pipeline length
echo "Testing limit..."

ls | wc -l
# This creates 3 processes: ls, wc, and the subshell

ls | cat | wc -l
# This creates 4 processes

ls | cat | cat | wc -l
# This creates 5 processes

ls | cat | cat | cat | wc -l
# Eventually you'll see:
# -bash: fork: retry: Resource temporarily unavailable
# or
# -bash: fork: retry: No child processes
```

**Explanation:**

**Process counting:**
```bash
ps x -L | wc -l
```
- `ps x` - Shows your processes
- `-L` - Shows threads (LWP = Light Weight Process)
- Each pipeline creates multiple processes

**Setting limits:**
```bash
ulimit -S -u $new_limit
```
- `-S` - Soft limit (can increase up to hard limit)
- `-u` - User processes
- Soft limit can be increased by user (up to hard)
- Hard limit (`-H`) can only be decreased

**Pipeline processes:**
When you run `ls | cat | wc -l`:
1. Bash forks for pipeline
2. `ls` process created
3. `cat` process created
4. `wc` process created
5. Total: ~4 new processes

**Testing limit:**
- Start with simple commands
- Add more pipes gradually
- When approaching limit, fork() fails
- Error: "Resource temporarily unavailable"

**Practical example:**
```bash
# Before setting limit
ulimit -u
# Output: 15000

ps x -L | wc -l
# Output: 23 (22 processes + 1 header)

# Set strict limit
ulimit -u 24

# This works (total processes still under 24)
ls | wc -l

# This might fail (too many processes)
ls | cat | cat | cat | cat | wc -l
# Error: bash: fork: retry: Resource temporarily unavailable
```

**Recovery:**
```bash
# Reset limit to default
ulimit -u unlimited  # If allowed
# Or open new terminal (limit is per-session)
```

---

### Task 4: umask Configuration

**Requirement:** Configure bash so new files are accessible only to owner, make permanent.

**Solution:**

```bash
# Check current umask
umask
# Example output: 0022

# Test current behavior
touch test_old.txt
ls -l test_old.txt
# Output: -rw-r--r-- (644 - others can read)

# Set restrictive umask
umask 0077

# Test new behavior
touch test_new.txt
ls -l test_new.txt
# Output: -rw------- (600 - only owner access)

mkdir test_dir
ls -ld test_dir
# Output: drwx------ (700 - only owner access)

# Make permanent - add to ~/.bashrc
echo "umask 0077" >> ~/.bashrc

# Verify addition
tail -1 ~/.bashrc

# Test persistence
# Logout and login again, or:
bash  # Start new shell
umask  # Should show 0077
```

**Explanation:**

**umask calculation:**
- **Files:** 666 - umask = permissions
- **Directories:** 777 - umask = permissions

**Common umask values:**

| umask | Files | Dirs | Description |
|-------|-------|------|-------------|
| 0022 | 644 | 755 | Default - others can read |
| 0027 | 640 | 750 | Group can read |
| 0077 | 600 | 700 | Owner only |
| 0002 | 664 | 775 | Group can write |

**Examples:**

```bash
# umask 0022 (default)
# Files: 666 - 022 = 644 (rw-r--r--)
# Dirs:  777 - 022 = 755 (rwxr-xr-x)

# umask 0077 (restrictive)
# Files: 666 - 077 = 600 (rw-------)
# Dirs:  777 - 077 = 700 (rwx------)
```

**Making permanent:**

```bash
# Add to ~/.bashrc
echo "umask 0077" >> ~/.bashrc
```

**Verification:**
```bash
# Test in new shell
bash -c 'umask'  # Should show 0077 after adding to .bashrc

# Test file creation
rm -f test.txt
touch test.txt
ls -l test.txt  # Should be -rw-------
```

**Note:** umask affects new files only, not existing files.

---

### Task 5: screen Basics

**Requirement:** Use screen with 3 windows (vi, man screen, watch date), detach, reattach.

**Solution:**

```bash
# Start screen
screen

# You're now in window 0
# Start vi
vi myfile.txt

# Create new window: Ctrl+a c
# You're now in window 1
man screen

# Create another window: Ctrl+a c
# You're now in window 2
watch date

# Navigate between windows:
# Ctrl+a 0  →  Go to window 0 (vi)
# Ctrl+a 1  →  Go to window 1 (man)
# Ctrl+a 2  →  Go to window 2 (watch)
# Ctrl+a n  →  Next window
# Ctrl+a p  →  Previous window
# Ctrl+a "  →  List all windows (menu)

# In vi window (Ctrl+a 0):
# Type some text, save with :w

# Detach session: Ctrl+a d
# You're back to regular shell
# Message: [detached from 12345.pts-1.hostname]

# Logout
exit

# --- Later, after re-login ---

# List active screen sessions
screen -ls
# Output:
# There is a screen on:
#     12345.pts-1.hostname    (Detached)
# 1 Socket in /run/screen/S-username.

# Reattach to session
screen -r

# Or specify by ID:
screen -r 12345

# You're back in screen!
# All windows still running:
# - vi has your text
# - man screen still open
# - watch date still updating
```

**Explanation:**

**Key screen commands:**

| Command | Action |
|---------|--------|
| `screen` | Start new session |
| `Ctrl+a c` | Create new window |
| `Ctrl+a n` | Next window |
| `Ctrl+a p` | Previous window |
| `Ctrl+a 0-9` | Go to window 0-9 |
| `Ctrl+a "` | List windows |
| `Ctrl+a d` | Detach session |
| `Ctrl+a k` | Kill current window |
| `Ctrl+a A` | Rename window |

**Detach vs. Exit:**
- **Detach (Ctrl+a d):** Processes continue running
- **Exit:** Terminates current window only

**Session management:**
```bash
# Start named session
screen -S mysession

# List sessions
screen -ls

# Reattach by name
screen -r mysession

# Reattach to attached session (force)
screen -x mysession  # Share session

# Kill session
screen -X -S 12345 quit
```

**Workflow demonstration:**

```bash
# Window 0: Long-running task
screen
./long_compilation.sh
# Ctrl+a c

# Window 1: Monitor logs
tail -f /var/log/syslog
# Ctrl+a c

# Window 2: Edit code
vi main.c

# Detach: Ctrl+a d
# Logout, go home

# Next day, re-login
screen -r
# Everything still running!
```

**Common gotcha:**
- `Ctrl+a` is screen's command prefix
- In vi, `Ctrl+a` increments number
- In screen, press `Ctrl+a a` to send `Ctrl+a` to application

---

### Task 6: screen Status Line

**Requirement:** Configure screen to show open windows in last line.

**Solution:**

```bash
# Create/edit ~/.screenrc
vi ~/.screenrc

# Add these lines:
hardstatus alwayslastline
hardstatus string '%{= kw}%-w%{= BW}%n %t%{-}%+w'

# Save and exit

# Test configuration
# Start new screen session
screen

# Create multiple windows
# Ctrl+a c  (creates window 1)
# Ctrl+a c  (creates window 2)

# You should see at bottom:
# 0 bash  1*bash  2 bash
#
# Where * indicates current window

# Alternative status line formats:

# Detailed status with load
hardstatus string '%{= kG}[ %{G}%H %{g}][%= %{=kw}%?%-Lw%?%{r}(%{W}%n*%f%t%?(%u)%?%{r})%{w}%?%+Lw%?%?%= %{g}][%{B}%Y-%m-%d %{W}%c %{g}]'

# Simple window list
hardstatus string '%{= wk}%-w%{= BW}%n %t%{-}%+w'

# With hostname and time
hardstatus string '%{= kG}[%{G}%H%{g}][%= %{= kw}%?%-Lw%?%{r}(%{W}%n*%f %t%?(%u)%?%{r})%{w}%?%+Lw%?%?%= %{g}][%{B} %m-%d %{W}%c %{g}]'
```

**Explanation:**

**~/.screenrc configuration:**
- `hardstatus alwayslastline` - Show status in last line
- `hardstatus string` - Format string

**Format codes:**

| Code | Meaning |
|------|---------|
| `%n` | Window number |
| `%t` | Window title |
| `%H` | Hostname |
| `%c` | Current time |
| `%w` | Window list |
| `%-w` | Windows before current |
| `%+w` | Windows after current |
| `%{= kw}` | Color (black bg, white fg) |
| `%{= BW}` | Bold white |

**Recommended simple configuration:**

```bash
# ~/.screenrc
hardstatus alwayslastline
hardstatus string '%{= kw}%-w%{= BW}%n %t%{-}%+w'

# Additional useful settings
startup_message off     # Disable startup message
defscrollback 10000     # Increase scrollback buffer
altscreen on            # Clear screen after closing apps like vi
```

**Status line examples:**

**Simple:**
```
0 bash  1*vi  2 bash
```

**Detailed:**
```
[hostname] 0 bash  1*vi  2 bash  3 top [12:34]
```

**Reload configuration:**
```bash
# In running screen session
Ctrl+a :
source ~/.screenrc
```

**Verification:**
```bash
# Start screen
screen

# Status line appears at bottom

# Create windows and switch between them
# Status updates to show current window with highlight
```

---

### Task 7: Monitor bash Processes

**Requirement:** Use two terminals, monitor bash process count every 1 second.

**Solution:**

**Terminal 1:**

```bash
# Using watch
watch -n 1 'ps -u $USER | grep bash | wc -l'

# Or using loop
while true; do
    clear
    echo "Bash processes: $(ps -u $USER | grep bash | wc -l)"
    echo "Time: $(date)"
    sleep 1
done

# Better version - show processes
watch -n 1 'ps -u $USER | grep bash'
```

**Terminal 2 (or screen window):**

```bash
# Login creates new bash process
# If using screen:
# Ctrl+a c  (creates new window with new bash)

# Or SSH from another terminal
ssh username@server
```

**Result in Terminal 1:**
```
Before: Bash processes: 2
After:  Bash processes: 3

# Breakdown:
# - Initial login shell
# - Shell running watch command
# - New login shell (Terminal 2)
```

**Explanation:**

**Counting bash processes:**

```bash
ps -u $USER | grep bash | wc -l
```
- `ps -u $USER` - Your processes only
- `grep bash` - Filter bash processes
- `wc -l` - Count lines

**Each terminal creates bash:**
- SSH login → new bash
- Screen window → new bash
- Terminal emulator → new bash

**Better monitoring script:**

```bash
#!/bin/bash
# monitor_bash.sh

while true; do
    clear
    echo "=== Bash Process Monitor ==="
    echo "Time: $(date)"
    echo
    
    count=$(ps -u $USER | grep -c bash)
    echo "Total bash processes: $count"
    echo
    
    echo "Process list:"
    ps -u $USER | grep bash | awk '{print $1, $4}'
    
    sleep 1
done
```

**Using screen for both windows:**

```bash
# Window 0: Monitor
watch -n 1 'ps -u $USER | grep bash | wc -l'

# Ctrl+a c  (new window)
# Window 1: Do work

# Ctrl+a 0 to see count increased
```

**Advanced monitoring:**

```bash
# Show bash with their terminals
watch -n 1 'ps -u $USER -o pid,tty,cmd | grep bash'

# Output:
#  PID TTY      CMD
# 1234 pts/0    bash
# 1456 pts/1    bash
# 1567 pts/2    bash
```

**Verification:**
- Start with one terminal: count = 1 or 2
- Open second terminal: count increases by 1
- Close terminal: count decreases by 1

---

### Task 8: at Command - Delayed Message

**Requirement:** Use `at` to display message in terminal at scheduled time.

**Solution:**

```bash
# Find your terminal device
my_terminal=$(ps x | grep bash | tr -s ' ' | cut -d ' ' -f 3 | head -1)
echo "My terminal: $my_terminal"
# Example output: /dev/pts/0

# Alternative method
my_terminal=$(who | grep $USER | tr -s ' ' | cut -d' ' -f2 | head -1)
echo "My terminal: $my_terminal"
# Example output: pts/0 (prepend /dev/)

# Schedule message for 2 minutes from now
at now + 2 minutes <<EOF
echo "Już czas..." > /dev/$my_terminal
EOF
# Or if my_terminal already has /dev/:
at now + 2 minutes <<EOF
echo "Już czas..." > $my_terminal
EOF

# Confirm scheduled job
atq
# Output:
# 1  Thu Dec 12 14:35:00 2024 a username

# Wait 2 minutes...
# Message appears in your terminal: "Już czas..."
```

**Complete script example:**

```bash
#!/bin/bash
# delayed_message.sh

# Get terminal
my_tty=$(tty)
echo "Scheduling message to: $my_tty"

# Schedule
at now + 2 minutes <<EOF
echo "Już czas..." > $my_tty
echo "Current time: \$(date)" > $my_tty
EOF

echo "Message scheduled. Check with: atq"
```

**Explanation:**

**Finding terminal device:**

Method 1 - from ps:
```bash
ps x | grep bash | tr -s ' ' | cut -d ' ' -f 3
# Output: /dev/pts/0
```

Method 2 - from who:
```bash
who | grep $USER | tr -s ' ' | cut -d' ' -f2
# Output: pts/0 (need to add /dev/)
```

Method 3 - using tty:
```bash
tty
# Output: /dev/pts/0
```

**Scheduling with at:**

```bash
at now + 2 minutes <<EOF
commands here
EOF
```

**at time specifications:**
```bash
at now + 5 minutes
at 15:30              # Today at 3:30 PM
at 3:30 PM tomorrow
at noon
at midnight
at 16:00 Dec 25
```

**Writing to terminal:**
```bash
echo "message" > /dev/pts/0
```
- Writes directly to terminal device
- User sees message immediately
- Requires write permission to terminal

**Managing at jobs:**
```bash
# List jobs
atq

# Remove job
atrm 1  # Remove job number 1

# View job contents
at -c 1 | tail -20
```

**Practical example - reminder system:**

```bash
#!/bin/bash
# reminder.sh

if [ $# -lt 2 ]; then
    echo "Usage: $0 <time> <message>"
    echo "Example: $0 '15:30' 'Meeting time!'"
    exit 1
fi

time=$1
message=$2
my_tty=$(tty)

at "$time" <<EOF
echo "=== REMINDER ===" > $my_tty
echo "$message" > $my_tty
echo "================" > $my_tty
EOF

echo "Reminder set for $time"
atq
```

**Usage:**
```bash
./reminder.sh "now + 10 minutes" "Take a break!"
```

---

### Task 9: cron Backup

**Requirement:** Schedule daily backup of ~/work at specific time (5 minutes from now).

**Solution:**

```bash
# Create work directory with test files
mkdir -p ~/work
echo "Important data" > ~/work/file1.txt
echo "More data" > ~/work/file2.txt

# Check current time
date
# Example: Thu Dec 12 14:25:00 UTC 2024

# Calculate time for 5 minutes from now
# If current time is 14:25, schedule for 14:30

# Edit crontab
crontab -e

# Add line (adjust minute and hour to 5 minutes from now):
30 14 * * * cp -r ~/work ~/work.kopia

# Or better - timestamped backup:
30 14 * * * cp -r ~/work ~/work.backup-$(date +\%Y\%m\%d-\%H\%M)

# Save and exit

# Verify crontab
crontab -l
# Output:
# 30 14 * * * cp -r ~/work ~/work.kopia

# Wait 5 minutes...

# After 5 minutes, check:
ls -ld ~/work.kopia
# Should exist

# Compare directories
diff -r ~/work ~/work.kopia
# Should be identical
```

**Better backup script:**

```bash
# Create backup script
cat > ~/backup_work.sh <<'EOF'
#!/bin/bash

SOURCE=~/work
BACKUP_DIR=~
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
BACKUP_NAME="work.backup-$TIMESTAMP"

# Check if source exists
if [ ! -d "$SOURCE" ]; then
    echo "Error: $SOURCE does not exist" >&2
    exit 1
fi

# Create backup
cp -r "$SOURCE" "$BACKUP_DIR/$BACKUP_NAME"

# Verify
if [ $? -eq 0 ]; then
    echo "Backup successful: $BACKUP_NAME"
else
    echo "Backup failed!" >&2
    exit 1
fi

# Keep only last 7 backups
cd "$BACKUP_DIR"
ls -t work.backup-* | tail -n +8 | xargs -r rm -rf

EOF

# Make executable
chmod +x ~/backup_work.sh

# Test script
~/backup_work.sh

# Add to crontab (schedule for 5 minutes from now)
crontab -e
# Add line (adjust time):
30 14 * * * /home/username/backup_work.sh >> /home/username/backup.log 2>&1
```

**Explanation:**

**crontab format:**
```
30 14 * * * command
│  │  │ │ │
│  │  │ │ └─── Day of week (0-7, 0=7=Sunday)
│  │  │ └───── Month (1-12)
│  │  └─────── Day of month (1-31)
│  └───────── Hour (0-23)
└─────────── Minute (0-59)
```

**Examples:**
```bash
# Every day at 2:30 AM
30 2 * * * command

# Every hour
0 * * * * command

# Every 15 minutes
*/15 * * * * command

# Weekdays at 9 AM
0 9 * * 1-5 command
```

**Crontab management:**
```bash
crontab -e   # Edit
crontab -l   # List
crontab -r   # Remove all
```

**Logging:**
```bash
# Redirect output to log
30 14 * * * ~/backup_work.sh >> ~/backup.log 2>&1

# Using logger
30 14 * * * ~/backup_work.sh && logger "Backup succeeded" || logger "Backup failed"
```

**Important notes:**
- cron uses absolute paths
- Environment different from login shell
- Specify full paths to commands and files
- Test scripts manually before adding to cron

**Verification:**
```bash
# After 5 minutes
ls -l ~/work.kopia
ls -l ~/work
diff -r ~/work ~/work.kopia

# Check cron ran
grep CRON /var/log/syslog  # Ubuntu/Debian
# Or check logs:
tail ~/backup.log
```

---

### Task 10: Remove cron Entry

**Requirement:** Remove the cron entry added in previous task.

**Solution:**

```bash
# Method 1: Edit crontab
crontab -e

# Delete the line:
# 30 14 * * * cp -r ~/work ~/work.kopia

# Save and exit

# Verify removal
crontab -l
# Should not show the backup line

# Method 2: Remove all cron jobs
crontab -r

# Verify
crontab -l
# Output: no crontab for username

# Method 3: Using sed (careful!)
crontab -l | grep -v "work.kopia" | crontab -

# Explanation:
# crontab -l             → List current crontab
# grep -v "work.kopia"   → Exclude lines with "work.kopia"
# crontab -              → Install filtered crontab
```

**Explanation:**

**Editing crontab:**
- `crontab -e` opens editor (usually vi or nano)
- Delete unwanted lines
- Save and exit
- Changes take effect immediately

**Removing all cron jobs:**
```bash
crontab -r
```
- **Warning:** Removes ALL cron jobs
- No confirmation prompt
- Use carefully

**Selective removal:**
```bash
# Backup current crontab first
crontab -l > ~/crontab.backup

# Remove specific entry
crontab -l | grep -v "pattern" | crontab -

# Restore if needed
crontab ~/crontab.backup
```

**Verification:**
```bash
# List cron jobs
crontab -l

# Check no jobs scheduled
crontab -l | wc -l
# Should be 0 or only show other jobs

# Wait past scheduled time
# Backup should NOT be created
```

**Managing cron entries:**

```bash
# View current crontab
crontab -l

# Edit
crontab -e

# Remove all
crontab -r

# Install from file
crontab ~/mycrontab.txt

# List other user's crontab (requires sudo)
sudo crontab -l -u username
```

---

### Task 11: umask Configuration (Duplicate of Task 4)

**Solution:** See Task 4 - this is a duplicate task.

```bash
# Set restrictive umask
umask 0077

# Make permanent
echo "umask 0077" >> ~/.bashrc

# Verify
bash -c 'umask'  # Should show 0077
```

---

### Task 12: User Login Alert Script

**Requirement:** Script that monitors and alerts when specified user logs in.

**Solution:**

```bash
#!/bin/bash
# useralert - Alert when user logs in

if [ $# -ne 1 ]; then
    echo "Usage: $0 <username>"
    echo "Example: $0 wojnicki &"
    exit 1
fi

target_user=$1
my_tty=$(tty)

echo "Monitoring for user: $target_user"
echo "Alerts will appear on: $my_tty"

while true; do
    # Check if user is logged in
    if who | grep -q "^$target_user "; then
        # User is logged in
        echo "===== USER ALERT =====" > "$my_tty"
        echo "User $target_user has logged in!" > "$my_tty"
        echo "Time: $(date)" > "$my_tty"
        echo "======================" > "$my_tty"
        
        # Wait until user logs out
        while who | grep -q "^$target_user "; do
            sleep 5
        done
        
        echo "User $target_user has logged out." > "$my_tty"
    fi
    
    sleep 5
done
```

**Enhanced version - alert all terminals:**

```bash
#!/bin/bash
# useralert - Alert when user logs in (all terminals)

if [ $# -ne 1 ]; then
    echo "Usage: $0 <username>"
    echo "Example: $0 wojnicki &"
    exit 1
fi

target_user=$1

echo "Monitoring for user: $target_user"
echo "Alerts will appear on all your terminals"

# Function to send alert to all my terminals
send_alert() {
    local message=$1
    
    # Get all my terminal devices
    who | grep "^$USER " | tr -s ' ' | cut -d' ' -f2 | while read terminal; do
        echo "$message" > "/dev/$terminal" 2>/dev/null
    done
}

while true; do
    # Check if target user is logged in
    if who | grep -q "^$target_user "; then
        # User logged in
        send_alert "===== USER ALERT ====="
        send_alert "User $target_user has logged in!"
        send_alert "Time: $(date)"
        send_alert "======================"
        
        # Wait until user logs out
        while who | grep -q "^$target_user "; do
            sleep 5
        done
        
        send_alert "User $target_user has logged out."
    fi
    
    sleep 5
done
```

**Installation and usage:**

```bash
# Save script
cat > ~/useralert <<'EOF'
[script content here]
EOF

# Make executable
chmod +x ~/useralert

# Add to PATH (optional)
mkdir -p ~/bin
mv ~/useralert ~/bin/
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Use the script
useralert wojnicki &
# Output: [1] 12345 (job ID and PID)

# Continue working - you'll be alerted when user logs in

# Check if running
jobs
ps aux | grep useralert

# Stop monitoring
kill %1  # Kill job 1
# Or
pkill -f "useralert wojnicki"
```

**Explanation:**

**Script logic:**

```
endless loop {
    check if user logged in (who | grep)
    if logged in {
        send alert
        wait until logged out
        send logout message
    }
    sleep 5 seconds
}
```

**Key components:**

1. **Check login:**
```bash
who | grep -q "^$target_user "
```
- `who` lists logged in users
- `grep -q` returns success if found
- `^` ensures username at start of line

2. **Find terminals:**
```bash
who | grep "^$USER " | cut -d' ' -f2
```
- Get all my terminals
- Extract terminal device name

3. **Send message:**
```bash
echo "message" > "/dev/$terminal"
```
- Write to terminal device
- Requires write permission (mesg y)

4. **Background execution:**
```bash
useralert wojnicki &
```
- `&` runs in background
- Returns PID
- Can continue working

**Testing:**

```bash
# Terminal 1: Start monitoring
./useralert wojnicki &

# Terminal 2: Simulate login (if testing on same machine)
# SSH from another session as wojnicki
# Or use: su - wojnicki

# Terminal 1 will show:
# ===== USER ALERT =====
# User wojnicki has logged in!
# Time: Thu Dec 12 15:30:45 UTC 2024
# ======================
```

**Advanced features:**

```bash
# Log to file
useralert wojnicki > ~/useralert.log 2>&1 &

# Alert with sound (if terminal supports it)
echo -e "\a===== USER ALERT =====" > "$my_tty"

# Email notification
send_alert "User $target_user logged in" | mail -s "Login Alert" user@example.com
```

**Stopping the script:**

```bash
# Find PID
ps aux | grep useralert

# Kill by PID
kill 12345

# Or kill by job
jobs
kill %1

# Kill all instances
pkill -f useralert
```

---

## Summary

Lab 9 covered essential system administration tools:

**Compression:** gzip reduces file sizes for storage and transfer  
**Archives:** tar bundles files preserving structure and permissions  
**Limits:** ulimit prevents resource exhaustion  
**Screen:** Terminal persistence and multiplexing for long tasks  
**Scheduling:** cron automates recurring tasks, at schedules one-time jobs  
**Monitoring:** Scripts can track system events and alert users

These tools form the foundation of automated system maintenance and efficient resource management.
