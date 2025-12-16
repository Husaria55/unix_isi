# Model Answers & Walkthrough - Lab 3: Advanced Data Processing and File Searching

## Overview

This document provides solutions to Lab 3 exercises focusing on grep, find, file type detection, and error handling.

---

## Introductory Questions

### Q1: What file descriptors does every Linux program have?

**Answer:**

Every Linux program has three standard file descriptors:

| Descriptor | Name | Number | Purpose |
|------------|------|--------|---------|
| **stdin** | Standard Input | 0 | Data input (keyboard, pipes) |
| **stdout** | Standard Output | 1 | Normal output (terminal, files) |
| **stderr** | Standard Error | 2 | Error messages |

**Why three separate streams?**
- **Separate concerns:** Normal output vs errors
- **Independent redirection:** Handle each differently
- **Pipelines:** Errors don't contaminate data flow
- **Debugging:** Easy to isolate problems

**Example:**
```bash
# Both outputs visible
ls existing_file non_existing_file

# Separate them
ls existing_file non_existing_file > output.txt 2> errors.txt
```

---

### Q2: Describe all redirections

**Answer:**

#### Basic Redirections

**`>` - Redirect stdout (overwrite)**
```bash
command > file.txt
# Sends normal output to file, overwrites if exists
```

**`>>` - Redirect stdout (append)**
```bash
command >> file.txt
# Appends normal output to file
```

**`<` - Redirect stdin**
```bash
command < input.txt
# Reads input from file instead of keyboard
```

#### Error Redirections

**`2>` - Redirect stderr (overwrite)**
```bash
command 2> errors.txt
# Sends errors to file, overwrites if exists
```

**`2>>` - Redirect stderr (append)**
```bash
command 2>> errors.log
# Appends errors to file
```

#### Combined Redirections

**`2>&1` - Redirect stderr to stdout**
```bash
command > file.txt 2>&1
# Both stdout and stderr go to file.txt
# Order matters: stdout redirect must come first!
```

**`>&2` or `1>&2` - Redirect stdout to stderr**
```bash
echo "Error message" >&2
# Useful in scripts to send messages to stderr
```

**`&>` - Redirect both stdout and stderr (overwrite)**
```bash
command &> all_output.txt
# Modern syntax, both streams to file
```

**`&>>` - Redirect both stdout and stderr (append)**
```bash
command &>> all_output.log
# Append both streams to file
```

#### Advanced (Optional)

**`<<` - Here document**
```bash
cat << EOF
Multiple lines
of input
EOF
```

**`<<<` - Here string**
```bash
grep "pattern" <<< "search this string"
```

**Examples:**
```bash
# Separate output and errors
gcc program.c > output.txt 2> errors.txt

# Discard errors
find / -name "*.txt" 2> /dev/null

# Combine and pipe
command 2>&1 | grep "error"

# Save everything
./script.sh &> complete.log
```

---

## Exercise Solutions

### 1. Finding Lines Containing "jpg" (Case-Insensitive)

**Task:** Find all lines containing "jpg" in `~wojnicki/lab/Lab3.md`, ignoring case.

**Solution:**
```bash
grep -i "jpg" ~wojnicki/lab/Lab3.md
```

**Explanation:**
- `grep` searches for patterns in files
- `-i` makes search case-insensitive
- Matches: jpg, JPG, Jpg, jPg, etc.
- Displays entire lines containing the pattern

**Output example:**
```
Files ending in .JPG
Process all .jpg images
```

**Alternative methods:**
```bash
# Using extended regex
grep -iE "jpg" ~wojnicki/lab/Lab3.md

# Using cat and pipe
cat ~wojnicki/lab/Lab3.md | grep -i "jpg"
```

**Common mistakes:**
- Forgetting `-i` (misses JPG, Jpg, etc.)
- Using wrong filename/path

---

### 2. Finding Lines with Context (2 Lines Before)

**Task:** Show lines containing "jpg" with 2 lines of context before each match.

**Solution:**
```bash
grep -B 2 -i "jpg" ~wojnicki/lab/Lab3.md
```

**Explanation:**
- `-B 2` shows 2 lines **before** each match
- `-i` case-insensitive (from previous task)
- Helps understand context of matches

**Output example:**
```
This is line before before
This is line before
Files ending in .JPG
--
Another context line
Previous line
Process all .jpg images
```

**Note:** `--` separates different match groups.

