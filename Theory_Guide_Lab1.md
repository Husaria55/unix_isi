# Theory Guide - Lab 1: Introduction to Unix Systems

## Table of Contents
1. [Introduction to Unix Philosophy](#introduction-to-unix-philosophy)
2. [The Shell and Terminal](#the-shell-and-terminal)
3. [Remote Access via SSH](#remote-access-via-ssh)
4. [Documentation Systems](#documentation-systems)
5. [File System Hierarchy](#file-system-hierarchy)
6. [File and Directory Operations](#file-and-directory-operations)
7. [Text Processing Basics](#text-processing-basics)
8. [Pipes and Redirection](#pipes-and-redirection)
9. [Command Reference](#command-reference)
10. [Important System Files](#important-system-files)

---

## Introduction to Unix Philosophy

The Unix philosophy is built on several key principles that guide how Unix systems and tools are designed:

1. **Write programs that do one thing and do it well** - Each command should have a single, well-defined purpose
2. **Write programs that work together** - Commands should be composable through pipes and redirection
3. **Write programs that handle text streams** - Text is the universal interface between programs
4. **Everything is a file** - Devices, processes, and system resources are represented as files

This philosophy explains why Unix commands are often short, focused utilities that can be combined in powerful ways.

---

## The Shell and Terminal

### What is a Shell?

A **shell** is a command-line interpreter that:
- Accepts commands from the user
- Interprets and executes those commands
- Provides access to the operating system's services

The most common shell on Linux systems is **bash** (Bourne Again Shell).

### Terminal vs Shell

- **Terminal**: The application window that displays text input/output
- **Shell**: The program running inside the terminal that interprets commands

### Important Key Combinations

| Combination | Purpose | Effect |
|-------------|---------|--------|
| **CTRL + D** | End of File (EOF) | Signals end of input; exits shell if typed at empty prompt |
| **CTRL + C** | Interrupt | Sends SIGINT signal to terminate the running program |
| **CTRL + Z** | Suspend | Pauses the current program (can be resumed with `fg`) |
| **CTRL + L** | Clear screen | Clears the terminal display |

---

## Remote Access via SSH

### What is SSH?

**SSH (Secure Shell)** is a cryptographic network protocol for securely accessing remote computers over an unsecured network.

### Connecting via SSH

**Linux/MacOS:**
```bash
ssh username@hostname
```

**Windows:**
- Use PuTTY (GUI application)
- Or use Windows PowerShell/Command Prompt (Windows 10+): `ssh username@hostname`

### Example:
```bash
ssh student@lab.kis.agh.edu.pl
```

### Authentication
SSH typically uses:
1. **Password authentication** - You enter your password
2. **Key-based authentication** - Uses cryptographic keys (more secure)

---

## Documentation Systems

Unix provides multiple levels of documentation access.

### 1. `man` - Manual Pages

The **manual pages** (man pages) are the primary documentation system in Unix.

**Structure:**
Man pages are organized into sections:
- **Section 1**: User commands
- **Section 2**: System calls
- **Section 3**: Library functions
- **Section 4**: Special files (devices)
- **Section 5**: File formats and conventions
- **Section 6**: Games
- **Section 7**: Miscellaneous
- **Section 8**: System administration commands

**Usage:**
```bash
man command_name         # Opens man page for command
man section command_name # Opens specific section
man -a command_name      # View all available manual pages sequentially
```

**Navigation within man:**
- **Space** or **Page Down**: Next page
- **Page Up**: Previous page
- **/pattern**: Search forward for pattern
- **?pattern**: Search backward for pattern
- **n**: Next match
- **N**: Previous match
- **q**: Quit

### 2. `whatis` - Brief Description

Displays a one-line description from the man page.

```bash
whatis ls
# Output: ls (1) - list directory contents
```

**Use case**: Quickly understand what a command does without opening full documentation.

### 3. `apropos` - Search Man Pages

Searches man page descriptions for keywords.

```bash
apropos keyword
```

**Example:**
```bash
apropos authentication
# Returns all commands related to authentication
```

**How it works**: Searches the NAME section of all man pages for the keyword.

**Pro tip**: `apropos` output can be lengthy, so pipe it to a pager:
```bash
apropos authentication | less
```

---

## File System Hierarchy

### Understanding Paths

**Absolute Path**: Starts from the root directory (`/`)
```bash
/home/student/documents/file.txt
```

**Relative Path**: Relative to current working directory
```bash
documents/file.txt
./documents/file.txt
```

### Special Path Notations

| Notation | Meaning |
|----------|---------|
| `/` | Root directory (top of file system) |
| `~` | Current user's home directory |
| `~username` | Specified user's home directory |
| `.` | Current directory |
| `..` | Parent directory |
| `-` | Previous working directory |

### Tilde Expansion

The shell performs **tilde expansion** before executing commands:
- `~` expands to `/home/your_username`
- `~wojnicki` expands to `/home/wojnicki`
- This is a shell feature, not part of the command itself

**Example:**
```bash
cd ~/documents
# Expands to: cd /home/your_username/documents
```

### Important Directories

| Directory | Purpose |
|-----------|---------|
| `/home` | User home directories |
| `/etc` | System configuration files |
| `/tmp` | Temporary files (often cleared on reboot) |
| `/usr` | User programs and data |
| `/var` | Variable data (logs, caches) |
| `/bin` | Essential user binaries |
| `/sbin` | System binaries (admin commands) |

---

## File and Directory Operations

### Navigating the File System

#### `pwd` - Print Working Directory

Displays the absolute path of the current directory.

```bash
pwd
# Output: /home/student
```

**Why it's useful**: Always know where you are in the file system.

#### `cd` - Change Directory

Changes the current working directory.

**Syntax:**
```bash
cd [directory]
```

**Common uses:**
```bash
cd /etc              # Absolute path
cd documents         # Relative path
cd ..                # Parent directory
cd                   # Home directory (no argument)
cd ~                 # Home directory (explicit)
cd -                 # Previous directory
cd ~/projects/lab1   # Using tilde expansion
```

### Listing Files

#### `ls` - List Directory Contents

**Basic syntax:**
```bash
ls [OPTIONS] [PATH]
```

**Essential options:**

| Option | Description |
|--------|-------------|
| `-l` | Long format (detailed information) |
| `-a` | Show all files (including hidden files starting with `.`) |
| `-h` | Human-readable sizes (with `-l`) |
| `-R` | Recursive listing (all subdirectories) |
| `-t` | Sort by modification time |
| `-r` | Reverse order |
| `-d` | List directories themselves, not contents |
| `-1` | One file per line |

**Long format (`ls -l`) output:**
```
-rw-r--r--  1 student users  1234 Dec 16 10:30 file.txt
```

Fields from left to right:
1. **File type and permissions**: `-rw-r--r--`
2. **Number of hard links**: `1`
3. **Owner**: `student`
4. **Group**: `users`
5. **Size in bytes**: `1234`
6. **Modification date**: `Dec 16 10:30`
7. **File name**: `file.txt`

**Examples:**
```bash
ls                    # Current directory
ls /home              # Specific directory
ls -la                # All files in long format
ls -lh                # Long format with human-readable sizes
ls -R ~               # Recursive listing of home directory
```

### Creating Directories

#### `mkdir` - Make Directory

Creates new directories.

**Syntax:**
```bash
mkdir [OPTIONS] directory_name
```

**Key options:**

| Option | Description |
|--------|-------------|
| `-p` | Create parent directories as needed (no error if exists) |
| `-v` | Verbose output |
| `-m mode` | Set permissions |

**Examples:**
```bash
mkdir lab1                    # Simple directory creation
mkdir -p ~/lab1/dziennik      # Create nested directories
mkdir -p path/to/deep/dir     # Create entire path at once
mkdir dir1 dir2 dir3          # Create multiple directories
```

**Why `-p` is important**: Without `-p`, `mkdir ~/lab1/dziennik` fails if `lab1` doesn't exist. With `-p`, both directories are created in one command.

#### `rmdir` - Remove Directory

Removes **empty** directories only.

```bash
rmdir directory_name
```

**Limitation**: Only works on empty directories. For non-empty directories, use `rm -r`.

### Copying Files

#### `cp` - Copy Files and Directories

**Syntax:**
```bash
cp [OPTIONS] source destination
```

**Essential options:**

| Option | Description |
|--------|-------------|
| `-r` or `-R` | Copy directories recursively |
| `-i` | Interactive mode (prompt before overwrite) |
| `-v` | Verbose output |
| `-p` | Preserve permissions, timestamps, ownership |
| `-a` | Archive mode (recursive + preserve everything) |
| `-u` | Copy only when source is newer |

**Common patterns:**

```bash
# Copy file to directory
cp /etc/passwd ~/tmp

# Copy file with new name
cp /etc/passwd ~/tmp/passwd.backup

# Copy multiple files to directory
cp file1 file2 file3 destination_dir/

# Copy directory recursively
cp -r source_dir/ destination_dir/

# Interactive copy (prompts before overwriting)
cp -i source dest

# Archive copy (preserves everything)
cp -a source_dir/ backup_dir/
```

**Important behavior:**
- If destination is an existing directory, the file is copied **into** it
- If destination doesn't exist, it becomes the new filename
- **Without `-i`, cp overwrites existing files without warning**

### Moving and Renaming Files

#### `mv` - Move or Rename

**Syntax:**
```bash
mv [OPTIONS] source destination
```

**Key options:**

| Option | Description |
|--------|-------------|
| `-i` | Interactive mode (prompt before overwrite) |
| `-v` | Verbose output |
| `-n` | Don't overwrite existing files |
| `-u` | Move only when source is newer |

**Common patterns:**

```bash
# Rename a file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt ~/documents/

# Move multiple files to directory
mv file1 file2 file3 destination_dir/

# Rename directory
mv old_dir_name new_dir_name

# Interactive move
mv -i source dest
```

**Key difference from cp:**
- `mv` doesn't need `-r` for directories
- `mv` removes the source after copying (it's a true move, not a copy)

### Deleting Files and Directories

#### `rm` - Remove Files and Directories

**Syntax:**
```bash
rm [OPTIONS] file_or_directory
```

**Essential options:**

| Option | Description |
|--------|-------------|
| `-r` or `-R` | Remove directories and contents recursively |
| `-f` | Force removal (ignore nonexistent files, no prompts) |
| `-i` | Interactive mode (prompt for each file) |
| `-v` | Verbose output |
| `-d` | Remove empty directories |

**Common patterns:**

```bash
# Remove single file
rm file.txt

# Remove multiple files
rm file1.txt file2.txt file3.txt

# Remove directory and all contents
rm -r directory_name/

# Force remove without prompts
rm -rf directory_name/

# Interactive removal
rm -i *.txt
```

**⚠️ CRITICAL SAFETY WARNING:**
- There is **NO UNDO** for `rm`
- Deleted files **CANNOT be recovered** (no "Recycle Bin")
- `rm -rf /` could delete the entire system
- **ALWAYS double-check before using `rm`, especially with `-r` and `-f`**
- Consider using `rm -i` when learning

**Permissions matter:**
- You can only remove files you have permission to delete
- You cannot remove files in directories where you lack write permission
- System directories (like `/tmp`) have special permissions

---

## Text Processing Basics

### Viewing Text Files

#### `more` - Simple Pager

Views text files one screen at a time (older, simpler pager).

**Controls:**
- **Space**: Next page
- **Enter**: Next line
- **q**: Quit

```bash
more filename.txt
```

**Limitation**: Cannot scroll backward.

#### `less` - Enhanced Pager

More powerful pager with bidirectional scrolling.

**Controls:**
- **Space** or **Page Down**: Next page
- **Page Up** or **b**: Previous page
- **/pattern**: Search forward
- **?pattern**: Search backward
- **n**: Next match
- **N**: Previous match
- **g**: Go to beginning
- **G**: Go to end
- **q**: Quit

```bash
less filename.txt
```

**Pro tip**: `less` is "more" with more features. Use `less` for most file viewing.

#### Choosing a Pager

- **Use `less`** for: Interactive viewing, searching, navigating large files
- **Use `more`** for: Quick, simple viewing when `less` isn't available
- **Use `cat`** for: Displaying entire file to terminal (or piping to other commands)

### Counting Text

#### `wc` - Word Count

Counts lines, words, and bytes in files.

**Syntax:**
```bash
wc [OPTIONS] [FILE]
```

**Options:**

| Option | Description |
|--------|-------------|
| `-l` | Count lines only |
| `-w` | Count words only |
| `-c` | Count bytes only |
| `-m` | Count characters only |
| `-L` | Print length of longest line |

**Output format (default):**
```bash
wc file.txt
# Output: 42 158 1024 file.txt
#         lines words bytes filename
```

**Examples:**
```bash
wc file.txt              # All counts
wc -l file.txt           # Line count only
ls /home | wc -l         # Count items from command output
```

**Practical uses:**
- Count log entries
- Verify file lengths
- Count users, processes, etc.

### Displaying Text

#### `echo` - Display Text

Prints arguments to standard output.

**Syntax:**
```bash
echo [OPTIONS] [STRING]
```

**Examples:**
```bash
echo "Hello World"       # Prints: Hello World
echo $HOME               # Prints value of HOME variable
echo *                   # Prints all files in current directory
echo -n "No newline"     # Suppress trailing newline
```

---

## Pipes and Redirection

### Understanding Standard Streams

Every Unix process has three standard streams:

1. **Standard Input (stdin)** - File descriptor 0 - Where input comes from
2. **Standard Output (stdout)** - File descriptor 1 - Where normal output goes
3. **Standard Error (stderr)** - File descriptor 2 - Where error messages go

### The Pipe Operator: `|`

**Purpose**: Connect the stdout of one command to the stdin of another.

**Syntax:**
```bash
command1 | command2
```

**How it works:**
1. `command1` executes and produces output
2. That output becomes the input for `command2`
3. Both commands run simultaneously (not sequentially)

**Examples:**

```bash
# Page through long output
ls -la /etc | less

# Count number of files
ls | wc -l

# Search through command output
apropos authentication | grep password

# Chain multiple commands
cat /etc/passwd | grep student | wc -l
```

**Practical applications:**
- Filter command output
- Process data through multiple stages
- Build complex data pipelines from simple commands

**The Unix Philosophy in Action**: Pipes embody the principle of combining simple, specialized tools into powerful workflows.

---

## Command Reference

### `passwd` - Change Password

Changes user password.

```bash
passwd           # Change your own password
passwd username  # Change another user's password (requires root)
```

### `date` - Display or Set Date/Time

**Syntax:**
```bash
date [OPTIONS] [+FORMAT]
```

**Common format specifiers:**

| Specifier | Description |
|-----------|-------------|
| `%Y` | Year (4 digits) |
| `%m` | Month (01-12) |
| `%d` | Day of month (01-31) |
| `%H` | Hour (00-23) |
| `%M` | Minute (00-59) |
| `%S` | Second (00-59) |
| `%A` | Full weekday name |
| `%B` | Full month name |

**Examples:**
```bash
date                    # Full date and time
date +%Y-%m-%d          # 2025-12-16
date +%H:%M:%S          # 10:30:45
date +%H:%M             # 10:30 (time without seconds)
date "+%Y-%m-%d %H:%M"  # 2025-12-16 10:30
```

---

## Important System Files

### `/etc/passwd`

The **password file** contains user account information.

**Format**: Each line represents one user account:
```
username:x:UID:GID:GECOS:home_directory:shell
```

**Field breakdown:**

1. **Username**: Login name
2. **Password placeholder**: `x` (actual passwords in `/etc/shadow`)
3. **UID**: User ID number (unique identifier)
4. **GID**: Primary Group ID number
5. **GECOS**: User information (full name, phone, etc.)
6. **Home directory**: Path to user's home directory
7. **Shell**: Default shell for the user

**Example line:**
```
student:x:1001:1001:John Smith,Room 123,555-1234:/home/student:/bin/bash
```

**Why it's important:**
- Essential for user authentication and authorization
- Readable by all users (but not writable)
- Used by many system services
- Historical: "passwd" because it originally stored encrypted passwords

### `/etc/passwd-`

Backup of `/etc/passwd` created before the last modification. Used for system recovery.

---

## Best Practices and Tips

### 1. Always Use Tab Completion
Press **Tab** to auto-complete filenames, commands, and paths. Saves time and prevents typos.

### 2. Use `-i` When Learning
Interactive mode (`-i` flag with `cp`, `mv`, `rm`) prompts before destructive operations. Remove it once confident.

### 3. Read the Manual
When unsure, always check: `man command_name`

### 4. Test Destructive Commands First
Before running `rm -r`, try `ls` to verify you're targeting the right files.

### 5. Understand Paths
Always be aware of your current directory (`pwd`) and use absolute paths when necessary.

### 6. Combine Commands
Use pipes (`|`) to build efficient workflows instead of creating temporary files.

### 7. Use `less` for Reading
When viewing files or long command output, pipe to `less` for comfortable navigation.

---

## Common Pitfalls

### 1. Case Sensitivity
Unix is **case-sensitive**: `file.txt`, `File.txt`, and `FILE.TXT` are different files.

### 2. Spaces in Arguments
Spaces separate arguments. Use quotes for filenames with spaces:
```bash
mkdir "my documents"    # Correct
mkdir my documents      # Wrong: creates 'my' and 'documents'
```

### 3. Overwriting Files
`cp` and `mv` overwrite without warning by default. Use `-i` when unsure.

### 4. No Undo for rm
Think twice, execute once. Deleted files are gone forever.

### 5. Permission Denied
If you get "Permission denied", check file permissions and ownership. You might need sudo (if authorized).

---

## Summary

This lab introduces fundamental Unix operations:

- **Navigation**: `pwd`, `cd`, `ls`
- **Documentation**: `man`, `apropos`, `whatis`
- **File operations**: `cp`, `mv`, `rm`, `mkdir`, `rmdir`
- **Text viewing**: `more`, `less`, `wc`, `echo`
- **Combining commands**: Pipes (`|`)
- **System interaction**: `date`, `passwd`, SSH

Mastering these basics provides the foundation for all advanced Unix work. The key is understanding that simple, composable tools following the Unix philosophy create powerful, flexible systems.
