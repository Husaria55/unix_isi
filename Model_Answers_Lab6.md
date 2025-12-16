# Model Answers - Lab 6: BASH Scripting Basics

## Introduction Questions

### Question 1: Single vs Double Quotes in BASH

**What's the difference between single quotes (') and double quotes (") in bash?**

**Short answer:** Single quotes preserve everything literally, double quotes allow variable and command substitution.

**Detailed explanation:**

**Single quotes (`'`):**
- **Literal interpretation** - everything inside is treated as plain text
- **No variable expansion**
- **No command substitution**
- **No escape sequences**

```bash
name="Alice"
echo 'Hello, $name'       # Output: Hello, $name
echo 'Today is $(date)'   # Output: Today is $(date)
echo 'Price: $10'         # Output: Price: $10
```

**Double quotes (`"`):**
- **Variable expansion enabled** - `$var` replaced with value
- **Command substitution works** - `$(command)` executes
- **Escape sequences work** - `\n`, `\t`, etc.
- **Preserves spaces**

```bash
name="Alice"
echo "Hello, $name"       # Output: Hello, Alice
echo "Today is $(date)"   # Output: Today is Mon Dec 16...
echo "Price: \$10"        # Output: Price: $10
```

**Comparison table:**

| Feature | Single Quotes | Double Quotes |
|---------|---------------|---------------|
| Variable expansion | No | Yes |
| Command substitution | No | Yes |
| Escape sequences | No | Yes |
| Literal `$` | Yes | No (use `\$`) |
| Preserve spaces | Yes | Yes |

**When to use:**
- **Single quotes:** When you want exact text, no interpretation
- **Double quotes:** When you need variable values, command output

---

### Question 2: Creating Aliases Permanently

**How to create alias permanently? How to check existing aliases?**

**Creating permanent alias:**

**Step 1: Edit ~/.bashrc**
```bash
vim ~/.bashrc
```

**Step 2: Add alias definition**
```bash
alias lst='ls -lt | head'
alias ll='ls -lah'
alias gs='git status'
```

**Step 3: Reload configuration**
```bash
source ~/.bashrc
```

**Or add without editing:**
```bash
echo "alias lst='ls -lt | head'" >> ~/.bashrc
source ~/.bashrc
```

**Why ~/.bashrc?**
- Executed every time BASH starts
- Applies to all new terminal sessions
- User-specific (in home directory)

---

**Checking existing aliases:**

**Method 1: Show all aliases**
```bash
alias
```

**Output:**
```
alias ll='ls -lah'
alias ls='ls --color=auto'
alias lst='ls -lt | head'
```

**Method 2: Check specific alias**
```bash
alias lst
# Output: alias lst='ls -lt | head'
```

**Method 3: Check if alias exists**
```bash
type lst
# Output: lst is aliased to `ls -lt | head'

type nonexistent
# Output: bash: type: nonexistent: not found
```

**Temporary vs Permanent:**

| Method | Scope | Survives logout? |
|--------|-------|------------------|
| `alias name='cmd'` | Current session | No |
| Add to `~/.bashrc` | All future sessions | Yes |

---

### Question 3: Environment Variables

**What are environment variables? How to create them?**

**Definition:**

**Environment variables** are named values that:
- Store system-wide or user-specific configuration
- Are inherited by child processes
- Affect program behavior

**Common environment variables:**

| Variable | Purpose | Example |
|----------|---------|---------|
| `HOME` | User's home directory | `/home/student` |
| `USER` | Current username | `student` |
| `PATH` | Command search directories | `/usr/bin:/bin` |
| `SHELL` | Current shell | `/bin/bash` |
| `EDITOR` | Default text editor | `vim` |
| `LANG` | Language/locale | `en_US.UTF-8` |

---

**Creating environment variables:**

**Method 1: Export command**
```bash
export MY_VAR="value"
```

**Method 2: Declare then export**
```bash
MY_VAR="value"
export MY_VAR
```

**Method 3: One-liner**
```bash
export DB_HOST="localhost" DB_PORT="5432"
```

---

**Permanent environment variables:**

**Add to ~/.bashrc:**
```bash
echo 'export MY_EDITOR=vim' >> ~/.bashrc
source ~/.bashrc
```

---

**Viewing environment variables:**

**All environment variables:**
```bash
env
# Or:
printenv
```

**Specific variable:**
```bash
echo $HOME
echo $PATH
printenv HOME
```

**Difference from regular variables:**

```bash
# Regular variable (not exported)
MY_VAR="test"
bash              # Start new shell
echo $MY_VAR      # Empty! Not inherited

# Environment variable (exported)
export MY_VAR="test"
bash              # Start new shell
echo $MY_VAR      # Output: test (inherited!)
```

**Key uses:**
- Configure programs (e.g., `EDITOR`, `PAGER`)
- Modify system behavior (e.g., `PATH`)
- Store credentials (e.g., `API_KEY`)
- Set locale (e.g., `LANG`, `LC_ALL`)

---

## Task Solutions

### Task 1: Permanent alias for recently modified files

**Objective:** Create permanent alias `lst` showing recently modified files.

**Solution:**

**Step 1: Design the command**
```bash
ls -lt | head
```

**Explanation:**
- `ls -lt` - List files, sorted by modification time (newest first)
- `head` - Show first 10 lines (most recent files)

**Step 2: Create alias**
```bash
alias lst='ls -lt | head'
```

**Step 3: Test it**
```bash
lst
```

**Example output:**
```
total 48
-rw-r--r-- 1 student student  234 Dec 16 14:30 recent_file.txt
-rw-r--r-- 1 student student  456 Dec 16 14:15 another.txt
drwxr-xr-x 2 student student 4096 Dec 16 13:45 mydir
...
```

**Step 4: Make permanent**
```bash
echo "alias lst='ls -lt | head'" >> ~/.bashrc
source ~/.bashrc
```

**Verification:**
```bash
# Check alias exists
alias lst

# Test in new shell
bash
lst
```

**Alternative versions:**

**Show top 5:**
```bash
alias lst='ls -lt | head -5'
```

**Show only files (no directories):**
```bash
alias lst='ls -lt | grep "^-" | head'
```

**Human-readable sizes:**
```bash
alias lst='ls -lht | head'
```

---

### Task 2: Custom Two-Line Prompt

**Objective:** Create 2-line prompt showing directory, background jobs, and $ symbol.

**Solution:**

**Desired prompt format:**
```
/home/student/project [2]
$
```

**Step 1: Design PS1 variable**

**Components needed:**
- `\w` - Current directory (full path)
- `\j` - Number of background jobs
- `\n` - Newline
- `\$` - `$` symbol (or `#` for root)

**PS1 definition:**
```bash
PS1="\w [\j]\n\$ "
```

**Step 2: Test temporarily**
```bash
export PS1="\w [\j]\n\$ "
```

**Your prompt changes immediately:**
```
/home/student [0]
$ cd project
/home/student/project [0]
$ sleep 100 &
[1] 12345
/home/student/project [1]
$
```

**Step 3: Make permanent**
```bash
echo 'export PS1="\w [\j]\n\$ "' >> ~/.bashrc
source ~/.bashrc
```

**Verification:**
```bash
# Logout and login again
exit

# SSH back in
ssh student@server

# Prompt should be 2-line
```

---

**Enhanced version with colors:**

```bash
PS1="\[\e[34m\]\w\[\e[0m\] \[\e[33m\][\j]\[\e[0m\]\n\$ "
```

**Colors:**
- Blue directory
- Yellow job count
- Normal rest

**To add:**
```bash
echo 'export PS1="\[\e[34m\]\w\[\e[0m\] \[\e[33m\][\j]\[\e[0m\]\n\$ "' >> ~/.bashrc
source ~/.bashrc
```

---

**Testing background jobs:**

```bash
sleep 100 &
sleep 200 &
jobs
# Prompt shows [2]
```

---

### Task 3: Rename .JPG to .jpg

**Objective:** Rename all .JPG files to .jpg in ~/lab6 directory.

**Setup:**
```bash
mkdir -p ~/lab6
# Copy files from /home/wojnicki/lab/zasoby
cp /home/wojnicki/lab/zasoby/*.JPG ~/lab6/
cd ~/lab6
ls
```

**Example files:**
```
photo1.JPG
photo2.JPG
image.JPG
```

**Solution:**

```bash
for file in *.JPG; do
    # Get filename without extension
    base=$(basename "$file" .JPG)
    # Rename to lowercase extension
    mv "$file" "${base}.jpg"
done
```

**Step-by-step breakdown:**

**1. Loop through all .JPG files:**
```bash
for file in *.JPG; do
```

**2. Extract base name (without .JPG):**
```bash
base=$(basename "$file" .JPG)
```

Example: `basename photo1.JPG .JPG` → `photo1`

**3. Rename file:**
```bash
mv "$file" "${base}.jpg"
```

Example: `mv photo1.JPG photo1.jpg`

**Complete command (one-liner):**
```bash
for file in *.JPG; do mv "$file" "$(basename "$file" .JPG).jpg"; done
```

**Verification:**
```bash
ls
# Output:
# photo1.jpg
# photo2.jpg
# image.jpg
```

**Safer version (check before overwriting):**
```bash
for file in *.JPG; do
    base=$(basename "$file" .JPG)
    newname="${base}.jpg"
    if [ -e "$newname" ]; then
        echo "Warning: $newname already exists, skipping"
    else
        mv "$file" "$newname"
        echo "Renamed: $file -> $newname"
    fi
done
```

---

### Task 4: Create JPG2jpg Command

**Objective:** Create executable command `JPG2jpg` with options.

**Requirements:**
- No argument: Convert in current directory
- One argument (directory): Convert in that directory
- `-h` option: Show help

**Solution:**

**Step 1: Create ~/bin directory**
```bash
mkdir -p ~/bin
```

**Step 2: Create script**
```bash
vim ~/bin/JPG2jpg
```

**Script content:**
```bash
#!/bin/bash

# JPG2jpg - Convert .JPG files to .jpg

# Function to convert files in directory
convert_directory() {
    local dir="$1"
    local count=0
    
    cd "$dir" || exit 1
    
    for file in *.JPG; do
        # Check if any .JPG files exist
        if [ ! -e "$file" ]; then
            echo "No .JPG files found in $dir"
            return
        fi
        
        base=$(basename "$file" .JPG)
        mv "$file" "${base}.jpg"
        echo "Converted: $file -> ${base}.jpg"
        count=$((count + 1))
    done
    
    echo "Total converted: $count files"
}

# Show help
show_help() {
    cat << EOF
JPG2jpg - Convert .JPG files to .jpg extension

Usage:
    JPG2jpg         Convert .JPG files in current directory
    JPG2jpg DIR     Convert .JPG files in specified directory
    JPG2jpg -h      Show this help message

Examples:
    JPG2jpg
    JPG2jpg ~/photos
    JPG2jpg /tmp/images

Description:
    Renames all files ending in .JPG to .jpg (lowercase extension)
    Does not modify the actual image data, only the filename
EOF
}

# Main logic
case "$1" in
    -h|--help)
        show_help
        exit 0
        ;;
    "")
        # No argument - use current directory
        convert_directory "."
        ;;
    *)
        # Check if argument is a directory
        if [ -d "$1" ]; then
            convert_directory "$1"
        else
            echo "Error: $1 is not a directory" >&2
            echo "Use 'JPG2jpg -h' for help" >&2
            exit 1
        fi
        ;;
esac
```

**Step 3: Make executable**
```bash
chmod +x ~/bin/JPG2jpg
```

**Step 4: Add ~/bin to PATH**

**Check if already in PATH:**
```bash
echo $PATH | grep "$HOME/bin"
```

**If not in PATH, add to ~/.bashrc:**
```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Verification:**
```bash
which JPG2jpg
# Output: /home/student/bin/JPG2jpg
```

---

**Usage examples:**

**1. Show help:**
```bash
JPG2jpg -h
```

**2. Convert current directory:**
```bash
cd ~/lab6
JPG2jpg
```

**Output:**
```
Converted: photo1.JPG -> photo1.jpg
Converted: photo2.JPG -> photo2.jpg
Total converted: 2 files
```

**3. Convert specific directory:**
```bash
JPG2jpg ~/lab6
```

**Output:**
```
Converted: photo1.JPG -> photo1.jpg
Converted: photo2.JPG -> photo2.jpg
Total converted: 2 files
```

**4. Error handling:**
```bash
JPG2jpg /nonexistent
# Output: Error: /nonexistent is not a directory
```

---

### Task 5: Remove Non-JPEG Files

**Objective:** Delete files in ~/lab6 that are not JPEG images.

**Solution:**

```bash
cd ~/lab6

for file in *; do
    # Skip if not a regular file
    if [ ! -f "$file" ]; then
        continue
    fi
    
    # Check file type
    filetype=$(file "$file")
    
    # If not JPEG, remove it
    if ! echo "$filetype" | grep -q "JPEG"; then
        echo "Removing non-JPEG: $file"
        rm "$file"
    else
        echo "Keeping JPEG: $file"
    fi
done
```

**Step-by-step breakdown:**

**1. Loop through all files:**
```bash
for file in *; do
```

**2. Skip directories:**
```bash
if [ ! -f "$file" ]; then
    continue
fi
```

**3. Check file type:**
```bash
filetype=$(file "$file")
```

Example output of `file`:
- `image.jpg: JPEG image data`
- `document.txt: ASCII text`

**4. Check if JPEG:**
```bash
if ! echo "$filetype" | grep -q "JPEG"; then
```

**5. Remove if not JPEG:**
```bash
rm "$file"
```

---

**Safer version (dry run):**

```bash
for file in *; do
    if [ ! -f "$file" ]; then
        continue
    fi
    
    if ! file "$file" | grep -q "JPEG"; then
        echo "Would remove: $file ($(file -b "$file"))"
        # Uncomment to actually delete:
        # rm "$file"
    fi
done
```

**One-liner version:**
```bash
for f in *; do [ -f "$f" ] && ! file "$f" | grep -q JPEG && rm "$f"; done
```

**Example scenario:**

**Before:**
```
$ ls -l
photo1.jpg     # JPEG
photo2.jpg     # JPEG
readme.txt     # Text file
data.csv       # CSV file
fake.jpg       # Actually a PNG with wrong extension
```

**Check file types:**
```bash
file *
# photo1.jpg: JPEG image data
# photo2.jpg: JPEG image data
# readme.txt: ASCII text
# data.csv: ASCII text
# fake.jpg: PNG image data
```

**Run script:**

**After:**
```
$ ls -l
photo1.jpg     # Kept (JPEG)
photo2.jpg     # Kept (JPEG)
# readme.txt removed (not JPEG)
# data.csv removed (not JPEG)
# fake.jpg removed (PNG, not JPEG)
```

---

### Task 6: Reminder System

**Objective:** Create reminder system that displays messages on login for today's date.

**Part A: Create reminder file**

**Create ~/reminder:**
```bash
vim ~/reminder
```

**Format:** `YYYY-MM-DD text to remind`

**Example content:**
```
2024-12-20 Submit lab report
2024-12-25 Christmas!
2025-01-01 New Year
2024-12-16 Today's reminder test
```

**Part B: Create reminder script**

```bash
vim ~/check_reminders.sh
```

**Script content:**
```bash
#!/bin/bash

# check_reminders.sh - Display reminders for today

REMINDER_FILE="$HOME/reminder"

# Check if reminder file exists
if [ ! -f "$REMINDER_FILE" ]; then
    exit 0  # Silently exit if no reminders
fi

# Get today's date in YYYY-MM-DD format
today=$(date +%Y-%m-%d)

# Check for reminders matching today's date
while IFS= read -r line; do
    # Extract date (first field)
    reminder_date=$(echo "$line" | cut -d' ' -f1)
    
    # Check if date matches today
    if [ "$reminder_date" = "$today" ]; then
        # Extract reminder text (everything after first space)
        reminder_text=$(echo "$line" | cut -d' ' -f2-)
        
        echo "=== REMINDER ==="
        echo "Date: $reminder_date"
        echo "Message: $reminder_text"
        echo "================"
    fi
done < "$REMINDER_FILE"
```

**Make executable:**
```bash
chmod +x ~/check_reminders.sh
```

**Test manually:**
```bash
~/check_reminders.sh
```

**Output (if today is 2024-12-16):**
```
=== REMINDER ===
Date: 2024-12-16
Message: Today's reminder test
================
```

---

**Part C: Auto-run on login**

**Add to ~/.bashrc:**
```bash
echo '~/check_reminders.sh' >> ~/.bashrc
```

**Or with header:**
```bash
cat >> ~/.bashrc << 'EOF'

# Check for reminders
if [ -f ~/check_reminders.sh ]; then
    ~/check_reminders.sh
fi
EOF
```

**Reload:**
```bash
source ~/.bashrc
```

**Test:**
```bash
# Logout and login
exit
ssh student@server

# Reminders should display automatically
```

---

**Enhanced version with colors:**

```bash
#!/bin/bash

REMINDER_FILE="$HOME/reminder"

if [ ! -f "$REMINDER_FILE" ]; then
    exit 0
fi

today=$(date +%Y-%m-%d)
found=0

while IFS= read -r line; do
    reminder_date=$(echo "$line" | cut -d' ' -f1)
    
    if [ "$reminder_date" = "$today" ]; then
        if [ $found -eq 0 ]; then
            echo -e "\n\e[33m╔════════════════════════════════════╗\e[0m"
            echo -e "\e[33m║         TODAY'S REMINDERS          ║\e[0m"
            echo -e "\e[33m╚════════════════════════════════════╝\e[0m"
        fi
        
        reminder_text=$(echo "$line" | cut -d' ' -f2-)
        echo -e "\e[32m➤\e[0m $reminder_text"
        found=1
    fi
done < "$REMINDER_FILE"

if [ $found -eq 1 ]; then
    echo ""
fi
```

---

### Task 7: Advanced File Pagination Script (*Optional)

**Objective:** Create `stronnicuj` command to split directory files into paginated groups.

**Arguments:**
1. Source directory (default: current)
2. Max files per page (default: 10)
3. Preserve structure: true/false (default: false)
4. Destination directory (default: {source}_stronnicowany)

**Solution:**

```bash
vim ~/bin/stronnicuj
```

**Script content:**
```bash
#!/bin/bash

# stronnicuj - Paginate files from directory into smaller groups

# Default values
SOURCE_DIR="."
MAX_FILES=10
PRESERVE_STRUCTURE=false
DEST_DIR=""

# Show help
show_help() {
    cat << EOF
stronnicuj - Paginate directory files into smaller groups

Usage:
    stronnicuj [SOURCE_DIR] [MAX_FILES] [PRESERVE_STRUCTURE] [DEST_DIR]

Arguments:
    SOURCE_DIR           Source directory (default: current directory)
    MAX_FILES           Maximum files per page (default: 10)
    PRESERVE_STRUCTURE  Keep directory structure: true/false (default: false)
    DEST_DIR            Destination directory (default: {source}_stronnicowany)

Examples:
    stronnicuj
    stronnicuj ~/photos 20
    stronnicuj ~/docs 15 true
    stronnicuj ~/src 10 false ~/backup

Description:
    Splits files from source directory into paginated subdirectories (1, 2, 3, ...)
    - If PRESERVE_STRUCTURE=true: maintains directory hierarchy
    - If PRESERVE_STRUCTURE=false: flattens all files into pages
EOF
}

# Parse arguments
if [ "$1" = "-h" ] || [ "$1" = "--help" ]; then
    show_help
    exit 0
fi

# Set source directory
if [ -n "$1" ]; then
    SOURCE_DIR="$1"
fi

# Set max files per page
if [ -n "$2" ]; then
    MAX_FILES="$2"
fi

# Set preserve structure flag
if [ -n "$3" ]; then
    if [ "$3" = "true" ]; then
        PRESERVE_STRUCTURE=true
    elif [ "$3" = "false" ]; then
        PRESERVE_STRUCTURE=false
    else
        echo "Error: PRESERVE_STRUCTURE must be 'true' or 'false'" >&2
        exit 1
    fi
fi

# Set destination directory
if [ -n "$4" ]; then
    DEST_DIR="$4"
else
    # Default: {source_name}_stronnicowany
    SOURCE_BASENAME=$(basename "$SOURCE_DIR")
    DEST_DIR="${SOURCE_BASENAME}_stronnicowany"
fi

# Validate source directory
if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: Source directory '$SOURCE_DIR' does not exist" >&2
    exit 1
fi

# Create destination directory
mkdir -p "$DEST_DIR"

echo "Source: $SOURCE_DIR"
echo "Destination: $DEST_DIR"
echo "Max files per page: $MAX_FILES"
echo "Preserve structure: $PRESERVE_STRUCTURE"
echo ""

# Find all files recursively
page_num=1
file_count=0

if [ "$PRESERVE_STRUCTURE" = "true" ]; then
    # Preserve directory structure
    find "$SOURCE_DIR" -type f | while read -r filepath; do
        # Get relative path
        relpath="${filepath#$SOURCE_DIR/}"
        
        # Calculate page number
        page_num=$(( (file_count / MAX_FILES) + 1 ))
        
        # Create destination path
        dest_path="$DEST_DIR/$page_num/$relpath"
        dest_dir=$(dirname "$dest_path")
        
        # Create directory structure
        mkdir -p "$dest_dir"
        
        # Copy file
        cp "$filepath" "$dest_path"
        echo "Copied to page $page_num: $relpath"
        
        file_count=$((file_count + 1))
    done
else
    # Flatten structure
    find "$SOURCE_DIR" -type f | while read -r filepath; do
        # Get just filename
        filename=$(basename "$filepath")
        
        # Calculate page number
        page_num=$(( (file_count / MAX_FILES) + 1 ))
        
        # Create page directory
        mkdir -p "$DEST_DIR/$page_num"
        
        # Copy file
        cp "$filepath" "$DEST_DIR/$page_num/$filename"
        echo "Copied to page $page_num: $filename"
        
        file_count=$((file_count + 1))
    done
fi

echo ""
echo "Done! Copied $file_count files into $page_num pages"
```

**Make executable:**
```bash
chmod +x ~/bin/stronnicuj
```

---

**Usage examples:**

**Example 1: Paginate current directory (default 10 files/page):**
```bash
mkdir test_dir
cd test_dir
touch file{1..35}.txt
cd ..

stronnicuj test_dir
```

**Result:**
```
test_dir_stronnicowany/
├── 1/
│   ├── file1.txt
│   ├── file2.txt
│   └── ... (10 files)
├── 2/
│   └── ... (10 files)
├── 3/
│   └── ... (10 files)
└── 4/
    └── ... (5 files)
```

---

**Example 2: Custom page size:**
```bash
stronnicuj ~/photos 20
```

---

**Example 3: Preserve directory structure:**
```bash
mkdir -p src/core src/utils
touch src/core/file{1..15}.h
touch src/utils/util{1..8}.c

stronnicuj src 10 true
```

**Result:**
```
src_stronnicowany/
├── 1/
│   ├── core/
│   │   ├── file1.h
│   │   └── ... (10 files total)
├── 2/
│   ├── core/
│   │   └── ... (remaining core files)
│   └── utils/
│       └── ... (utils files)
```

---

**Example 4: Flatten structure:**
```bash
stronnicuj src 10 false
```

**Result:**
```
src_stronnicowany/
├── 1/
│   ├── file1.h
│   ├── file2.h
│   ├── util1.c
│   └── ... (10 files mixed)
├── 2/
│   └── ... (10 files)
```

---

## Summary Question

### How to create custom Unix command?

**Steps to create custom command accessible as `JPG2jpg`:**

**1. Create script file:**
```bash
vim ~/bin/JPG2jpg
```

**2. Add shebang:**
```bash
#!/bin/bash
```

**3. Write script logic:**
```bash
#!/bin/bash
echo "My custom command"
```

**4. Make executable:**
```bash
chmod +x ~/bin/JPG2jpg
```

**5. Ensure ~/bin in PATH:**
```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**6. Use command from anywhere:**
```bash
JPG2jpg
```

---

**Key requirements:**
1. **Script file** with shebang
2. **Execute permission** (`chmod +x`)
3. **Located in PATH directory** (like `~/bin`, `/usr/local/bin`)
4. **No extension needed** (not `JPG2jpg.sh`, just `JPG2jpg`)

**Verification:**
```bash
which JPG2jpg          # Shows path
type JPG2jpg           # Shows it's a file
JPG2jpg --help         # Run it
```

---

## Common Mistakes

### 1. Forgetting to Quote Variables

```bash
# Wrong - breaks with spaces
for file in $files; do

# Correct
for file in "$files"; do
```

### 2. Spaces in Variable Assignment

```bash
# Wrong
var = "value"

# Correct
var="value"
```

### 3. Using = for Number Comparison

```bash
# Wrong
if [ $count = 5 ]; then

# Correct
if [ $count -eq 5 ]; then
```

### 4. Not Making Script Executable

```bash
# Create script
vim myscript.sh

# Wrong
./myscript.sh    # Permission denied

# Correct
chmod +x myscript.sh
./myscript.sh
```

### 5. Forgetting Shebang

```bash
# Wrong (no shebang)
echo "Hello"

# Correct
#!/bin/bash
echo "Hello"
```

### 6. Not Adding ~/bin to PATH

```bash
# Create command in ~/bin
chmod +x ~/bin/mycommand

# Won't work if ~/bin not in PATH
mycommand    # Command not found

# Fix:
export PATH="$HOME/bin:$PATH"
```

---

## Key Takeaways

1. **Single quotes** preserve literal text, **double quotes** allow variable expansion
2. **Permanent aliases** go in `~/.bashrc`
3. **Environment variables** use `export` and are inherited by child processes
4. **Custom commands** need: shebang, execute permission, location in PATH
5. **basename** extracts filename from path
6. **file** command detects file type
7. **PS1** variable controls prompt appearance
8. **Loops** process multiple files or items
9. **Conditionals** make decisions based on tests
10. **Functions** organize reusable code

BASH scripting transforms repetitive tasks into automated tools!