**Alternative options:**
```bash
# 2 lines after
grep -A 2 -i "jpg" file

# 2 lines before AND after
grep -C 2 -i "jpg" file

# or equivalently:
grep -2 -i "jpg" file
```

**Common mistakes:**
- Using `-A` instead of `-B` (shows after, not before)
- Confusing number of lines

---

### 3. Adding Line Numbers to Output

**Task:** Show lines containing "jpg" with 2 lines before AND line numbers.

**Solution:**
```bash
grep -B 2 -n -i "jpg" ~wojnicki/lab/Lab3.md
```

**Explanation:**
- `-B 2` shows 2 lines before
- `-n` displays line numbers
- `-i` case-insensitive
- Line numbers help locate matches in file

**Output example:**
```
23-This is line before before
24-This is line before
25:Files ending in .JPG
--
42-Another context line
43-Previous line
44:Process all .jpg images
```

**Note:** 
- Context lines use `-` after line number
- Matching lines use `:` after line number

**Alternative:**
```bash
# Just matches with line numbers
grep -n -i "jpg" ~wojnicki/lab/Lab3.md
```

**Common mistakes:**
- Forgetting `-n` flag
- Misunderstanding `-` vs `:` in output

---

### 4. Counting Lines with Letter "k"

**Task:** Count how many lines contain the letter "k" in the lab description.

**Solution:**
```bash
grep -c "k" ~wojnicki/lab/Lab3.md
```

**Or case-insensitive (k or K):**
```bash
grep -i -c "k" ~wojnicki/lab/Lab3.md
```

**Explanation:**
- `grep` searches for pattern
- `-c` counts matching lines (not occurrences!)
- Outputs a single number

**Output example:**
```
42
```

**Alternative methods:**
```bash
# Manual count with wc
grep "k" ~wojnicki/lab/Lab3.md | wc -l

# Case-insensitive
grep -i "k" ~wojnicki/lab/Lab3.md | wc -l
```

**Important distinction:**
```bash
# Counts LINES with "k"
grep -c "k" file

# Counts total OCCURRENCES of "k"
grep -o "k" file | wc -l
```

**Common mistakes:**
- Thinking `-c` counts occurrences (it counts lines!)
- Not considering case sensitivity

---

### 5. Counting Lines WITHOUT Letter "k"

**Task:** Count lines that DON'T contain letter "k".

**Solution:**
```bash
grep -v -c "k" ~wojnicki/lab/Lab3.md
```

**Explanation:**
- `-v` inverts match (shows non-matching lines)
- `-c` counts those lines
- Opposite of previous exercise

**Alternative methods:**
```bash
# Using wc
grep -v "k" ~wojnicki/lab/Lab3.md | wc -l

# Total lines minus lines with k
total=$(wc -l < ~wojnicki/lab/Lab3.md)
with_k=$(grep -c "k" ~wojnicki/lab/Lab3.md)
echo $((total - with_k))
```

**Verification:**
```bash
# Lines with k + lines without k = total lines
grep -c "k" file    # e.g., 42
grep -v -c "k" file  # e.g., 38
wc -l file          # e.g., 80
# 42 + 38 = 80 ✓
```

**Common mistakes:**
- Forgetting `-v` (counts lines WITH k instead)

---

### 6. Counting Total Occurrences of Letter "k"

**Task:** Count how many times the letter "k" appears (not just how many lines).

**Solution:**
```bash
grep -o "k" ~wojnicki/lab/Lab3.md | wc -l
```

**Or case-insensitive:**
```bash
grep -i -o "k" ~wojnicki/lab/Lab3.md | wc -l
```

**Explanation:**
- `-o` shows only matching parts (one per line)
- Each "k" appears on its own line
- `wc -l` counts those lines = total occurrences

**Visual example:**
```
Input line: "kite makes me think"
With -o:
k
k
k

wc -l output: 3
```

**Using tr (alternative approach):**
```bash
cat ~wojnicki/lab/Lab3.md | tr -cd 'k' | wc -c
```

**Explanation of tr method:**
- `tr -cd 'k'` deletes all characters except 'k'
- `wc -c` counts remaining characters (all 'k's)

**For case-insensitive tr method:**
```bash
cat ~wojnicki/lab/Lab3.md | tr -cd 'kK' | wc -c
```

**Comparison:**
```bash
# Task 4: Lines containing k
grep -c "k" file           # e.g., 42 lines

# Task 6: Total k occurrences  
grep -o "k" file | wc -l   # e.g., 156 total
```

**Common mistakes:**
- Using `-c` without `-o` (counts lines, not occurrences)
- Forgetting case sensitivity

