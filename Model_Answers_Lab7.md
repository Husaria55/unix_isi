# Model Answers - Lab 7: Advanced BASH

## Task Solutions

### Task 1 & 2: SSH Key Generation and Exchange

**Objective:** Generate SSH keys and configure password-less login.

**Task 1: Generate Key Pair**

**Step 1: Generate keys**
```bash
ssh-keygen
```

**Interactive prompts:**
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa): [Press Enter - use default]
Created directory '/home/user/.ssh'.
Enter passphrase (empty for no passphrase): [Press Enter - no password]
Enter same passphrase again: [Press Enter]
```

**Output:**
```
Your identification has been saved in /home/user/.ssh/id_rsa
Your public key has been saved in /home/user/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:abc123... user@computer
```

**⚠️ Warning:** Don't overwrite existing keys if you're already using them!

**Verify keys created:**
```bash
ls -la ~/.ssh/
```

**Output:**
```
-rw------- 1 user user 2602 Dec 16 14:30 id_rsa        # Private key
-rw-r--r-- 1 user user  571 Dec 16 14:30 id_rsa.pub    # Public key
```

---

**Task 2: Copy Key to Server (Using SSH, Not SCP)**

**Method: Manual with SSH**

**Step 1: View your public key**
```bash
cat ~/.ssh/id_rsa.pub
```

**Copy the entire output** (looks like):
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC... user@computer
```

**Step 2: SSH to server (with password)**
```bash
ssh student@server
# Enter password when prompted
```

**Step 3: Create .ssh directory on server**
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
```

**Step 4: Add public key**
```bash
vim ~/.ssh/authorized_keys
# Paste the public key (one line)
# Save and exit (:wq)
```

**Step 5: Set correct permissions**
```bash
chmod 600 ~/.ssh/authorized_keys
```

**Step 6: Exit and test**
```bash
exit  # Logout from server

# Try connecting again
ssh student@server
# Should connect WITHOUT asking for password!
```

---

**Alternative: One-liner (still using ssh, not scp)**

From local machine:
```bash
cat ~/.ssh/id_rsa.pub | ssh student@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

**Explanation:**
- `cat ~/.ssh/id_rsa.pub` - Read public key
- `|` - Pipe to ssh
- `ssh student@server "commands"` - Execute commands on server
- `mkdir -p ~/.ssh` - Create directory
- `cat >> ~/.ssh/authorized_keys` - Append piped key
- `chmod` - Set correct permissions

---

### Task 3: Calculator Script with bc

**Objective:** Create script `k` that evaluates arithmetic expressions with floating point.

**Solution:**

```bash
vim ~/bin/k
```

**Script content:**
```bash
#!/bin/bash

# k - Calculator using bc

# Check if argument provided
if [ $# -eq 0 ]; then
    echo "Usage: k 'expression'" >&2
    echo "Example: k '2.2*9'" >&2
    exit 1
fi

# Evaluate expression using bc
result=$(echo "scale=10; $1" | bc)

# Output result
echo "$result"
```

**Make executable:**
```bash
chmod +x ~/bin/k
```

**Usage examples:**
```bash
k '2.2*9'
# Output: 19.8

k '10/3'
# Output: 3.3333333333

k '2^10'
# Output: 1024

k 'sqrt(16)'
# Output: 4

k '(5+3)*2/4'
# Output: 4.0000000000
```

**Enhanced version with better output:**
```bash
#!/bin/bash

if [ $# -eq 0 ]; then
    echo "Usage: k 'expression'" >&2
    exit 1
fi

# Calculate with scale=10
result=$(echo "scale=10; $1" | bc)

# Remove trailing zeros
result=$(echo "$result" | sed 's/\.0*$//' | sed 's/\.\([0-9]*[1-9]\)0*$/.\1/')

echo "$result"
```

