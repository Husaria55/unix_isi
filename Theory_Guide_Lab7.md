# Theory Guide - Lab 7: Advanced BASH - SSH, Parallel Processing, Signals

## Table of Contents
1. [SSH Key Authentication](#ssh-key-authentication)
2. [Parallel Processing in BASH](#parallel-processing-in-bash)
3. [Signal Handling](#signal-handling)
4. [Debugging BASH Scripts](#debugging-bash-scripts)
5. [Command Reference](#command-reference)

---

## SSH Key Authentication

### What is SSH Key Authentication?

**SSH keys** provide secure, password-less authentication to remote servers.

**Components:**
- **Private key** - Kept secret on your computer
- **Public key** - Shared with servers you want to access

**Analogy:** Like a lock (public key) and key (private key) - only your private key opens the lock.

### Generating SSH Keys

**Command:**
```bash
ssh-keygen
```

**Interactive prompts:**
```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa): [Press Enter]
Enter passphrase (empty for no passphrase): [Press Enter or type password]
Enter same passphrase again: [Press Enter or same password]
```

**Output:**
```
Your identification has been saved in /home/user/.ssh/id_rsa
Your public key has been saved in /home/user/.ssh/id_rsa.pub
```

**Key types:**
| Type | Security | Speed | Command |
|------|----------|-------|---------|
| RSA | Good | Fast | `ssh-keygen -t rsa -b 4096` |
| ED25519 | Best | Fastest | `ssh-keygen -t ed25519` |

### Key Files

**Default locations:**
- **Private key:** `~/.ssh/id_rsa` (or `id_ed25519`)
- **Public key:** `~/.ssh/id_rsa.pub` (or `id_ed25519.pub`)

**⚠️ Important:** Never share your private key!

### Copying Public Key to Server

**Method 1: Using ssh (manual):**
```bash
# Display your public key
cat ~/.ssh/id_rsa.pub

# SSH to server
ssh user@server

# Create .ssh directory if needed
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add public key (paste the key from your local machine)
vim ~/.ssh/authorized_keys
# Paste the public key, save and exit

# Set permissions
chmod 600 ~/.ssh/authorized_keys
```

**Method 2: Using ssh-copy-id (easier):**
```bash
ssh-copy-id user@server
```

**Method 3: Manual with redirection:**
```bash
cat ~/.ssh/id_rsa.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### Testing Key Authentication

**First attempt (should ask for password):**
```bash
ssh user@server
# Enter password
```

**After copying key (no password):**
```bash
ssh user@server
# Connected without password!
```

---

## Parallel Processing in BASH

### Background Processes

**Syntax:**
```bash
command &
```

**Example:**
```bash
sleep 10 &
echo "Sleep is running in background"
```

### Running Multiple Processes Concurrently

**Sequential (slow):**
```bash
process1
process2
process3
# Total time: time1 + time2 + time3
```

**Parallel (fast):**
```bash
process1 &
process2 &
process3 &
wait
# Total time: max(time1, time2, time3)
```

### The `wait` Command

**Purpose:** Wait for background processes to complete.

**Syntax:**
```bash
wait [PID]
```

**Examples:**

**Wait for all background processes:**
```bash
sleep 5 &
sleep 3 &
echo "Waiting..."
wait
echo "All done!"
```

**Wait for specific process:**
```bash
sleep 10 &
pid=$!
echo "Started process $pid"
wait $pid
echo "Process $pid finished"
```

**Practical example:**
```bash
#!/bin/bash

# Start multiple downloads concurrently
wget url1 &
wget url2 &
wget url3 &

# Wait for all to complete
wait

echo "All downloads complete"
```

---

## Signal Handling

### The `trap` Command

**Purpose:** Catch signals and execute commands.

**Syntax:**
```bash
trap 'commands' SIGNAL
```

**Common signals:**
| Signal | Number | Trigger | Default Action |
|--------|--------|---------|----------------|
| SIGINT | 2 | Ctrl+C | Terminate |
| SIGTERM | 15 | `kill` | Terminate |
| SIGUSR1 | 10 | User-defined | Terminate |
| SIGUSR2 | 12 | User-defined | Terminate |
| SIGTSTP | 20 | Ctrl+Z | Stop |

### Ignoring Signals

**Ignore Ctrl+C:**
```bash
#!/bin/bash

trap '' SIGINT

echo "Try pressing Ctrl+C - it won't work!"
sleep 10
echo "Done!"
```

**Restore default behavior:**
```bash
trap - SIGINT
```

### Custom Signal Handlers

**Example:**
```bash
#!/bin/bash

cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
    exit 0
}

# Trap SIGINT and SIGTERM
trap cleanup SIGINT SIGTERM

echo "Running... Press Ctrl+C to stop"
while true; do
    echo "Working..."
    sleep 1
done
```

**Multiple signals, one handler:**
```bash
trap 'echo "Caught signal!"; exit 1' SIGINT SIGTERM SIGHUP
```

### Common trap Patterns

**Cleanup on exit:**
```bash
#!/bin/bash

tempfile=$(mktemp)

cleanup() {
    rm -f "$tempfile"
}

trap cleanup EXIT

# Script continues...
```

**Prevent interruption:**
```bash
trap '' SIGINT SIGTERM
critical_operation
trap - SIGINT SIGTERM
```

---

## Debugging BASH Scripts

### `set -v` (Verbose Mode)

**Effect:** Print commands before executing them (exactly as written).

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

### `set -x` (Execution Trace)

**Effect:** Print commands with variable expansion before executing.

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

### Comparison: `set -v` vs `set -x`

```bash
#!/bin/bash

name="Alice"
count=5

echo "=== With set -v ==="
set -v
echo "Hello, $name"
echo "Count: $count"
set +v

echo ""
echo "=== With set -x ==="
set -x
echo "Hello, $name"
echo "Count: $count"
set +x
```

**Output:**
```
=== With set -v ===
echo "Hello, $name"
Hello, Alice
echo "Count: $count"
Count: 5

=== With set -x ===
+ echo 'Hello, Alice'
Hello, Alice
+ echo 'Count: 5'
Count: 5
```

**Summary:**
| Mode | Shows | Use When |
|------|-------|----------|
| `set -v` | Literal commands | Understanding script flow |
| `set -x` | Expanded commands | Debugging variable values |

### Turning Debug Mode Off

```bash
set +v    # Disable verbose
set +x    # Disable trace
```

### Debug Specific Sections

```bash
#!/bin/bash

# Normal execution
echo "Starting..."

# Enable debugging for specific section
set -x
for i in {1..3}; do
    echo "Number: $i"
done
set +x

# Back to normal
echo "Done"
```

---

## Additional Commands

### The `bc` Command

**Purpose:** Arbitrary precision calculator.

**Basic usage:**
```bash
echo "2 + 3" | bc
# Output: 5

echo "10 / 3" | bc
# Output: 3 (integer division)
```

**Floating point:**
```bash
echo "scale=2; 10 / 3" | bc
# Output: 3.33

echo "scale=4; 22 / 7" | bc
# Output: 3.1428
```

**In scripts:**
```bash
result=$(echo "scale=2; $1 * $2" | bc)
echo "Result: $result"
```

**Common operations:**
```bash
# Power
echo "2^10" | bc
# Output: 1024

# Square root
echo "sqrt(16)" | bc
# Output: 4

# Complex expression
echo "scale=2; (5 + 3) * 2 / 4" | bc
# Output: 4.00
```

### The `convert` Command

**Purpose:** Image conversion (from ImageMagick).

**Basic usage:**
```bash
convert input.jpg output.png
```

**Common operations:**
```bash
# Convert format
convert image.jpg image.png

# Resize
convert image.jpg -resize 800x600 small.jpg

# Rotate
convert image.jpg -rotate 90 rotated.jpg

# Quality
convert image.jpg -quality 85 compressed.jpg
```

**Batch conversion:**
```bash
for img in *.jpg; do
    convert "$img" "${img%.jpg}.png"
done
```

### The `uptime` Command

**Purpose:** Show system uptime and load average.

```bash
uptime
# Output: 14:30:45 up 10 days, 3:42, 5 users, load average: 0.15, 0.18, 0.20
```

**Parsing uptime:**
```bash
# Get load average
uptime | awk -F'load average:' '{print $2}'

# Get number of users
uptime | awk -F',' '{print $4}' | awk '{print $1}'
```

### Temporary Files with `mktemp`

**Purpose:** Create temporary files safely.

**Basic usage:**
```bash
tempfile=$(mktemp)
echo "data" > "$tempfile"
# Use tempfile
rm "$tempfile"
```

**Temporary directory:**
```bash
tempdir=$(mktemp -d)
cd "$tempdir"
# Do work
rm -rf "$tempdir"
```

**With custom template:**
```bash
tempfile=$(mktemp /tmp/myapp.XXXXXX)
```

---

## Command Reference

### SSH Keys

```bash
ssh-keygen                    # Generate key pair
ssh-keygen -t ed25519        # Generate ED25519 key
cat ~/.ssh/id_rsa.pub        # View public key
ssh-copy-id user@server      # Copy key to server
ssh user@server              # Connect with key
```

### Background Processing

```bash
command &                     # Run in background
jobs                          # List background jobs
wait                          # Wait for all background jobs
wait $PID                     # Wait for specific process
$!                            # PID of last background process
```

### Signal Handling

```bash
trap 'commands' SIGNAL        # Catch signal
trap 'cleanup' SIGINT         # Catch Ctrl+C
trap '' SIGINT                # Ignore signal
trap - SIGINT                 # Restore default
kill -SIGUSR1 $PID           # Send custom signal
```

### Debugging

```bash
set -v                        # Verbose mode
set -x                        # Execution trace
set +v                        # Disable verbose
set +x                        # Disable trace
bash -x script.sh            # Run with tracing
```

### Utilities

```bash
bc                            # Calculator
echo "2+3" | bc              # Simple calculation
echo "scale=2; 10/3" | bc    # Floating point
convert img.jpg img.png      # Image conversion
uptime                        # System uptime
mktemp                        # Create temp file
mktemp -d                     # Create temp directory
```

---

## Best Practices

### 1. Always Use `wait` with Background Jobs

```bash
# Good
process1 &
process2 &
wait
echo "All done"

# Bad - script might exit before processes finish
process1 &
process2 &
echo "All done"  # Might print before processes finish
```

### 2. Cleanup with trap

```bash
#!/bin/bash

tempfile=$(mktemp)
trap "rm -f $tempfile" EXIT

# Script continues...
# Tempfile automatically cleaned up
```

### 3. Quote Variables in trap

```bash
# Good
file="/tmp/my file.txt"
trap "rm -f '$file'" EXIT

# Bad - breaks with spaces
trap "rm -f $file" EXIT
```

### 4. Check Background Process Status

```bash
command &
pid=$!

if wait $pid; then
    echo "Success"
else
    echo "Failed with exit code $?"
fi
```

### 5. Use `bc` for Floating Point

```bash
# Don't use bash arithmetic for decimals
result=$((10 / 3))  # Result: 3 (truncated)

# Use bc
result=$(echo "scale=2; 10 / 3" | bc)  # Result: 3.33
```

---

## Common Patterns

### Parallel Processing with Error Handling

```bash
#!/bin/bash

pids=()

for item in "${items[@]}"; do
    process_item "$item" &
    pids+=($!)
done

failed=0
for pid in "${pids[@]}"; do
    if ! wait "$pid"; then
        failed=$((failed + 1))
    fi
done

if [ $failed -gt 0 ]; then
    echo "$failed processes failed"
    exit 1
fi
```

### Signal-Safe Logging

```bash
#!/bin/bash

LOGFILE="myapp.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$LOGFILE"
}

cleanup() {
    log "Received signal, cleaning up..."
    rm -f /tmp/tempfiles.*
    exit 0
}

trap cleanup SIGINT SIGTERM

log "Application started"
# Main script...
```

### Conditional Debugging

```bash
#!/bin/bash

DEBUG=${DEBUG:-0}

if [ "$DEBUG" -eq 1 ]; then
    set -x
fi

# Script continues...
```

**Usage:**
```bash
./script.sh        # Normal
DEBUG=1 ./script.sh  # With debugging
```

---

## Summary

Lab 7 covers advanced BASH techniques:

**SSH Keys:**
- `ssh-keygen` - Generate key pair
- Public/private key authentication
- Password-less login

**Parallel Processing:**
- `&` - Run in background
- `wait` - Wait for completion
- `$!` - Last background PID

**Signal Handling:**
- `trap` - Catch signals
- SIGINT, SIGTERM, SIGUSR1, SIGUSR2
- Custom handlers and cleanup

**Debugging:**
- `set -v` - Show commands as written
- `set -x` - Show expanded commands
- Selective debugging

**Utilities:**
- `bc` - Floating point calculator
- `convert` - Image conversion
- `uptime` - System load
- `mktemp` - Temporary files

**Key Principle:** Advanced scripting enables robust, parallel, and secure automation.