---

### 7. Displaying File in One Line

**Task:** Display the entire lab description in a single line.

**Solution:**
```bash
cat ~wojnicki/lab/Lab3.md | tr -d '\n'
```

**Or:**
```bash
tr -d '\n' < ~wojnicki/lab/Lab3.md
```

**Explanation:**
- `tr -d '\n'` deletes all newline characters
- All lines concatenate into one

**Verification:**
```bash
# Check it's really one line
cat ~wojnicki/lab/Lab3.md | tr -d '\n' | wc -l
# Output: 1
```

**To add newline at end:**
```bash
cat ~wojnicki/lab/Lab3.md | tr -d '\n'; echo
```

**Alternative using sed:**
```bash
sed ':a;N;$!ba;s/\n//g' ~wojnicki/lab/Lab3.md
```

**Alternative using awk:**
```bash
awk '{printf "%s",$0}' ~wojnicki/lab/Lab3.md
```

**Common mistakes:**
- Using `tr '\n' ' '` (replaces with spaces, still multiple words)
- Not verifying with `wc -l`

---

### 8. Copying All .JPG Files

**Task:** Copy all files ending in .JPG from zasoby directory to ~/lab3/jpg using a single command.

**Solution:**

**First, create target directory:**
```bash
mkdir -p ~/lab3/jpg
```

**Then copy files:**
```bash
cp ~wojnicki/lab/zasoby/*.JPG ~/lab3/jpg/
```

**Or using find:**
```bash
find ~wojnicki/lab/zasoby -name "*.JPG" -exec cp {} ~/lab3/jpg/ \;
```

**Explanation:**
- `*.JPG` wildcard matches all files ending in .JPG
- `cp` copies all matched files to destination
- Shell expands wildcard before running cp

**Verification:**
```bash
ls ~/lab3/jpg/
# Should list all copied .JPG files
```

**If case doesn't matter:**
```bash
# Using find for case-insensitive
find ~wojnicki/lab/zasoby -iname "*.jpg" -exec cp {} ~/lab3/jpg/ \;
```

**Common mistakes:**
- Not creating destination directory first
- Wrong wildcard pattern
- Copying from wrong source directory

---

### 9. Removing Non-JPEG Files

**Task:** Delete files from ~/lab3/jpg that aren't actually JPEG images (regardless of filename).

**Solution:**
```bash
cd ~/lab3/jpg
for file in *; do
    if ! file "$file" | grep -q "JPEG\|JPG"; then
        rm "$file"
    fi
done
```

**Or more concise:**
```bash
cd ~/lab3/jpg && file * | grep -v "JPEG\|image data" | cut -d: -f1 | xargs rm
```

**Explanation:**
1. `file *` examines all files, shows types
2. `grep -v "JPEG\|image data"` filters out actual JPEGs
3. `cut -d: -f1` extracts filenames
4. `xargs rm` deletes those files

**Step-by-step breakdown:**

**Step 1 - Check file types:**
```bash
file ~/lab3/jpg/*
# Output:
# file1.JPG: JPEG image data
# file2.JPG: ASCII text
# file3.JPG: JPEG image data
```

**Step 2 - Find non-JPEGs:**
```bash
file ~/lab3/jpg/* | grep -v "JPEG"
# Output:
# file2.JPG: ASCII text
```

**Step 3 - Extract filenames and delete:**
```bash
file ~/lab3/jpg/* | grep -v "JPEG" | cut -d: -f1 | xargs rm
```

**Safe version (shows what would be deleted):**
```bash
cd ~/lab3/jpg
for file in *; do
    if ! file "$file" | grep -q "JPEG"; then
        echo "Would delete: $file ($(file -b $file))"
        # rm "$file"  # Uncomment to actually delete
    fi
done
```

**Common mistakes:**
- Trusting file extensions (task specifically says ignore names)
- Not testing before deleting
- Wrong grep pattern

---

### 10. Finding All Directories in /home (No Errors)

**Task:** Find all directories recursively in /home without showing error messages.

**Solution:**
```bash
find /home -type d 2> /dev/null
```

**Explanation:**
- `find /home` searches /home directory
- `-type d` matches only directories
- `2> /dev/null` redirects errors to null device (discards them)

**Why errors occur:**
- Permission denied on some directories
- Normal users can't read all of /home

**Alternative (redirect to file for review):**
```bash
find /home -type d 2> ~/lab3/find_errors.txt
```