**Tests:**
```bash
k '10/3'        # 3.3333333333
k '5/2'         # 2.5
k '4/2'         # 2 (not 2.0)
```

---

### Task 4: System Monitor with Signal Handling

**Objective:** Script that logs system info every 3 seconds, handles signals.

**Solution:**

```bash
vim ~/monitor.sh
```

**Script content:**
```bash
#!/bin/bash

# System monitor script

# Configuration
LOGFILE=~/myps.log

# Signal handlers
handle_sigusr1() {
    # Clean file and notify
    > "$LOGFILE"
    echo "File $LOGFILE cleaned" >&2
}

# Trap signals
trap '' SIGINT SIGTERM          # Ignore Ctrl+C and kill
trap 'handle_sigusr1' SIGUSR1   # Custom handler for SIGUSR1

# Main loop
while true; do
    # Get current date/time
    datetime=$(date '+%Y-%m-%d %H:%M:%S')
    
    # Get number of processes
    process_count=$(ps aux --no-headers | wc -l)
    
    # Get load average (1, 5, 15 minutes)
    load=$(uptime | awk -F'load average:' '{print $2}' | tr -d ' ')
    load1=$(echo $load | cut -d',' -f1)
    load5=$(echo $load | cut -d',' -f2)
    load15=$(echo $load | cut -d',' -f3)
    
    # Log entry
    echo "$datetime,$process_count,$load1,$load5,$load15" >> "$LOGFILE"
    
    # Wait 3 seconds
    sleep 3
done
```

**Make executable:**
```bash
chmod +x ~/monitor.sh
```

**Usage:**

**Start monitoring:**
```bash
~/monitor.sh &
[1] 12345
```

**Check log file:**
```bash
tail ~/myps.log
```

**Output:**
```
2024-12-16 14:30:00,234,0.15,0.18,0.20
2024-12-16 14:30:03,235,0.14,0.17,0.19
2024-12-16 14:30:06,233,0.16,0.18,0.20
```

**Test signal resistance:**
```bash
# Try Ctrl+C (script keeps running)
fg
Ctrl+C  # Ignored!
Ctrl+Z
bg
```

**Clean log file:**
```bash
kill -SIGUSR1 12345
```

**Check stderr:**
```
File /home/student/myps.log cleaned
```

**Stop script:**
```bash
kill -9 12345  # Only force kill works
```

---

### Task 5: Parallel Image Conversion

**Objective:** Convert JPG to PNG concurrently with error handling.

**Solution:**

```bash
vim ~/convert_images.sh
```

**Script content:**
```bash
#!/bin/bash

# Parallel image converter JPG to PNG

VERBOSE=0
TARGET_DIR=""

# Parse options
while [ $# -gt 0 ]; do
    case "$1" in
        -v)
            VERBOSE=1
            shift
            ;;
        *)
            if [ -z "$TARGET_DIR" ]; then
                TARGET_DIR="$1"
            fi
            shift
            ;;
    esac
done

# Check if directory provided
if [ -z "$TARGET_DIR" ]; then
    echo "Usage: $0 [-v] directory" >&2
    exit 1
fi

# Check if directory exists
if [ ! -d "$TARGET_DIR" ]; then
    echo "Error: Directory $TARGET_DIR does not exist" >&2
    exit 1
fi

# Function to convert single image
convert_image() {
    local file="$1"
    local verbose="$2"
    
    # Get base name
    local basename=$(basename "$file" .jpg)
    local dirname=$(dirname "$file")
    local output="$dirname/${basename}.png"
    
    # Check if file is JPEG
    if ! file "$file" | grep -q "JPEG"; then
        echo "Error: $file is not a JPEG image" >&2
        return 1
    fi
    
    # Check if output already exists
    if [ -e "$output" ]; then
        echo "Error: $output already exists, skipping" >&2
        return 1
    fi
    
    # Verbose output
    if [ "$verbose" -eq 1 ]; then
        echo "Converting: $file"
    fi
    
    # Convert
    convert "$file" "$output"
    
    return 0
}

# Start timer if verbose
if [ "$VERBOSE" -eq 1 ]; then
    START_TIME=$(date +%s.%N)
fi

# Find all JPG files and convert concurrently
pids=()
for file in "$TARGET_DIR"/*.jpg "$TARGET_DIR"/*.JPG; do
    # Skip if glob didn't match
    [ -e "$file" ] || continue
    
    # Convert in background
    convert_image "$file" "$VERBOSE" &
    pids+=($!)
done

# Wait for all conversions
for pid in "${pids[@]}"; do
    wait "$pid"
done

# Show timing if verbose
if [ "$VERBOSE" -eq 1 ]; then
    END_TIME=$(date +%s.%N)
    DURATION=$(echo "scale=1; $END_TIME - $START_TIME" | bc)
    echo "Total conversion time: ${DURATION}s"
fi
```

