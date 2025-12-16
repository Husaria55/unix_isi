# Theory Guide - Lab 2: Basic Data Processing

## Table of Contents
1. [Introduction to Data Processing](#introduction-to-data-processing)
2. [Output Redirection](#output-redirection)
3. [Command Substitution](#command-substitution)
4. [Wildcards and Globbing](#wildcards-and-globbing)
5. [Grouping Commands](#grouping-commands)
6. [Text Processing Commands](#text-processing-commands)
7. [Command Reference](#command-reference)
8. [Practical Examples](#practical-examples)

---

## Introduction to Data Processing

Unix's power lies in its ability to process text data through pipelines and redirections. Lab 2 focuses on:
- **Redirecting output** to files instead of the terminal
- **Substituting command output** into other commands
- **Processing text streams** through specialized tools
- **Combining simple tools** to solve complex problems

The Unix philosophy: "Write programs that handle text streams, because that is a universal interface."

---

## Output Redirection

### Understanding Standard Streams (Review)

Every Unix process has three standard streams:
1. **stdin (0)** - Standard Input - where data comes from
2. **stdout (1)** - Standard Output - where normal output goes
3. **stderr (2)** - Standard Error - where error messages go

### Redirection Operators

#### `>` - Redirect stdout (Overwrite)

**Syntax:**
```bash
command > file
```

**Behavior:**
- Redirects standard output to a file
- **OVERWRITES** the file if it exists
- Creates the file if it doesn't exist
- **DESTRUCTIVE** - old content is lost forever

**Examples:**
```bash
echo "Hello" > file.txt          # Creates/overwrites file.txt
date > today.txt                 # Saves current date to file
ls -l > listing.txt              # Saves directory listing
```

**Visual representation:**
```
command --[stdout]--> file (OVERWRITE)
```

**⚠️ WARNING:** Using `>` on an existing file **destroys its contents** without warning!

#### `>>` - Redirect stdout (Append)

**Syntax:**
```bash
command >> file
```

**Behavior:**
- Redirects standard output to a file
- **APPENDS** to the end if file exists
- Creates the file if it doesn't exist
- **NON-DESTRUCTIVE** - preserves existing content

**Examples:**
```bash
echo "First line" > file.txt     # Creates file
echo "Second line" >> file.txt   # Appends to file
echo "Third line" >> file.txt    # Appends more
```

**Visual representation:**
```
command --[stdout]--> file (APPEND to end)
```

**Use cases:**
- Log files (adding entries)
- Building files incrementally
- Collecting output from multiple commands

#### Comparison: `>` vs `>>`

| Operator | Behavior | Use When |
|----------|----------|----------|
| `>` | Overwrites | Starting fresh, single output |
| `>>` | Appends | Adding to existing data, logs |

**Example showing difference:**
```bash
echo "A" > test.txt    # File contains: A
echo "B" > test.txt    # File contains: B (A is gone!)
echo "C" >> test.txt   # File contains: B\nC
```

### Input Redirection: `<`

**Syntax:**
```bash
command < input_file
```

**Behavior:**
- Provides file contents as standard input to a command
- Command reads from file instead of keyboard

**Examples:**
```bash
wc -l < file.txt           # Count lines from file
sort < unsorted.txt        # Sort contents of file
mail user < message.txt    # Send email from file
```

**Practical use:**
Many commands can read from files directly, but `<` is useful for:
- Commands that only read from stdin
- Clarity in scripts
- Combining with other redirections

---

## Command Substitution

### What is Command Substitution?

**Command substitution** allows you to use the output of one command as an argument to another command.

### Syntax

**Modern (preferred):**
```bash
$(command)
```

**Legacy (backticks):**
```bash
`command`
```

**Recommendation:** Always use `$(command)` syntax - it's clearer and nests better.

### How It Works

1. The shell executes the command inside `$()`
2. The command's output replaces `$(command)`
3. The outer command sees the substituted text

**Example:**
```bash
echo "Today is $(date +%Y-%m-%d)"
```

**Execution flow:**
1. `date +%Y-%m-%d` executes → outputs `2025-12-16`
2. Shell substitutes: `echo "Today is 2025-12-16"`
3. Final output: `Today is 2025-12-16`

### Practical Examples

#### 1. Using command output in filenames

```bash
# Create file with today's date in name
echo "Log entry" > log_$(date +%Y%m%d).txt
# Creates: log_20251216.txt
```

#### 2. Using command output as arguments

```bash
# Show info about date command
ls -l $(which date)
# Expands to: ls -l /usr/bin/date
```

#### 3. Storing command output in variables

```bash
current_user=$(whoami)
echo "You are: $current_user"
```

#### 4. Nested substitution

```bash
# Find and display file size
echo "Date command is $(wc -c < $(which date)) bytes"
```

### Common Use Cases

| Use Case | Example |
|----------|---------|
| Dynamic filenames | `mv file.txt backup_$(date +%s).txt` |
| Counting results | `echo "Found $(find . -name "*.txt" | wc -l) files"` |
| Path resolution | `cat $(find . -name config.txt)` |
| Variable assignment | `files=$(ls *.txt)` |

### Pitfalls to Avoid

**Problem: Word splitting**
```bash
# Wrong: Files with spaces break
files=$(ls)
for f in $files; do echo $f; done  # Breaks on spaces!

# Right: Quote the substitution
for f in $(ls); do echo "$f"; done  # Still not ideal
# Best: Use proper glob
for f in *; do echo "$f"; done
```

**Problem: Unexpected newlines**
```bash
# Output includes newlines
content=$(cat multiline.txt)
# Use quotes to preserve them:
echo "$content"
```

---

## Wildcards and Globbing

### What are Wildcards?

**Wildcards** (also called **glob patterns**) are special characters that match patterns in filenames.

### The Asterisk: `*`

**Matches:** Zero or more characters

**Examples:**

```bash
*.txt           # All files ending with .txt
file*           # Files starting with 'file'
*               # All files in current directory
*.tar.gz        # All .tar.gz files
report_*_2024   # report_january_2024, report_feb_2024, etc.
```

**Important:** `*` does NOT match hidden files (starting with `.`)

To match hidden files:
```bash
.*              # All hidden files
*               # All visible files
```

### The Question Mark: `?`

**Matches:** Exactly one character

**Examples:**
```bash
file?.txt       # file1.txt, fileA.txt, but not file12.txt
???             # Any three-character filename
image?.jpg      # image1.jpg, imageA.jpg
```

### Character Classes: `[...]`

**Matches:** Any one character from the set

**Examples:**
```bash
file[123].txt      # file1.txt, file2.txt, file3.txt
[A-Z]*             # Files starting with uppercase letter
[0-9][0-9].dat     # 00.dat, 01.dat, ..., 99.dat
*.[ch]             # Files ending in .c or .h
```

**Ranges:**
- `[a-z]` - lowercase letters
- `[A-Z]` - uppercase letters
- `[0-9]` - digits
- `[a-zA-Z]` - all letters

### Brace Expansion: `{...}`

**Not a wildcard, but related** - generates multiple strings

**Examples:**
```bash
file{1,2,3}.txt     # Expands to: file1.txt file2.txt file3.txt
{Jan,Feb,Mar}_2024  # Expands to: Jan_2024 Feb_2024 Mar_2024
file.{txt,md,pdf}   # Expands to: file.txt file.md file.pdf
```

### Practical Wildcard Examples

```bash
# Copy all JPG files
cp *.jpg ~/pictures/

# List all text and markdown files
ls *.{txt,md}

# Remove all backup files
rm *~ *.bak

# Find files with 4-digit names
ls [0-9][0-9][0-9][0-9]

# Move all files starting with 'temp'
mv temp* /tmp/
```

### Important Behaviors

**Expansion happens in the shell:**
```bash
# Shell expands BEFORE command runs
echo *.txt
# Shell sees: echo file1.txt file2.txt file3.txt
# Command receives multiple arguments
```

**No matches = literal string (usually):**
```bash
# If no .xyz files exist:
ls *.xyz
# Might produce: ls: cannot access '*.xyz': No such file or directory
```

**Quoting prevents expansion:**
```bash
echo "*.txt"    # Prints: *.txt (literal)
echo '*.txt'    # Prints: *.txt (literal)
echo *.txt      # Prints: file1.txt file2.txt ... (expanded)
```

---

## Grouping Commands

### Parentheses: `( commands )`

**Purpose:** Execute commands in a subshell

**Syntax:**
```bash
(command1; command2; command3)
```

**Characteristics:**
- Commands run in a **subshell** (separate process)
- Variable changes don't affect parent shell
- Current directory changes are isolated
- Exit status is that of the last command

**Examples:**

```bash
# Group output for redirection
(echo "Line 1"; echo "Line 2") > file.txt

# Change directory temporarily
(cd /tmp && ls)
# Still in original directory after this

# Chain commands with output redirection
(date; whoami; pwd) > system_info.txt
```

### Curly Braces: `{ commands; }`

**Purpose:** Execute commands in current shell

**Syntax:**
```bash
{ command1; command2; command3; }
```

**Important:** Note the spaces and semicolon!
- Space after `{`
- Space before `}`
- Semicolon after last command

**Characteristics:**
- Commands run in **current shell**
- Variable changes affect current shell
- More efficient (no subshell overhead)
- Must be terminated with `;` or newline

**Examples:**
```bash
# Group for redirection
{ echo "Header"; cat file.txt; } > output.txt

# Set variables that persist
{ x=5; y=10; }
echo $x  # Prints: 5
```

### Logical Operators with Grouping

```bash
# AND with grouping
(command1 && command2) || command3

# All commands must succeed
(mkdir dir && cd dir && touch file)

# Execute if previous group fails
(command1 || command2) && command3
```

---

## Text Processing Commands

### Overview

Unix provides powerful tools for processing text:
- **Extract columns:** `cut`
- **Transform characters:** `tr`
- **Select lines:** `head`, `tail`
- **Sort data:** `sort`
- **Remove duplicates:** `uniq`
- **Generate sequences:** `seq`
- **Display text:** `cat`, `echo`

---

## Command Reference

### `cat` - Concatenate and Display Files

**Purpose:** Display, concatenate, or create files

**Basic syntax:**
```bash
cat [OPTIONS] [FILE...]
```

**Common options:**

| Option | Description |
|--------|-------------|
| `-n` | Number all lines |
| `-b` | Number non-empty lines only |
| `-A` | Show all characters (including non-printable) |
| `-s` | Squeeze multiple blank lines into one |

**Use cases:**

**1. Display file contents:**
```bash
cat file.txt
```

**2. Concatenate files:**
```bash
cat file1.txt file2.txt file3.txt
```

**3. Create a file (interactive input):**
```bash
cat > newfile.txt
Type your content here...
Press CTRL+D to finish
```

**4. Append to file:**
```bash
cat >> existingfile.txt
More content...
CTRL+D
```

**5. Concatenate to new file:**
```bash
cat file1.txt file2.txt > combined.txt
```

**6. Number lines:**
```bash
cat -n file.txt
```

**Why the name "cat"?**
"Concatenate" - its original purpose was to concatenate files.

---

### `cut` - Extract Columns from Text

**Purpose:** Extract specific columns or fields from text

**Basic syntax:**
```bash
cut [OPTIONS] [FILE]
```

**Essential options:**

| Option | Description |
|--------|-------------|
| `-d DELIMITER` | Specify field delimiter (default is TAB) |
| `-f FIELDS` | Select fields (columns) |
| `-c CHARACTERS` | Select character positions |
| `--complement` | Invert selection |

**Field specification:**
- `-f 1` - First field
- `-f 1,3,5` - Fields 1, 3, and 5
- `-f 1-3` - Fields 1 through 3
- `-f 3-` - Field 3 to end of line

**Examples:**

**1. Extract first field (colon-delimited):**
```bash
cut -d ':' -f 1 /etc/passwd
# Extracts usernames
```

**2. Extract multiple fields:**
```bash
cut -d ':' -f 1,3,6 /etc/passwd
# Extracts username, UID, and home directory
```

**3. Extract character ranges:**
```bash
cut -c 1-5 file.txt
# First 5 characters of each line
```

**4. CSV processing:**
```bash
cut -d ',' -f 2 data.csv
# Extract second column from CSV
```

**Important notes:**
- Default delimiter is TAB (not space!)
- Specify delimiter with `-d`
- Cannot reorder fields (always outputs in original order)
- Cannot repeat fields

---

### `tr` - Translate or Delete Characters

**Purpose:** Translate, delete, or squeeze characters

**Basic syntax:**
```bash
tr [OPTIONS] SET1 [SET2]
```

**Reads from stdin only** (doesn't take file arguments directly)

**Common options:**

| Option | Description |
|--------|-------------|
| `-d` | Delete characters in SET1 |
| `-s` | Squeeze repeated characters in SET1 |
| `-c` | Complement SET1 (all characters NOT in SET1) |

**Use cases:**

**1. Translate characters (uppercase to lowercase):**
```bash
echo "HELLO" | tr 'A-Z' 'a-z'
# Output: hello
```

**2. Delete characters:**
```bash
echo "Hello, World!" | tr -d ','
# Output: Hello World!
```

**3. Squeeze repeated characters:**
```bash
echo "hello    world" | tr -s ' '
# Output: hello world
```

**4. Replace characters:**
```bash
echo "hello" | tr 'el' 'ip'
# Output: hippo
```

**5. Delete newlines (join lines):**
```bash
cat file.txt | tr -d '\n'
```

**6. Replace spaces with newlines:**
```bash
echo "word1 word2 word3" | tr ' ' '\n'
# Output:
# word1
# word2
# word3
```

**Character sets:**
- `'a-z'` - lowercase letters
- `'A-Z'` - uppercase letters
- `'0-9'` - digits
- `' '` - space
- `'\n'` - newline
- `':'` - colon

**Important:** `tr` operates character-by-character, not on words or patterns.

---

### `head` - Display Beginning of Files

**Purpose:** Display first N lines of a file

**Basic syntax:**
```bash
head [OPTIONS] [FILE]
```

**Common options:**

| Option | Description |
|--------|-------------|
| `-n NUM` | Display first NUM lines (default: 10) |
| `-c NUM` | Display first NUM bytes |
| `-q` | Quiet mode (don't print headers) |
| `-v` | Verbose mode (always print headers) |

**Examples:**

```bash
head file.txt              # First 10 lines
head -n 5 file.txt         # First 5 lines
head -5 file.txt           # Same as above (shorthand)
head -n 1 file.txt         # First line only
head -c 100 file.txt       # First 100 bytes
```

**With pipes:**
```bash
ls -l | head -n 20         # First 20 lines of ls output
cat file.txt | head -n 3   # First 3 lines
```

**Multiple files:**
```bash
head -n 5 file1.txt file2.txt
# Shows first 5 lines of each with headers
```

---

### `tail` - Display End of Files

**Purpose:** Display last N lines of a file

**Basic syntax:**
```bash
tail [OPTIONS] [FILE]
```

**Common options:**

| Option | Description |
|--------|-------------|
| `-n NUM` | Display last NUM lines (default: 10) |
| `-c NUM` | Display last NUM bytes |
| `-f` | Follow mode (watch file for new content) |
| `-q` | Quiet mode |
| `+NUM` | Start at line NUM (not last NUM lines) |

**Examples:**

```bash
tail file.txt              # Last 10 lines
tail -n 5 file.txt         # Last 5 lines
tail -5 file.txt           # Same (shorthand)
tail -n 1 file.txt         # Last line only
tail -c 100 file.txt       # Last 100 bytes
```

**Follow mode (watch logs):**
```bash
tail -f /var/log/syslog    # Watch log file in real-time
# Press CTRL+C to stop
```

**Start from line N:**
```bash
tail -n +5 file.txt        # From line 5 to end
```

**Combining head and tail:**
```bash
# Extract specific line range (lines 10-20)
head -n 20 file.txt | tail -n 11

# Extract line 15 only
head -n 15 file.txt | tail -n 1
```

---

### `sort` - Sort Lines of Text

**Purpose:** Sort lines alphabetically or numerically

**Basic syntax:**
```bash
sort [OPTIONS] [FILE]
```

**Essential options:**

| Option | Description |
|--------|-------------|
| `-n` | Numeric sort (not alphabetic) |
| `-r` | Reverse sort (descending) |
| `-u` | Remove duplicates (unique) |
| `-k FIELD` | Sort by field number |
| `-t CHAR` | Field delimiter |
| `-o FILE` | Output to file |
| `-f` | Case-insensitive |

**Examples:**

**1. Basic sort:**
```bash
sort file.txt              # Alphabetical, ascending
```

**2. Numeric sort:**
```bash
sort -n numbers.txt        # Treats as numbers: 2, 10, 100
# Without -n: 10, 100, 2 (alphabetical)
```

**3. Reverse sort:**
```bash
sort -r file.txt           # Descending order
sort -nr numbers.txt       # Numeric, descending
```

**4. Remove duplicates:**
```bash
sort -u file.txt           # Sort and remove duplicates
```

**5. Sort by field:**
```bash
sort -t ':' -k 3 -n /etc/passwd
# Sort by field 3 (UID), numeric, colon-delimited
```

**6. Sort CSV by column:**
```bash
sort -t ',' -k 2 data.csv  # Sort by 2nd column
```

**Practical combinations:**
```bash
# Sort numbers, largest first
sort -nr numbers.txt

# Sort unique values
sort file.txt | uniq
# Or:
sort -u file.txt

# Sort by second field, numeric
cut -d ',' -f 2 data.csv | sort -n
```

---

### `uniq` - Remove Duplicate Lines

**Purpose:** Remove or count consecutive duplicate lines

**Basic syntax:**
```bash
uniq [OPTIONS] [INPUT] [OUTPUT]
```

**⚠️ CRITICAL:** `uniq` only removes **consecutive** duplicates!
**Always sort first:** `sort file.txt | uniq`

**Common options:**

| Option | Description |
|--------|-------------|
| `-c` | Count occurrences |
| `-d` | Show only duplicate lines |
| `-u` | Show only unique lines |
| `-i` | Case-insensitive |

**Examples:**

**1. Remove duplicates (after sorting):**
```bash
sort file.txt | uniq
```

**2. Count occurrences:**
```bash
sort file.txt | uniq -c
# Output:
#   3 apple
#   2 banana
#   1 cherry
```

**3. Show only duplicates:**
```bash
sort file.txt | uniq -d
```

**4. Show only unique (non-repeated) lines:**
```bash
sort file.txt | uniq -u
```

**Why sort first?**
```bash
# File contents:
# apple
# banana
# apple

# Wrong:
uniq file.txt
# Output: apple, banana, apple (didn't remove second apple)

# Right:
sort file.txt | uniq
# Output: apple, banana
```

---

### `seq` - Generate Sequences

**Purpose:** Generate sequences of numbers

**Basic syntax:**
```bash
seq [OPTIONS] [FIRST] [INCREMENT] LAST
```

**Common forms:**
```bash
seq LAST              # 1 to LAST
seq FIRST LAST        # FIRST to LAST
seq FIRST INC LAST    # FIRST to LAST by INCREMENT
```

**Options:**

| Option | Description |
|--------|-------------|
| `-s STRING` | Separator (default: newline) |
| `-f FORMAT` | Printf-style format |
| `-w` | Equal width (pad with zeros) |

**Examples:**

**1. Simple sequence:**
```bash
seq 5
# Output:
# 1
# 2
# 3
# 4
# 5
```

**2. Range:**
```bash
seq 3 7
# Output: 3, 4, 5, 6, 7 (each on new line)
```

**3. With increment:**
```bash
seq 0 2 10
# Output: 0, 2, 4, 6, 8, 10
```

**4. Custom separator:**
```bash
seq -s ', ' 1 5
# Output: 1, 2, 3, 4, 5
```

**5. Zero-padded:**
```bash
seq -w 8 11
# Output:
# 08
# 09
# 10
# 11
```

**6. Floating point:**
```bash
seq 1.0 0.5 3.0
# Output: 1.0, 1.5, 2.0, 2.5, 3.0
```

**Practical uses:**
```bash
# Create numbered files
for i in $(seq 1 10); do touch file_$i.txt; done

# Generate test data
seq 1 100 > numbers.txt

# Loop iterations
seq 5 | while read i; do echo "Iteration $i"; done
```

---

### `wc` - Word Count (Review)

**Options:**
- `-l` - Count lines
- `-w` - Count words
- `-c` - Count bytes
- `-m` - Count characters

**Examples:**
```bash
wc file.txt              # Lines, words, bytes
wc -l file.txt           # Line count only
cat file.txt | wc -l     # Count via pipe
```

---

### `which` - Locate Command

**Purpose:** Show full path of executable commands

**Basic syntax:**
```bash
which [OPTIONS] command
```

**Examples:**
```bash
which date               # Output: /usr/bin/date
which python             # Shows Python path
which ls                 # Output: /usr/bin/ls
```

**Use with command substitution:**
```bash
ls -l $(which date)      # Show info about date command
file $(which bash)       # What type is bash executable?
```

---

## Practical Examples

### Example 1: Building a Log System

**Goal:** Create daily log files with timestamps

```bash
# Create log entry with timestamp
echo "$(date +%H:%M) Application started" >> ~/logs/$(date +%Y%m%d).log
```

**Breakdown:**
1. `$(date +%H:%M)` - Current time: "10:30"
2. `$(date +%Y%m%d)` - Today's date: "20251216"
3. `>>` - Append to file
4. Creates: `~/logs/20251216.log`
5. Content: "10:30 Application started"

### Example 2: Data Processing Pipeline

**Goal:** Extract unique usernames, sorted

```bash
cut -d ':' -f 1 /etc/passwd | sort | uniq
```

**Breakdown:**
1. `cut -d ':' -f 1` - Extract first field (username)
2. `sort` - Sort alphabetically
3. `uniq` - Remove duplicates (must be sorted first!)

### Example 3: Report Generation

**Goal:** Count files by extension

```bash
ls -1 | grep -o '\.[^.]*$' | sort | uniq -c | sort -nr
```

**Breakdown:**
1. `ls -1` - List files, one per line
2. `grep -o '\.[^.]*$'` - Extract file extensions
3. `sort` - Sort extensions
4. `uniq -c` - Count each extension
5. `sort -nr` - Sort by count, descending

### Example 4: Creating Test Data

**Goal:** Generate 100 numbered files

```bash
seq 1 100 | while read n; do
    echo "File number $n" > file_$(printf "%03d" $n).txt
done
```

**Creates:**
- file_001.txt
- file_002.txt
- ...
- file_100.txt

---

## Best Practices

### 1. Be Careful with Redirection

```bash
# WRONG - Destroys file!
sort file.txt > file.txt    # File becomes empty!

# RIGHT - Use temporary file
sort file.txt > file.txt.tmp && mv file.txt.tmp file.txt
```

### 2. Quote Command Substitutions

```bash
# WRONG - Word splitting issues
files=$(ls *.txt)
for f in $files; do ...

# BETTER
for f in $(ls *.txt); do echo "$f"; done

# BEST - Use glob directly
for f in *.txt; do echo "$f"; done
```

### 3. Test Before Redirecting to Files

```bash
# Test first
command args

# If looks good, redirect
command args > output.txt
```

### 4. Use Append (>>) for Logs

```bash
# Good for logs
echo "Entry" >> log.txt

# Bad for logs (overwrites!)
echo "Entry" > log.txt
```

### 5. Check Delimiter with cut

```bash
# Wrong - assumes space delimiter
cut -f 1 /etc/passwd    # Fails! Uses tab by default

# Right - specify delimiter
cut -d ':' -f 1 /etc/passwd
```

---

## Common Patterns

### Pattern 1: Extract and Process

```bash
# Extract column, process, save
cut -d ',' -f 2 data.csv | sort -n | uniq > results.txt
```

### Pattern 2: Generate Sequential Data

```bash
# Create numbered backups
seq 1 10 | while read n; do
    cp important.txt backup_$n.txt
done
```

### Pattern 3: Building Reports

```bash
# Header + data
echo "Count,Item" > report.csv
sort items.txt | uniq -c | tr -s ' ' ',' >> report.csv
```

### Pattern 4: Dynamic Filenames

```bash
# Timestamp in filename
tar -czf backup_$(date +%Y%m%d_%H%M%S).tar.gz files/
```

---

## Summary

Lab 2 introduces essential text processing and redirection:

**Redirection:**
- `>` - Overwrite file with output
- `>>` - Append output to file
- `<` - Input from file

**Command Substitution:**
- `$(command)` - Use command output as argument

**Wildcards:**
- `*` - Match zero or more characters
- `?` - Match exactly one character
- `[...]` - Match character class

**Text Processing:**
- `cat` - Display/concatenate files
- `cut` - Extract columns
- `tr` - Transform characters
- `head`/`tail` - Select lines
- `sort` - Sort lines
- `uniq` - Remove duplicates
- `seq` - Generate numbers
- `wc` - Count lines/words/bytes

**Key Principle:** Combine simple tools through pipes and redirections to solve complex problems.
