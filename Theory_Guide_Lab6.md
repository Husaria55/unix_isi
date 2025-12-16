# Theory Guide - Lab 6: BASH Scripting Basics

## Table of Contents
1. [Introduction to BASH Scripting](#introduction-to-bash-scripting)
2. [Variables](#variables)
3. [Quoting](#quoting)
4. [Special Variables](#special-variables)
5. [Conditional Statements](#conditional-statements)
6. [Loops](#loops)
7. [Functions](#functions)
8. [Aliases](#aliases)
9. [Custom Commands](#custom-commands)
10. [BASH Prompt Customization](#bash-prompt-customization)
11. [Command Reference](#command-reference)

---

## Introduction to BASH Scripting

### What is a BASH Script?

A **BASH script** is a text file containing a sequence of commands that the BASH shell can execute.

**Basic script structure:**
```bash
#!/bin/bash
# This is a comment

echo "Hello, World!"
```

**Key elements:**
- **Shebang (`#!/bin/bash`)**: Tells system which interpreter to use
- **Comments**: Lines starting with `#`
- **Commands**: Any command you can run in terminal

### Creating and Running Scripts

**Create script:**
```bash
vim myscript.sh
```

**Make executable:**
```bash
chmod +x myscript.sh
```

**Run script:**
```bash
./myscript.sh        # From current directory
bash myscript.sh     # Explicitly with bash
```

---

## Variables

### Declaring Variables

**Syntax:**
```bash
variable_name=value
```

**⚠️ Important:** No spaces around `=`!

**Examples:**
```bash
# Correct
name="John"
count=5

# Wrong
name = "John"     # Error!
count= 5          # Error!
```

### Using Variables

**Access with `$`:**
```bash
name="Alice"
echo $name           # Alice
echo "Hello, $name"  # Hello, Alice
```

**Curly braces (recommended for clarity):**
```bash
name="Alice"
echo "Hello, ${name}!"     # Hello, Alice!
echo "File: ${name}.txt"   # File: Alice.txt
```

### Variable Types

**String variables:**
```bash
greeting="Hello"
user="student"
```

**Numeric variables (treated as strings by default):**
```bash
count=10
result=$((count + 5))   # Arithmetic expansion
echo $result            # 15
```

### Environment Variables

**Definition:** Variables available to all processes spawned from current shell.

**Export variable:**
```bash
export MY_VAR="value"
```

**Common environment variables:**
| Variable | Description |
|----------|-------------|
| `HOME` | User's home directory |
| `USER` | Current username |
| `PATH` | Command search path |
| `PWD` | Current working directory |
| `SHELL` | Current shell |

**View environment variables:**
```bash
env             # All environment variables
printenv        # All environment variables
echo $HOME      # Specific variable
```

**Example:**
```bash
export EDITOR=vim
$EDITOR myfile.txt    # Opens vim
```

---

## Quoting

### Single Quotes vs Double Quotes

**Single quotes (`'`)**: Literal string, **no variable expansion**

```bash
name="Alice"
echo 'Hello, $name'    # Output: Hello, $name
```

**Double quotes (`"`)**: Variable expansion **enabled**

```bash
name="Alice"
echo "Hello, $name"    # Output: Hello, Alice
```

### When to Use Each

**Single quotes:**
- Preserve exactly as written
- No interpretation of special characters

```bash
echo 'Price: $10'      # Output: Price: $10
echo 'Use $(command)'  # Output: Use $(command)
```

**Double quotes:**
- Allow variable substitution
- Allow command substitution
- Preserve spaces

```bash
message="Hello World"
echo "$message"        # Output: Hello World
echo "Today: $(date)"  # Output: Today: Mon Dec 16...
```

**No quotes (careful!):**
- Word splitting
- Glob expansion

```bash
files="*.txt"
echo $files            # Expands to: file1.txt file2.txt ...
echo "$files"          # Output: *.txt
```

### Escaping

**Backslash (`\`)**: Escape single character

```bash
echo "Price: \$10"     # Output: Price: $10
echo "Quote: \"text\"" # Output: Quote: "text"
```

---

## Special Variables

### Script Arguments

**Positional parameters:**

| Variable | Description |
|----------|-------------|
| `$0` | Script name (with path) |
| `$1` | First argument |
| `$2` | Second argument |
| `$3` | Third argument |
| ... | ... |
| `$9` | Ninth argument |
| `${10}` | Tenth argument (braces required) |

**Example script:**
```bash
#!/bin/bash
# save as greet.sh

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
```

**Usage:**
```bash
./greet.sh Alice Bob
# Output:
# Script name: ./greet.sh
# First argument: Alice
# Second argument: Bob
```

### Special Parameter Variables

| Variable | Description |
|----------|-------------|
| `$#` | Number of arguments |
| `$*` | All arguments as single string |
| `$@` | All arguments as separate strings |
| `$?` | Exit status of last command |
| `$$` | Process ID of current shell |
| `$!` | Process ID of last background command |

**Examples:**

**`$#` - Count arguments:**
```bash
#!/bin/bash
echo "You provided $# arguments"
```

```bash
./script.sh a b c
# Output: You provided 3 arguments
```

**`$*` vs `$@`:**
```bash
#!/bin/bash
echo "Using \$*:"
for arg in "$*"; do
    echo "  Arg: $arg"
done

echo "Using \$@:"
for arg in "$@"; do
    echo "  Arg: $arg"
done
```

```bash
./script.sh one two three
# Using $*:
#   Arg: one two three
# Using $@:
#   Arg: one
#   Arg: two
#   Arg: three
```

**`$?` - Exit status:**
```bash
ls /existing/path
echo $?    # 0 (success)

ls /nonexistent
echo $?    # 2 (error)
```

---

## Conditional Statements

### if Statement

**Basic syntax:**
```bash
if [ condition ]; then
    commands
fi
```

**if-else:**
```bash
if [ condition ]; then
    commands1
else
    commands2
fi
```

**if-elif-else:**
```bash
if [ condition1 ]; then
    commands1
elif [ condition2 ]; then
    commands2
else
    commands3
fi
```

### Test Conditions

**String comparisons:**
| Operator | Description |
|----------|-------------|
| `=` or `==` | Equal |
| `!=` | Not equal |
| `-z` | String is empty |
| `-n` | String is not empty |

**Examples:**
```bash
if [ "$name" = "Alice" ]; then
    echo "Hello Alice"
fi

if [ -z "$var" ]; then
    echo "Variable is empty"
fi
```

**Numeric comparisons:**
| Operator | Description |
|----------|-------------|
| `-eq` | Equal |
| `-ne` | Not equal |
| `-lt` | Less than |
| `-le` | Less than or equal |
| `-gt` | Greater than |
| `-ge` | Greater than or equal |

**Examples:**
```bash
if [ $count -gt 10 ]; then
    echo "Count is greater than 10"
fi
```

**File tests:**
| Operator | Description |
|----------|-------------|
| `-e` | File exists |
| `-f` | Regular file exists |
| `-d` | Directory exists |
| `-r` | File is readable |
| `-w` | File is writable |
| `-x` | File is executable |
| `-s` | File exists and not empty |

**Examples:**
```bash
if [ -f "file.txt" ]; then
    echo "File exists"
fi

if [ -d "mydir" ]; then
    echo "Directory exists"
fi
```

**Logical operators:**
| Operator | Description |
|----------|-------------|
| `-a` or `&&` | AND |
| `-o` or `\|\|` | OR |
| `!` | NOT |

**Examples:**
```bash
if [ -f "file.txt" -a -r "file.txt" ]; then
    echo "File exists and is readable"
fi

if [ "$name" = "Alice" ] || [ "$name" = "Bob" ]; then
    echo "Hello Alice or Bob"
fi
```

### Modern Test Syntax

**Double brackets `[[ ]]`** (recommended):

**Advantages:**
- Pattern matching
- No word splitting
- More forgiving syntax

```bash
if [[ $name == "Alice" ]]; then
    echo "Hello Alice"
fi

if [[ $file == *.txt ]]; then
    echo "Text file"
fi
```

---

## Loops

### for Loop

**Syntax:**
```bash
for variable in list; do
    commands
done
```

**Examples:**

**Iterate over list:**
```bash
for name in Alice Bob Charlie; do
    echo "Hello, $name"
done
```

**Iterate over files:**
```bash
for file in *.txt; do
    echo "Processing: $file"
done
```

**Iterate over command output:**
```bash
for user in $(cat users.txt); do
    echo "User: $user"
done
```

**C-style for loop:**
```bash
for ((i=1; i<=5; i++)); do
    echo "Number: $i"
done
```

### while Loop

**Syntax:**
```bash
while [ condition ]; do
    commands
done
```

**Example:**
```bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    count=$((count + 1))
done
```

**Reading file line by line:**
```bash
while read line; do
    echo "Line: $line"
done < file.txt
```

**Infinite loop:**
```bash
while true; do
    echo "Running..."
    sleep 1
done
```

### Loop Control

**`break`**: Exit loop
```bash
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break
    fi
    echo $i
done
```

**`continue`**: Skip to next iteration
```bash
for i in {1..5}; do
    if [ $i -eq 3 ]; then
        continue
    fi
    echo $i
done
# Output: 1 2 4 5
```

---

## Functions

### Defining Functions

**Syntax 1:**
```bash
function_name() {
    commands
}
```

**Syntax 2:**
```bash
function function_name {
    commands
}
```

**Example:**
```bash
greet() {
    echo "Hello, $1!"
}

greet Alice    # Output: Hello, Alice!
```

### Function Arguments

Functions use same argument variables as scripts:

```bash
add() {
    result=$(($1 + $2))
    echo $result
}

sum=$(add 5 3)
echo "Sum: $sum"    # Sum: 8
```

### Return Values

**`return`**: Exit status (0-255)
```bash
is_even() {
    if [ $(($1 % 2)) -eq 0 ]; then
        return 0    # True
    else
        return 1    # False
    fi
}

if is_even 4; then
    echo "Even number"
fi
```

**`echo`**: Output value
```bash
get_greeting() {
    echo "Hello, $1"
}

message=$(get_greeting "World")
echo $message    # Hello, World
```

---

## Aliases

### What are Aliases?

**Aliases** are shortcuts for commands.

**Syntax:**
```bash
alias name='command'
```

**Examples:**
```bash
alias ll='ls -la'
alias gs='git status'
alias ..='cd ..'
```

### Creating Aliases

**Temporary (current session only):**
```bash
alias ll='ls -la'
```

**Permanent (survives logout):**

Add to `~/.bashrc`:
```bash
echo "alias ll='ls -la'" >> ~/.bashrc
source ~/.bashrc
```

### Viewing Aliases

**All aliases:**
```bash
alias
```

**Specific alias:**
```bash
alias ll
```

### Removing Aliases

**Current session:**
```bash
unalias ll
```

**Permanent:**
Remove line from `~/.bashrc`

### Common Useful Aliases

```bash
alias ll='ls -lah'
alias la='ls -A'
alias l='ls -CF'
alias grep='grep --color=auto'
alias ..='cd ..'
alias ...='cd ../..'
alias h='history'
alias c='clear'
```

---

## Custom Commands

### Creating Custom Commands

**Steps to create system-wide accessible command:**

1. **Create script file**
2. **Add shebang**
3. **Make executable**
4. **Place in PATH directory**

### Using ~/bin for Custom Commands

**Setup:**
```bash
# Create ~/bin directory
mkdir -p ~/bin

# Add to PATH (in ~/.bashrc)
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

**Create command:**
```bash
# Create script
vim ~/bin/mycommand

# Content:
#!/bin/bash
echo "My custom command!"

# Make executable
chmod +x ~/bin/mycommand

# Use it:
mycommand
```

**Now `mycommand` can be run from anywhere!**

### The `basename` Command

**Purpose:** Extract filename from path.

**Syntax:**
```bash
basename PATH [SUFFIX]
```

**Examples:**
```bash
basename /home/user/file.txt
# Output: file.txt

basename /home/user/file.txt .txt
# Output: file

# In script:
for path in /home/user/*.txt; do
    filename=$(basename "$path")
    echo "Processing: $filename"
done
```

### The `file` Command

**Purpose:** Determine file type.

**Syntax:**
```bash
file filename
```

**Examples:**
```bash
file image.jpg
# Output: image.jpg: JPEG image data

file script.sh
# Output: script.sh: Bourne-Again shell script

file document.pdf
# Output: document.pdf: PDF document

# In script:
if file "$filename" | grep -q "JPEG"; then
    echo "It's a JPEG image"
fi
```

---

## BASH Prompt Customization

### The PS1 Variable

**PS1** controls the primary prompt appearance.

**Default:**
```bash
echo $PS1
# Might show: \u@\h:\w\$
```

### Prompt Escape Sequences

| Sequence | Description |
|----------|-------------|
| `\u` | Username |
| `\h` | Hostname (short) |
| `\H` | Hostname (full) |
| `\w` | Current directory (full path) |
| `\W` | Current directory (basename only) |
| `\d` | Date |
| `\t` | Time (24-hour) |
| `\T` | Time (12-hour) |
| `\n` | Newline |
| `\$` | `#` if root, `$` otherwise |
| `\j` | Number of jobs |

### Custom Prompt Examples

**Simple prompt:**
```bash
PS1="\u@\h:\w\$ "
# Output: student@server:~/project$
```

**Two-line prompt:**
```bash
PS1="\u@\h:\w\n\$ "
# Output:
# student@server:~/project
# $
```

**With time:**
```bash
PS1="[\t] \u@\h:\w\$ "
# Output: [14:30:45] student@server:~/project$
```

**Colorized prompt:**
```bash
PS1="\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ "
# Green username, blue directory
```

**Color codes:**
| Code | Color |
|------|-------|
| `\e[30m` | Black |
| `\e[31m` | Red |
| `\e[32m` | Green |
| `\e[33m` | Yellow |
| `\e[34m` | Blue |
| `\e[35m` | Magenta |
| `\e[36m` | Cyan |
| `\e[37m` | White |
| `\e[0m` | Reset |

### Permanent Prompt Configuration

**Add to ~/.bashrc:**
```bash
echo 'PS1="\u@\h:\w\n\$ "' >> ~/.bashrc
source ~/.bashrc
```

### Number of Background Jobs

**Display job count:**
```bash
PS1="\w [\j]\$ "
# Output: ~/project [2]$
```

---

## Command Reference

### Scripting Basics

```bash
#!/bin/bash                # Shebang
variable=value             # Assign variable
$variable                  # Use variable
"$variable"                # Safe variable use
'$variable'                # Literal (no expansion)
```

### Script Arguments

```bash
$0                         # Script name
$1, $2, ...                # Arguments
$#                         # Argument count
$*                         # All arguments (string)
$@                         # All arguments (array)
$?                         # Last exit status
```

### Conditionals

```bash
if [ condition ]; then
    commands
fi

if [[ condition ]]; then   # Modern syntax
    commands
fi
```

### Loops

```bash
for var in list; do
    commands
done

while [ condition ]; do
    commands
done
```

### Functions

```bash
function_name() {
    commands
}

function_name arg1 arg2
```

### Aliases

```bash
alias name='command'       # Create alias
alias                      # List aliases
unalias name               # Remove alias
```

### Utilities

```bash
basename /path/file.txt    # Get filename
file myfile                # Detect file type
```

---

## Best Practices

### 1. Use Meaningful Variable Names

```bash
# Good
username="Alice"
file_count=10

# Bad
a="Alice"
x=10
```

### 2. Quote Variables

```bash
# Good
if [ "$name" = "Alice" ]; then
    echo "$name"
fi

# Bad (can break with spaces)
if [ $name = Alice ]; then
    echo $name
fi
```

### 3. Check Arguments

```bash
#!/bin/bash
if [ $# -lt 1 ]; then
    echo "Usage: $0 filename"
    exit 1
fi
```

### 4. Use Functions for Repeated Code

```bash
print_error() {
    echo "ERROR: $1" >&2
}

print_error "File not found"
print_error "Invalid input"
```

### 5. Add Comments

```bash
# Calculate total from list
total=0
for num in $numbers; do
    total=$((total + num))
done
```

### 6. Make Scripts Portable

```bash
#!/bin/bash
# Use $(command) instead of `command`
date_string=$(date +%Y%m%d)
```

### 7. Error Handling

```bash
command
if [ $? -ne 0 ]; then
    echo "Command failed" >&2
    exit 1
fi

# Or:
command || { echo "Failed" >&2; exit 1; }
```

---

## Common Patterns

### Check if File Exists

```bash
if [ -f "file.txt" ]; then
    echo "File exists"
else
    echo "File not found"
    exit 1
fi
```

### Process All Files in Directory

```bash
for file in /path/to/dir/*; do
    if [ -f "$file" ]; then
        echo "Processing: $file"
    fi
done
```

### Read File Line by Line

```bash
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt
```

### Get User Input

```bash
read -p "Enter your name: " name
echo "Hello, $name"
```

### Arithmetic

```bash
result=$((5 + 3))
count=$((count + 1))
```

### String Manipulation

```bash
# Length
length=${#string}

# Substring
sub=${string:0:5}

# Replace
new=${string/old/new}
```

---

## Common Mistakes

### 1. Spaces in Assignment

```bash
# Wrong
var = 5

# Correct
var=5
```

### 2. Missing Quotes

```bash
filename="my file.txt"

# Wrong (breaks with spaces)
if [ -f $filename ]; then

# Correct
if [ -f "$filename" ]; then
```

### 3. Using = for Numbers

```bash
# Wrong
if [ $count = 5 ]; then

# Correct
if [ $count -eq 5 ]; then
```

### 4. Forgetting to Make Script Executable

```bash
# Create script
vim myscript.sh

# Wrong
./myscript.sh    # Permission denied

# Correct
chmod +x myscript.sh
./myscript.sh
```

### 5. Not Using Shebang

```bash
# Wrong (no shebang)
echo "Hello"

# Correct
#!/bin/bash
echo "Hello"
```

---

## Summary

Lab 6 covers BASH scripting fundamentals:

**Variables:**
- Declaration: `var=value`
- Usage: `$var` or `${var}`
- Environment: `export VAR=value`

**Quoting:**
- Single quotes: Literal (`'$var'` → `$var`)
- Double quotes: Expansion (`"$var"` → value)

**Special Variables:**
- `$0` - Script name
- `$1, $2, ...` - Arguments
- `$#` - Argument count
- `$?` - Exit status

**Control Flow:**
- Conditionals: `if [ condition ]; then ... fi`
- Loops: `for`, `while`
- Functions: `name() { ... }`

**Aliases:**
- Create: `alias name='command'`
- Permanent: Add to `~/.bashrc`

**Custom Commands:**
- Place in `~/bin`
- Add `~/bin` to PATH
- Make executable: `chmod +x`

**Utilities:**
- `basename` - Extract filename
- `file` - Detect file type

**Prompt:**
- Customize with PS1
- Escape sequences: `\u`, `\w`, `\n`, `\j`

**Key Principle:** BASH scripts automate repetitive tasks and create custom tools.