**Make executable:**
```bash
chmod +x ~/convert_images.sh
```

**Usage examples:**

**Setup test images:**
```bash
mkdir ~/test_images
# Upload some JPG files to ~/test_images
```

**Convert without verbose:**
```bash
~/convert_images.sh ~/test_images
```

**Convert with verbose:**
```bash
~/convert_images.sh -v ~/test_images
```

**Output:**
```
Converting: /home/student/test_images/photo1.jpg
Converting: /home/student/test_images/photo2.jpg
Converting: /home/student/test_images/photo3.jpg
Total conversion time: 2.3s
```

**Verbose as second parameter:**
```bash
~/convert_images.sh ~/test_images -v
```

**Error handling:**
```bash
# Try converting non-JPEG
cp /etc/passwd ~/test_images/fake.jpg
~/convert_images.sh ~/test_images
# Output: Error: /home/student/test_images/fake.jpg is not a JPEG image
```

**Try converting again (already exists):**
```bash
~/convert_images.sh ~/test_images
# Output: Error: /home/student/test_images/photo1.png already exists, skipping
```

---

### Task 6: Debug with set -v and set -x

**Objective:** Trace script execution.

**Using set -v:**
```bash
vim ~/convert_debug_v.sh
```

```bash
#!/bin/bash
set -v

# Shows commands as written
TARGET_DIR="$1"
for file in "$TARGET_DIR"/*.jpg; do
    echo "Processing: $file"
done
```

**Run:**
```bash
chmod +x ~/convert_debug_v.sh
~/convert_debug_v.sh ~/test_images
```

**Output shows literal commands:**
```
TARGET_DIR="$1"
for file in "$TARGET_DIR"/*.jpg; do
    echo "Processing: $file"
done
Processing: /home/student/test_images/photo1.jpg
```

---

**Using set -x:**
```bash
vim ~/convert_debug_x.sh
```

```bash
#!/bin/bash
set -x

# Shows expanded commands
TARGET_DIR="$1"
for file in "$TARGET_DIR"/*.jpg; do
    echo "Processing: $file"
done
```

**Run:**
```bash
chmod +x ~/convert_debug_x.sh
~/convert_debug_x.sh ~/test_images
```

**Output shows variable expansion:**
```
+ TARGET_DIR=/home/student/test_images
+ for file in "$TARGET_DIR"/*.jpg
+ echo 'Processing: /home/student/test_images/photo1.jpg'
Processing: /home/student/test_images/photo1.jpg
```

---

### Task 7: Directory Monitor with Signals

**Objective:** Monitor directory for new files, handle multiple signals.

**Solution:**
```bash
vim ~/monitor_dir.sh
```

