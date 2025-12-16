# Theory Guide - Lab 3: Advanced Data Processing and File Searching

## Table of Contents
1. [Introduction to Advanced Processing](#introduction-to-advanced-processing)
2. [Error Redirection](#error-redirection)
3. [Pattern Matching with grep](#pattern-matching-with-grep)
4. [File Searching with find](#file-searching-with-find)
5. [File Type Detection](#file-type-detection)
6. [Command Execution Tools](#command-execution-tools)
7. [Text Manipulation Tools](#text-manipulation-tools)
8. [Command Reference](#command-reference)

---

## Introduction to Advanced Processing

Lab 3 builds on previous skills by introducing:
- **Pattern searching** within files (`grep`)
- **File system searching** (`find`)
- **Error handling** (stderr redirection)
- **Batch operations** (`xargs`, `find -exec`)
- **File type detection** (`file`)
- **Output duplication** (`tee`)

These tools enable sophisticated data processing and system management tasks.

---

## Error Redirection

### Understanding File Descriptors

Every process has three standard file descriptors:

| Descriptor | Name | Number | Purpose |
|------------|------|--------|---------|
| stdin | Standard Input | 0 | Input data |
| stdout | Standard Output | 1 | Normal output |
| stderr | Standard Error | 2 | Error messages |

### Why Separate Error Output?

**Problem:** Errors and normal output mixed together:
```bash
ls existing_file non_existing_file
# Mixed output makes parsing difficult
```

**Solution:** Redirect errors separately:
```bash
ls existing_file non_existing_file 2> errors.txt
# Only normal output shown; errors in errors.txt
```

### Error Redirection Syntax

#### `2>` - Redirect stderr (Overwrite)

**Syntax:**
```bash
command 2> error_file
```

**Example:**
```bash
find / -name "*.txt" 2> /dev/null
# Shows results; hides "Permission denied" errors
```

#### `2>>` - Redirect stderr (Append)

**Syntax:**
```bash
command 2>> error_log
```

**Example:**
```bash
./script.sh 2>> errors.log
# Appends errors to log file
```

### Combined Redirections

#### Separate stdout and stderr

```bash
command > output.txt 2> errors.txt
```

**Example:**
```bash
gcc program.c > compilation_output.txt 2> compilation_errors.txt
```

#### Redirect both to same file

**Modern syntax:**
```bash
command &> all_output.txt
command &>> all_output.txt  # Append
```

**Traditional syntax:**
```bash
command > all_output.txt 2>&1
command >> all_output.txt 2>&1  # Append
```

**Order matters with traditional syntax:**
```bash
# Correct:
command > file 2>&1         # Both to file

# Wrong:
command 2>&1 > file         # Only stdout to file!
```

#### Redirect stderr to stdout

```bash
command 2>&1
```

**Use case:** Pipe errors with normal output:
```bash
command 2>&1 | grep "error"
```

#### Redirect stdout to stderr

```bash
command >&2
# Or explicitly:
command 1>&2
```

**Use case:** Error messages in scripts:
```bash
echo "Error: File not found" >&2
```

### Special File: `/dev/null`

**Purpose:** The "black hole" - discards all data written to it

**Common uses:**

```bash
# Hide all output
command > /dev/null 2>&1

# Hide errors only
command 2> /dev/null

# Hide normal output only
command > /dev/null

# Silent execution
command &> /dev/null
```

**Example:**
```bash
# Find files without "Permission denied" noise
find / -name "config.txt" 2> /dev/null
```

### Practical Examples

**1. Save output, display errors:**
```bash
make > build.log 2>&1
```

**2. Suppress errors, see output:**
```bash
grep -r "pattern" /etc/ 2> /dev/null
```

**3. Separate success and failures:**
```bash
./process_files.sh > successes.txt 2> failures.txt
```

**4. Debug script with stderr:**
```bash
echo "Debug: Variable x=$x" >&2
```

---

## Pattern Matching with grep

### What is grep?

**grep** = "Global Regular Expression Print"

**Purpose:** Search for patterns in text files or streams.

### Basic Syntax

```bash
grep [OPTIONS] PATTERN [FILE...]
```

**Behavior:**
- Without files: reads from stdin
- With files: searches specified files
- Prints lines that match the pattern

### Essential Options

| Option | Description |
|--------|-------------|
| `-i` | Case-insensitive search |
| `-v` | Invert match (show non-matching lines) |
| `-n` | Show line numbers |
| `-c` | Count matching lines |
| `-l` | List filenames with matches only |
| `-r` or `-R` | Recursive search through directories |
| `-A NUM` | Show NUM lines after match |
| `-B NUM` | Show NUM lines before match |
| `-C NUM` | Show NUM lines before and after match |
| `-w` | Match whole words only |
| `-x` | Match whole lines only |
| `-o` | Show only matching part |
| `--color` | Highlight matches |
| `-E` | Extended regex (same as `egrep`) |
| `-F` | Fixed strings (same as `fgrep`) |

### Basic Pattern Matching

**Simple string search:**
```bash
grep "hello" file.txt
# Finds lines containing "hello"
```

**Case-insensitive:**
```bash
grep -i "hello" file.txt
# Matches: hello, Hello, HELLO, HeLLo, etc.
```

**Whole word match:**
```bash
grep -w "cat" file.txt
# Matches: "cat" but not "catalog" or "bobcat"
```

**Count matches:**
```bash
grep -c "error" logfile.txt
# Output: 42 (number of matching lines)
```

### Context Display

**Show lines around matches:**

```bash
# 2 lines before each match
grep -B 2 "pattern" file.txt

# 3 lines after each match
grep -A 3 "pattern" file.txt

# 2 lines before and after (context)
grep -C 2 "pattern" file.txt
```

**Example:**
```bash
grep -A 2 -B 2 "ERROR" app.log
# Shows error line plus 2 lines before and 2 after
```

### Line Numbers

```bash
grep -n "TODO" code.py
# Output: 15:# TODO: Fix this bug
#         42:# TODO: Optimize algorithm
```

### Inverted Match

```bash
# Show lines that DON'T contain pattern
grep -v "comment" file.txt

# Remove blank lines
grep -v "^$" file.txt

# Show files without pattern
grep -L "pattern" *.txt
```

### Regular Expressions Basics

**Special characters:**

| Character | Meaning |
|-----------|---------|
| `.` | Any single character |
| `^` | Start of line |
| `$` | End of line |
| `*` | Zero or more of previous |
| `+` | One or more of previous (with `-E`) |
| `?` | Zero or one of previous (with `-E`) |
| `[]` | Character class |
| `\|` | OR (with `-E`) |
| `()` | Grouping (with `-E`) |

**Examples:**

```bash
# Lines starting with "Error"
grep "^Error" file.txt

# Lines ending with "."
grep "\.$" file.txt

# Lines with 3-digit numbers
grep -E "[0-9]{3}" file.txt

# Email addresses (simple pattern)
grep -E "[a-zA-Z0-9]+@[a-zA-Z0-9]+\.[a-zA-Z]+" file.txt

# IP addresses (simple pattern)
grep -E "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" file.txt
```

### Practical grep Patterns

**Find specific file types:**
```bash
grep -i "\.jpg$" file_list.txt
```

**Find function definitions (Python):**
```bash
grep -n "^def " script.py
```

**Find TODO comments:**
```bash
grep -r "TODO\|FIXME\|XXX" .
```

**Find non-empty lines:**
```bash
grep -v "^$" file.txt
```

**Find errors in logs:**
```bash
grep -i "error\|warning\|fail" /var/log/syslog
```

### Combining grep with Other Tools

```bash
# Count unique IPs in log
cut -d ' ' -f 1 access.log | sort -u | wc -l

# Find processes by name
ps aux | grep "python"

# Search in specific file types
find . -name "*.txt" -exec grep -H "pattern" {} \;

# Recursive search with line numbers
grep -rn "function_name" /project/src/
```

---

## File Searching with find

### What is find?

**find** searches for files and directories based on various criteria.

### Basic Syntax

```bash
find [PATH] [OPTIONS] [TESTS] [ACTIONS]
```

**Components:**
- **PATH:** Where to search (default: current directory)
- **OPTIONS:** Modify behavior
- **TESTS:** Criteria for matching files
- **ACTIONS:** What to do with matches

### Essential Tests (Criteria)

| Test | Description |
|------|-------------|
| `-name PATTERN` | Match filename (case-sensitive) |
| `-iname PATTERN` | Match filename (case-insensitive) |
| `-type TYPE` | Match file type (f=file, d=directory, l=link) |
| `-size N` | Match file size |
| `-mtime N` | Modified N days ago |
| `-user NAME` | Owned by user |
| `-perm MODE` | Has permissions |
| `-path PATTERN` | Match full path |
| `-empty` | Empty files/directories |

### Basic find Examples

**Find by name:**
```bash
# Find specific file
find /home -name "config.txt"

# Case-insensitive
find /home -iname "readme.md"

# Pattern matching
find . -name "*.txt"

# All Python files
find /project -name "*.py"
```

**Find by type:**
```bash
# All directories
find /home -type d

# All regular files
find /var/log -type f

# All symbolic links
find /usr/bin -type l
```

**Find by size:**
```bash
# Files larger than 100MB
find / -size +100M

# Files smaller than 1KB
find . -size -1k

# Exactly 512 bytes
find . -size 512c
```

### Limiting Search Depth

```bash
# Search only in /home (not subdirectories)
find /home -maxdepth 1 -type d

# Search down to 3 levels
find /home -maxdepth 3 -name "*.txt"

# Start search at depth 2
find /home -mindepth 2 -name "*.jpg"
```

**Example:**
```bash
# Find directories in /home (3 levels deep max)
find /home -maxdepth 3 -type d
```

### Combining Tests

**Logical operators:**

| Operator | Meaning |
|----------|---------|
| `-and` or `-a` | Both conditions (default) |
| `-or` or `-o` | Either condition |
| `-not` or `!` | Negate condition |
| `( )` | Grouping (escape with `\`) |

**Examples:**

```bash
# Files ending in .txt OR .md
find . -name "*.txt" -or -name "*.md"

# Directories containing "log" in name
find /var -type d -name "*log*"

# Files NOT owned by root
find /home -not -user root

# Large Python files
find . -name "*.py" -and -size +1M

# Empty directories
find /tmp -type d -empty
```

### Actions

| Action | Description |
|--------|-------------|
| `-print` | Print filename (default) |
| `-ls` | Detailed listing |
| `-delete` | Delete matched files (DANGEROUS!) |
| `-exec COMMAND {} \;` | Execute command on each match |
| `-exec COMMAND {} +` | Execute command with all matches |
| `-ok COMMAND {} \;` | Interactive exec (prompts for each) |

### `-exec` - Execute Commands

**Basic syntax:**
```bash
find [path] [tests] -exec command {} \;
```

**Components:**
- `{}` - Placeholder for found filename
- `\;` - Terminates the command

**Examples:**

**1. Display file info:**
```bash
find . -name "*.txt" -exec ls -l {} \;
```

**2. Copy files:**
```bash
find ~/lab1 -type f -exec cp {} ~/backup/ \;
```

**3. Change permissions:**
```bash
find /project -name "*.sh" -exec chmod +x {} \;
```

**4. Search content:**
```bash
find . -name "*.py" -exec grep -H "TODO" {} \;
```

**5. Delete old files:**
```bash
find /tmp -name "*.tmp" -mtime +7 -exec rm {} \;
```

### Single vs Multiple Execution

**`\;` - Run once per file (less efficient):**
```bash
find . -name "*.txt" -exec echo "Found: {}" \;
# Runs: echo "Found: file1.txt"
#       echo "Found: file2.txt"
#       echo "Found: file3.txt"
```

**`+` - Run once with all files (more efficient):**
```bash
find . -name "*.txt" -exec echo "Found: " {} +
# Runs: echo "Found: " file1.txt file2.txt file3.txt
```

### Suppressing Errors

```bash
# Hide "Permission denied" errors
find / -name "*.conf" 2> /dev/null

# Or redirect to null
find /etc -type f -name "*.conf" 2>&1 > /dev/null
```

### Practical find Examples

**Find and archive:**
```bash
find /project -name "*.log" -exec gzip {} \;
```

**Find recent files:**
```bash
find /home -mtime -7  # Modified in last 7 days
```

**Find and move:**
```bash
find . -name "*.bak" -exec mv {} ~/backup/ \;
```

**Find empty files:**
```bash
find /tmp -type f -empty -delete
```

**Find files with pattern in name:**
```bash
find /home -name "*2024*" -type f
```

### Common Patterns

```bash
# All JPG files (case-insensitive)
find . -iname "*.jpg"

# Directories only, 3 levels max
find /home -maxdepth 3 -type d

# Find and count
find . -name "*.txt" | wc -l

# Find with full details
find /var/log -name "*.log" -ls

# Interactive deletion
find . -name "*.tmp" -ok rm {} \;
```

---

## File Type Detection

### The `file` Command

**Purpose:** Determine file type by examining file content (not just extension).

**Syntax:**
```bash
file [OPTIONS] FILE...
```

### How file Works

**Doesn't rely on extensions** - examines file contents:
- **Magic numbers** - Special bytes at file start
- **Content analysis** - Text patterns, structure
- **Metadata** - File headers, markers

**Example:**
```bash
# File named "image.txt" but actually a JPEG
file image.txt
# Output: image.txt: JPEG image data, ...
```

### Common File Types

```bash
file document.txt
# Output: ASCII text

file image.jpg
# Output: JPEG image data, JFIF standard 1.01

file program
# Output: ELF 64-bit LSB executable, x86-64

file archive.tar.gz
# Output: gzip compressed data

file script.sh
# Output: Bourne-Again shell script, ASCII text executable

file video.mp4
# Output: ISO Media, MP4 Base Media v1

file /dev/null
# Output: character special (0/0)
```

### Useful Options

| Option | Description |
|--------|-------------|
| `-b` | Brief mode (omit filename) |
| `-i` | Show MIME type |
| `-L` | Follow symbolic links |
| `-f LIST` | Read filenames from file |

**Examples:**

```bash
# Brief output
file -b document.pdf
# Output: PDF document, version 1.4

# MIME type
file -i image.jpg
# Output: image.jpg: image/jpeg; charset=binary

# Follow symlinks
file -L /usr/bin/python
# Shows target file type, not "symbolic link"
```

### Practical Uses

**1. Verify file types:**
```bash
# Check if files are actually JPEGs
for f in *.jpg; do
    if ! file "$f" | grep -q "JPEG"; then
        echo "$f is not a JPEG"
    fi
done
```

**2. Find misnamed files:**
```bash
# Find .txt files that aren't text
find . -name "*.txt" -exec file {} \; | grep -v "ASCII\|UTF-8"
```

**3. Filter by actual type:**
```bash
# Remove non-image files from jpg directory
find ~/pictures -name "*.jpg" -exec sh -c '
    if ! file "$1" | grep -q "image data"; then
        echo "Removing non-image: $1"
        rm "$1"
    fi
' _ {} \;
```

**4. Batch file identification:**
```bash
file * | grep "shell script"
# Lists all shell scripts
```

---

## Command Execution Tools

### `xargs` - Build and Execute Commands

**Purpose:** Convert stdin into arguments for another command.

**Problem it solves:**
```bash
# This doesn't work as expected:
find . -name "*.txt" | rm
# rm doesn't read filenames from stdin!

# Solution with xargs:
find . -name "*.txt" | xargs rm
```

**Basic syntax:**
```bash
command | xargs [OPTIONS] command2
```

### How xargs Works

1. Reads items from stdin (space/newline separated)
2. Groups them into manageable chunks
3. Executes specified command with those arguments

**Example:**
```bash
echo "file1.txt file2.txt file3.txt" | xargs cat
# Executes: cat file1.txt file2.txt file3.txt
```

### Common Options

| Option | Description |
|--------|-------------|
| `-n NUM` | Max NUM arguments per command |
| `-I REPLACE` | Replace string in command |
| `-t` | Print command before executing |
| `-p` | Prompt before executing |
| `-0` | Input items separated by null (not space) |
| `-d DELIM` | Custom delimiter |

### Practical xargs Examples

**1. Process multiple files:**
```bash
find . -name "*.txt" | xargs wc -l
```

**2. Delete files from list:**
```bash
cat files_to_delete.txt | xargs rm
```

**3. One argument at a time:**
```bash
find . -name "*.jpg" | xargs -n 1 basename
```

**4. Replace string:**
```bash
find . -name "*.txt" | xargs -I {} cp {} /backup/
# {} is replaced with each filename
```

**5. Prompt before action:**
```bash
find . -name "*.log" | xargs -p rm
# Asks: rm file.log?...
```

**6. Handle filenames with spaces:**
```bash
find . -name "*.txt" -print0 | xargs -0 rm
# -print0 uses null separator
# -0 expects null-separated input
```

### xargs vs find -exec

| Aspect | xargs | find -exec |
|--------|-------|------------|
| **Efficiency** | Groups arguments | One per file |
| **Syntax** | More flexible | More integrated |
| **Spaces in names** | Needs `-0` | Handles naturally |

**When to use xargs:**
- Processing piped input
- Need advanced argument control
- Efficiency matters (many files)

**When to use find -exec:**
- Files found by `find`
- Filenames with spaces/special chars
- Simple one-command operations

---

### `tee` - Duplicate Output

**Purpose:** Read from stdin, write to stdout AND files simultaneously.

**Analogy:** A pipe "tee" connector - splits flow into two directions.

**Basic syntax:**
```bash
command | tee [OPTIONS] file
```

### How tee Works

```
command → tee → stdout (terminal)
              ↓
              file
```

**Example:**
```bash
ls -l | tee listing.txt
# Shows output on screen
# AND saves to listing.txt
```

### Common Options

| Option | Description |
|--------|-------------|
| `-a` | Append to file (not overwrite) |

### Practical tee Examples

**1. Save output while viewing:**
```bash
./script.sh | tee output.txt
# See output in real-time
# Save complete log to file
```

**2. Save to multiple files:**
```bash
command | tee file1.txt file2.txt file3.txt
```

**3. Append to log:**
```bash
command | tee -a logfile.txt
```

**4. Combine with redirection:**
```bash
# Save both stdout and stderr
command 2>&1 | tee log.txt
```

**5. Progress monitoring:**
```bash
tar -czf backup.tar.gz /data | tee progress.log
```

**6. Debug pipelines:**
```bash
# See intermediate results
cat data | tee step1.txt | grep pattern | tee step2.txt | sort
```

### Real-world Use Cases

**Installation logs:**
```bash
sudo apt install package 2>&1 | tee install.log
```

**Build logs:**
```bash
make 2>&1 | tee build.log
```

**Long-running commands:**
```bash
./long_process.sh | tee -a process.log
# Monitor in real-time, keep permanent log
```

---

## Text Manipulation Tools

### `tac` - Reverse Line Order

**Purpose:** Opposite of `cat` - displays file in reverse line order.

**Syntax:**
```bash
tac [FILE]
```

**Example:**

**File content:**
```
Line 1
Line 2
Line 3
```

**Output of `tac`:**
```
Line 3
Line 2
Line 1
```

**Use cases:**
- View log files (newest first)
- Reverse processing order
- Stack-like operations

**Example:**
```bash
# Show most recent log entries first
tac /var/log/syslog | head -20
```

### `rev` - Reverse Characters

**Purpose:** Reverse each line character by character.

**Syntax:**
```bash
rev [FILE]
```

**Example:**

**Input:**
```
Hello World
12345
```

**Output:**
```
dlroW olleH
54321
```

**Use cases:**
- Text puzzles
- Format conversion
- Mirroring text

**Practical combination:**
```bash
# Reverse entire file (lines AND characters)
tac file.txt | rev
```

---

## Command Reference Summary

### grep - Pattern Searching
```bash
grep [OPTIONS] PATTERN [FILE]
-i      Case-insensitive
-v      Invert match
-n      Line numbers
-A NUM  NUM lines after
-B NUM  NUM lines before
-C NUM  Context (before + after)
-r      Recursive
-w      Whole word
-c      Count matches
```

### find - File Searching
```bash
find [PATH] [TESTS] [ACTIONS]
-name PATTERN     Filename match
-type f/d/l       File/Directory/Link
-size +/-N        Size criteria
-mtime N          Modified N days ago
-maxdepth N       Limit depth
-exec CMD {} \;   Execute command
2> /dev/null      Hide errors
```

### file - Type Detection
```bash
file [OPTIONS] FILE
-b      Brief output
-i      MIME type
-L      Follow links
```

### xargs - Argument Builder
```bash
command | xargs [OPTIONS] command2
-n NUM      Max args per invocation
-I STR      Replace string
-0          Null-separated input
-p          Prompt before running
```

### tee - Output Duplication
```bash
command | tee [OPTIONS] file
-a      Append mode
```

---

## Best Practices

### 1. Always Handle Errors

```bash
# Good: Hide expected errors
find / -name "config.txt" 2> /dev/null

# Bad: Pollutes output
find / -name "config.txt"
```

### 2. Use Appropriate Tools

```bash
# Good: find with -exec
find . -name "*.txt" -exec grep "pattern" {} \;

# Less efficient: find + xargs (but works)
find . -name "*.txt" | xargs grep "pattern"
```

### 3. Quote Variables

```bash
# Good:
find . -name "*.txt" -exec cp {} "$backup_dir/" \;

# Bad: Breaks with spaces in $backup_dir
find . -name "*.txt" -exec cp {} $backup_dir/ \;
```

### 4. Test Destructive Commands

```bash
# Test first:
find . -name "*.tmp" -print

# Then delete:
find . -name "*.tmp" -delete
```

### 5. Use -print0 with xargs -0 for Spaces

```bash
# Handles spaces correctly:
find . -name "*.txt" -print0 | xargs -0 rm

# May fail with spaces:
find . -name "*.txt" | xargs rm
```

---

## Summary

Lab 3 introduces powerful searching and processing tools:

**Error Management:**
- `2>` - Redirect stderr
- `2>&1` - Combine streams
- `/dev/null` - Discard output

**Pattern Matching:**
- `grep` - Search text patterns
- `-i`, `-v`, `-n`, `-A`, `-B`, `-C` - Essential options

**File Searching:**
- `find` - Locate files by criteria
- `-name`, `-type`, `-size` - Common tests
- `-exec` - Execute commands on results
- `-maxdepth` - Control search depth

**Utilities:**
- `file` - Detect file types
- `xargs` - Build command arguments
- `tee` - Duplicate output
- `tac` - Reverse lines
- `rev` - Reverse characters

**Key Principle:** Combine tools to build sophisticated file processing pipelines.