**Verification:**
```bash
# Count directories found
find /home -type d 2> /dev/null | wc -l
```

**Common mistakes:**
- Not redirecting stderr (screen filled with errors)
- Using `> /dev/null` instead of `2>` (hides results, not errors!)
- Wrong redirection: `2>&1 > /dev/null` (incorrect order)

---

### 11. Display and Save find Output Simultaneously

**Task:** Display directories from previous exercise on screen AND save to ~/lab3/home.

**Solution:**
```bash
find /home -type d 2> /dev/null | tee ~/lab3/home
```

**Explanation:**
- `find` searches for directories
- `2> /dev/null` hides errors
- `| tee ~/lab3/home` splits output: shows on screen + saves to file

**How tee works:**
```
find → tee → screen (stdout)
           ↓
         ~/lab3/home
```

**To append instead of overwrite:**
```bash
find /home -type d 2> /dev/null | tee -a ~/lab3/home
```

**Verification:**
```bash
# Check file was created and has content
wc -l ~/lab3/home
cat ~/lab3/home | head
```

**Alternative (save both output and errors):**
```bash
find /home -type d 2>&1 | tee ~/lab3/home_with_errors
```

**Common mistakes:**
- Using `>` instead of `| tee` (only saves, doesn't display)
- Not handling errors properly

---

### 12. Finding Directories with "0" in Name

**Task:** Find directories containing digit "0" in their names within /home, hide errors.

**Solution:**
```bash
find /home -type d -name "*0*" 2> /dev/null
```

**Explanation:**
- `find /home` searches /home
- `-type d` only directories
- `-name "*0*"` name contains "0" anywhere
- `2> /dev/null` hides errors

**Pattern matching:**
- `*0*` matches: home0, backup10, 2024, etc.
- `*` matches any characters before/after "0"

**Examples of matches:**
```
/home/user0
/home/user/folder10
/home/student/2024
/home/backup/v0.1
```

**Case-insensitive (if needed):**
```bash
find /home -type d -iname "*0*" 2> /dev/null
```

**Verification:**
```bash
# Count matching directories
find /home -type d -name "*0*" 2> /dev/null | wc -l
```

**Common mistakes:**
- Wrong wildcard: `0*` only matches names starting with 0
- Not specifying `-type d` (includes files too)

---

### 13. Limiting Search Depth

**Task:** Repeat previous but limit to 3 levels deep.

**Solution:**
```bash
find /home -maxdepth 3 -type d -name "*0*" 2> /dev/null
```

**Explanation:**
- `-maxdepth 3` stops recursion after 3 levels
- Faster and more focused results

**Level counting:**
```
/home           ← level 1
/home/user      ← level 2
/home/user/dir  ← level 3
/home/user/dir/sub ← level 4 (not searched)
```

**Different depth limits:**
```bash
# Only /home itself
find /home -maxdepth 1 -type d -name "*0*" 2> /dev/null

# Up to 2 levels
find /home -maxdepth 2 -type d -name "*0*" 2> /dev/null

# Start at level 2 (skip /home direct children)
find /home -mindepth 2 -maxdepth 3 -type d -name "*0*" 2> /dev/null
```

**Comparison:**
```bash
# Without limit (might be thousands)
find /home -type d -name "*0*" 2> /dev/null | wc -l

# With limit (much fewer)
find /home -maxdepth 3 -type d -name "*0*" 2> /dev/null | wc -l
```

**Common mistakes:**
- Confusing maxdepth with number of subdirectories
- Not understanding level counting

---

### 14. Finding and Copying Text Files

**Task:** Find all text files in ~/lab1 and copy them to ~/kopia_zapasowa.

**Solution:**

**First, create backup directory:**
```bash
mkdir -p ~/kopia_zapasowa
```

**Then find and copy:**
```bash
find ~/lab1 -type f -name "*.txt" -exec cp {} ~/kopia_zapasowa/ \;
```

**Explanation:**
- `find ~/lab1` searches lab1 directory
- `-type f` only regular files (not directories)
- `-name "*.txt"` files ending in .txt
- `-exec cp {} ~/kopia_zapasowa/ \;` copies each file
  - `{}` placeholder for found filename
  - `\;` terminates the -exec command

**Alternative using xargs:**
```bash
find ~/lab1 -type f -name "*.txt" | xargs -I {} cp {} ~/kopia_zapasowa/
```

**For all text files (not just .txt extension):**
```bash
find ~/lab1 -type f -exec file {} \; | grep "text" | cut -d: -f1 | xargs -I {} cp {} ~/kopia_zapasowa/
```

**Verification:**
```bash
# List copied files
ls ~/kopia_zapasowa/

# Count files
find ~/lab1 -name "*.txt" | wc -l
ls ~/kopia_zapasowa/*.txt | wc -l
# Should match
```

**Common mistakes:**
- Forgetting `\;` at end of -exec
- Wrong syntax: `cp ~/kopia_zapasowa/ {}` (reversed arguments)
- Not creating destination directory first

---

### 15. Case-Insensitive Search for "lab"

**Task:** Find all files in ~wojnicki/lab with "lab" in name (ignore case).

**Solution:**
```bash
find ~wojnicki/lab -iname "*lab*"
```

**Explanation:**
- `find ~wojnicki/lab` searches the directory
- `-iname` case-insensitive name matching
- `"*lab*"` matches: lab, Lab, LAB, LabFile, mylab, etc.

**Examples of matches:**
```
Lab1.md
laboratory.txt
LABFILE
research_lab_data
```

**Alternative (using grep after find):**
```bash
find ~wojnicki/lab -type f | grep -i "lab"
```

**Only files (not directories):**
```bash
find ~wojnicki/lab -type f -iname "*lab*"
```

**Verification:**
```bash
# Count matches
find ~wojnicki/lab -iname "*lab*" | wc -l
```

**Common mistakes:**
- Using `-name` instead of `-iname` (misses case variants)
- Forgetting wildcards: `-iname "lab"` (exact match only)

---

## Summary Question

### Explain the complex command:

```bash
find ~wojnicki -type f -name "*.txt" -exec cp {} ~/kopia_zapasowa/ \; 2>&1 | tee -a plik.txt
```

**Complete Explanation:**

**Part 1: `find ~wojnicki -type f -name "*.txt"`**
- `find ~wojnicki` - Search in wojnicki's home directory
- `-type f` - Find only regular files (not directories)
- `-name "*.txt"` - Match files ending in .txt

**Part 2: `-exec cp {} ~/kopia_zapasowa/ \;`**
- `-exec` - Execute command on each found file
- `cp` - Copy command
- `{}` - Placeholder replaced with found filename
- `~/kopia_zapasowa/` - Destination directory
- `\;` - Terminates the -exec command

**Part 3: `2>&1`**
- `2>&1` - Redirect stderr (errors) to stdout
- Combines error messages with normal output
- Allows both to be piped together

**Part 4: `| tee -a plik.txt`**
- `|` - Pipe combined output
- `tee` - Duplicate stream
- `-a` - Append mode (don't overwrite)
- `plik.txt` - Log file

**What it does step by step:**

1. **Find** all .txt files in ~wojnicki
2. **Copy** each to ~/kopia_zapasowa/
3. **Combine** stdout and stderr
4. **Display** on screen AND **append** to plik.txt

**Example output in plik.txt:**
```
Copied: /home/wojnicki/file1.txt
Copied: /home/wojnicki/docs/file2.txt
cp: cannot stat '/home/wojnicki/protected/file3.txt': Permission denied
```

**Use cases:**
- Backup files with logging
- Monitor copy operation in real-time
- Preserve record of errors for review

**Alternative without tee (just logging):**
```bash
find ~wojnicki -type f -name "*.txt" -exec cp {} ~/kopia_zapasowa/ \; &>> plik.txt
```

---

## Quick Reference

### Essential grep Patterns
```bash
grep -i pattern file         # Case-insensitive
grep -v pattern file         # Invert (non-matching)
grep -n pattern file         # Line numbers
grep -B 2 pattern file       # 2 lines before
grep -A 2 pattern file       # 2 lines after
grep -C 2 pattern file       # 2 lines context
grep -c pattern file         # Count lines
grep -o pattern file         # Only matches
```

### Essential find Patterns
```bash
find /path -name "*.txt"           # By name
find /path -iname "*.txt"          # Case-insensitive
find /path -type f                 # Files only
find /path -type d                 # Directories only
find /path -maxdepth 3             # Limit depth
find /path -name "*.txt" -exec cmd {} \;  # Execute command
find /path -name "*.txt" 2>/dev/null      # Hide errors
```

### Error Handling
```bash
command 2> errors.txt        # Save errors
command 2> /dev/null         # Discard errors
command &> all.txt           # Save everything
command 2>&1 | tee log.txt   # Display and log
```

---

**End of Model Answers for Lab 3**