**Script content:**
```bash
#!/bin/bash

# Directory monitor with signal handling

# Check arguments
if [ $# -lt 1 ]; then
    echo "Usage: $0 directory" >&2
    exit 1
fi

MONITOR_DIR="$1"

# Check if directory exists
if [ ! -d "$MONITOR_DIR" ]; then
    echo "Error: Directory $MONITOR_DIR does not exist" >&2
    exit 1
fi

# Create temporary files
TEMP_LOG=$(mktemp)
TEMP_FILES_LIST=$(mktemp)

echo "Temporary files created:"
echo "Log: $TEMP_LOG"
echo "Files list: $TEMP_FILES_LIST"

# Signal handlers
show_files_list() {
    echo "=== Files List ==="
    cat "$TEMP_FILES_LIST"
    echo "=================="
}

show_log() {
    echo "=== Event Log ==="
    cat "$TEMP_LOG"
    echo "================="
}

cleanup_and_exit() {
    echo "Cleaning up and exiting..."
    rm -f "$TEMP_LOG" "$TEMP_FILES_LIST"
    exit 0
}

# Trap signals
trap 'show_files_list' SIGUSR1
trap 'show_log' SIGUSR2
trap 'cleanup_and_exit' SIGINT SIGTERM

# Initial snapshot
current_files=$(find "$MONITOR_DIR" -type f 2>/dev/null | sort)

echo "Monitoring $MONITOR_DIR for new files..."
echo "PID: $$"
echo "Send SIGUSR1 to see files list: kill -SIGUSR1 $$"
echo "Send SIGUSR2 to see event log: kill -SIGUSR2 $$"
echo "Press Ctrl+C to stop"

# Main monitoring loop
while true; do
    sleep 5
    
    # Get current files
    new_snapshot=$(find "$MONITOR_DIR" -type f 2>/dev/null | sort)
    
    # Find new files
    new_files=$(comm -13 <(echo "$current_files") <(echo "$new_snapshot"))
    
    # Process new files
    if [ -n "$new_files" ]; then
        while IFS= read -r filepath; do
            filename=$(basename "$filepath")
            timestamp=$(date '+%Y-%m-%d %H:%M:%S')
            
            # Add to files list
            echo "$filename" >> "$TEMP_FILES_LIST"
            
            # Log event
            echo "$timestamp: Dodano plik: $filename" >> "$TEMP_LOG"
            
            echo "New file detected: $filename"
        done <<< "$new_files"
    fi
    
    # Update snapshot
    current_files="$new_snapshot"
done
```

**Make executable:**
```bash
chmod +x ~/monitor_dir.sh
```

**Usage:**

**Start monitoring:**
```bash
~/monitor_dir.sh ~/Documents &
```

**Output:**
```
Temporary files created:
Log: /tmp/tmp.Ab12cD
Files list: /tmp/tmp.Xy34Zw
Monitoring /home/student/Documents for new files...
PID: 12345
Send SIGUSR1 to see files list: kill -SIGUSR1 12345
Send SIGUSR2 to see event log: kill -SIGUSR2 12345
Press Ctrl+C to stop
[1] 12345
```

**Create new files:**
```bash
touch ~/Documents/test1.txt
sleep 6
touch ~/Documents/test2.txt
```

**View files list:**
```bash
kill -SIGUSR1 12345
```

**Output:**
```
=== Files List ===
test1.txt
test2.txt
==================
```

**View event log:**
```bash
kill -SIGUSR2 12345
```

**Output:**
```
=== Event Log ===
2024-12-16 14:35:00: Dodano plik: test1.txt
2024-12-16 14:35:06: Dodano plik: test2.txt
=================
```

**Stop monitoring:**
```bash
kill 12345
# Or bring to foreground and Ctrl+C
fg
Ctrl+C
```

**Output:**
```
Cleaning up and exiting...
```

---

## Summary Questions

### Question 1: Detecting New Files in Directory

**How to check which new files appeared in directory since last check?**

**Method 1: Using find with timestamp comparison**
```bash
# Store initial snapshot
initial_files=$(find /path/to/dir -type f)

# ... time passes ...

# Get current snapshot
current_files=$(find /path/to/dir -type f)

# Find new files
new_files=$(comm -13 <(echo "$initial_files" | sort) <(echo "$current_files" | sort))
```

**Method 2: Using find with -newer**
```bash
# Create reference file
touch /tmp/reference

# ... time passes ...

# Find files newer than reference
find /path/to/dir -type f -newer /tmp/reference
```

**Method 3: Store and compare file lists**
```bash
# Initial
ls /path/to/dir > /tmp/before.txt

# ... time passes ...

# Compare
ls /path/to/dir > /tmp/after.txt
diff /tmp/before.txt /tmp/after.txt
```

**Method 4: Using comm (most reliable)**
```bash
#!/bin/bash

DIR="/path/to/monitor"
prev_files=$(mktemp)
curr_files=$(mktemp)

# Initial snapshot
find "$DIR" -type f | sort > "$prev_files"

while true; do
    sleep 5
    
    # Current snapshot
    find "$DIR" -type f | sort > "$curr_files"
    
    # Find new files (in curr but not in prev)
    new=$(comm -13 "$prev_files" "$curr_files")
    
    if [ -n "$new" ]; then
        echo "New files:"
        echo "$new"
    fi
    
    # Update
    cp "$curr_files" "$prev_files"
done
```

---

### Question 2: Difference Between set -v and set -x

**What's the practical difference? What do they change?**

**set -v (Verbose):**
- **Shows:** Commands as written in script
- **When:** Before execution
- **Use:** Understanding script flow and logic

**Example:**
```bash
#!/bin/bash
set -v

name="Alice"
echo "Hello, $name"
```

**Output:**
```
name="Alice"
echo "Hello, $name"
Hello, Alice
```

---

**set -x (Execution trace):**
- **Shows:** Commands with variable expansion
- **When:** Before execution
- **Use:** Debugging variable values and command results

**Example:**
```bash
#!/bin/bash
set -x

name="Alice"
echo "Hello, $name"
```

**Output:**
```
+ name=Alice
+ echo 'Hello, Alice'
Hello, Alice
```

---

**Side-by-side comparison:**

```bash
#!/bin/bash

count=5
files="file1.txt file2.txt"

echo "=== With set -v ==="
set -v
for file in $files; do
    echo "Processing: $file"
done
set +v

echo ""
echo "=== With set -x ==="
set -x
for file in $files; do
    echo "Processing: $file"
done
set +x
```

**Output:**
```
=== With set -v ===
for file in $files; do
    echo "Processing: $file"
done
Processing: file1.txt
Processing: file2.txt

=== With set -x ===
+ for file in $files
+ echo 'Processing: file1.txt'
Processing: file1.txt
+ for file in $files
+ echo 'Processing: file2.txt'
Processing: file2.txt
```

**Summary table:**

| Feature | set -v | set -x |
|---------|--------|--------|
| Shows variables | No (literal) | Yes (expanded) |
| Shows iteration | Shows loop construct | Shows each iteration |
| Prefix | None | `+` symbol |
| Best for | Understanding structure | Debugging values |
| Noise level | Lower | Higher |

**When to use:**
- **set -v:** Script logic unclear, want to see flow
- **set -x:** Variables have wrong values, need to see substitution

---

## Key Takeaways

1. **SSH keys** enable secure, password-less authentication
2. **Background processing** with `&` and `wait` enables parallelism
3. **trap** catches signals for cleanup and custom behavior
4. **SIGINT/SIGTERM** can be ignored for critical operations
5. **SIGUSR1/SIGUSR2** provide custom signal handlers
6. **bc** enables floating-point arithmetic
7. **set -v** shows literal commands, **set -x** shows expanded commands
8. **mktemp** creates secure temporary files
9. **Parallel processing** speeds up bulk operations significantly
10. **Signal handling** makes scripts robust and controllable

Advanced BASH scripting enables professional-grade automation!
