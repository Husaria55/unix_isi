# Model Answers & Walkthrough - Lab 2: Basic Data Processing

## Overview

This document provides complete solutions to all Lab 2 exercises, focusing on text processing, redirection, command substitution, and wildcards.

---

## Introductory Questions

### Q1: How do we check what a command is?

**Answer:**

Use the `which` command to find the path to executable commands:

```bash
which command_name
```

**Example:**
```bash
which date
# Output: /usr/bin/date
```

**Q1.1: What if it's not a file?**

Some commands are **built into the shell** (not external programs). Use `type` command:

```bash
type command_name
```

**Examples:**
```bash
type cd
# Output: cd is a shell builtin

type ls
# Output: ls is /usr/bin/ls

type which
# Output: which is /usr/bin/which
```

**Other useful commands:**
```bash
# Get information about any command
type -a command_name    # Show all definitions

# Example with alias
alias ll='ls -la'
type ll
# Output: ll is aliased to `ls -la'
```

---

### Q2: What redirections are available in bash?

**Answer:**

| Redirection | Description |
|-------------|-------------|
| `>` | Redirect stdout to file (overwrite) |
| `>>` | Redirect stdout to file (append) |
| `<` | Redirect stdin from file |
| `2>` | Redirect stderr to file (overwrite) |
| `2>>` | Redirect stderr to file (append) |
| `&>` | Redirect both stdout and stderr (overwrite) |
| `&>>` | Redirect both stdout and stderr (append) |
| `2>&1` | Redirect stderr to stdout |
| `1>&2` or `>&2` | Redirect stdout to stderr |
| `<<` | Here document (multi-line input) |
| `<<<` | Here string (single-line input) |

**Examples:**
```bash
# Standard redirections
command > output.txt           # stdout to file
command >> output.txt          # append stdout
command < input.txt            # stdin from file

# Error redirection
command 2> errors.txt          # stderr to file
command 2>> errors.txt         # append stderr

# Combined
command > out.txt 2> err.txt   # separate files
command &> all.txt             # both to same file
command > out.txt 2>&1         # stderr to stdout, both to file
```

---

### Q3: Space after single-letter options?

**Answer:**

**It depends on the command**, but general guidelines:

**Option takes an argument:**
- **Both work usually:** `-d':'` or `-d ':'`
- **Space is preferred for readability:** `-d ':'`

**Examples:**
```bash
# Both correct:
cut -d':' -f1 /etc/passwd
cut -d ':' -f 1 /etc/passwd

# Space preferred (more readable):
cut -d ' ' -f 3 file.txt
grep -A 5 pattern file
```

**Option without argument:**
```bash
# No space needed/wanted
ls -la          # Correct
ls -l -a        # Also correct, but can be combined

# Some options can stack
tar -xzf        # Same as tar -x -z -f
```

**Best practice:** Use space for clarity unless you're combining flags.

---

## Exercise Solutions

### 1. Finding Command Path and Displaying Info

**Task:** Find the path to the `date` command and display all file information using `ls` and `which`.

**Solution:**
```bash
ls -l $(which date)
```

**Sample output:**
```
-rwxr-xr-x 1 root root 137584 Sep 5 2024 /usr/bin/date
```

**Explanation:**
1. `which date` finds the path: `/usr/bin/date`
2. `$(which date)` substitutes that path into the command
3. `ls -l /usr/bin/date` shows detailed file information
4. The shell executes: `ls -l /usr/bin/date`

**Alternative solutions:**
```bash
# Store in variable first
date_path=$(which date)
ls -l $date_path

# Or combine differently
ls -l `which date`  # Legacy backtick syntax
```

**What the output tells us:**
- `-rwxr-xr-x` - File permissions (executable by all)
- `root root` - Owned by root user and group
- `137584` - File size in bytes
- `/usr/bin/date` - Full path to the executable

**Common mistakes:**
- Forgetting to use command substitution: `ls -l which date` (wrong!)
- Using quotes around substitution: `ls -l "$(which date)"` (unnecessary but not wrong)

---

### 2. Extracting Specific Lines from a File

**Task:** Display lines 2 and 3 from `/etc/passwd` using `head` and `tail`.

**Solution:**
```bash
head -n 3 /etc/passwd | tail -n 2
```

**Explanation:**
1. `head -n 3 /etc/passwd` - Get first 3 lines
2. `|` - Pipe those 3 lines to next command
3. `tail -n 2` - Get last 2 lines (of the 3) = lines 2 and 3

**Visual representation:**
```
Lines in file:    head -n 3:    tail -n 2:
1 root:x:...      1 root:...    2 daemon:...
2 daemon:x:...    2 daemon:...  3 bin:...
3 bin:x:...   ->  3 bin:...  -> 
4 sys:x:...
...
```

**Alternative approach (less efficient):**
```bash
# Get line 2 only
head -n 2 /etc/passwd | tail -n 1

# Get line 3 only  
head -n 3 /etc/passwd | tail -n 1

# Run both
head -n 2 /etc/passwd | tail -n 1
head -n 3 /etc/passwd | tail -n 1
```

**To get just a single line (e.g., line 15):**
```bash
head -n 15 /etc/passwd | tail -n 1
```

**Common mistakes:**
- Reversing head/tail: `tail -n 3 | head -n 2` (gets wrong lines)
- Wrong counts: `head -n 2 | tail -n 3` (can't get 3 lines from 2!)

---

### 3. Generating Number Sequences to File

**Task:** Write numbers 1-10 three times to `~/lab2/numbers.txt`, each number on a new line.

**Solution:**

**First, create the directory:**
```bash
mkdir -p ~/lab2
```

**Then write the numbers three times:**
```bash
seq 1 10 > ~/lab2/numbers.txt
seq 1 10 >> ~/lab2/numbers.txt
seq 1 10 >> ~/lab2/numbers.txt
```

**Explanation:**
1. `seq 1 10` generates numbers 1 through 10 (inclusive)
2. First command with `>` creates/overwrites the file
3. Next two commands with `>>` append to the file
4. Result: 30 lines total (1-10 repeated three times)

**Alternative (one-liner):**
```bash
mkdir -p ~/lab2 && (seq 1 10; seq 1 10; seq 1 10) > ~/lab2/numbers.txt
```

**Another alternative (using loop):**
```bash
mkdir -p ~/lab2
for i in {1..3}; do seq 1 10 >> ~/lab2/numbers.txt; done
```

**Verification:**
```bash
wc -l ~/lab2/numbers.txt
# Output: 30 ~/lab2/numbers.txt

cat ~/lab2/numbers.txt
# Output:
# 1
# 2
# ...
# 10
# 1
# 2
# ...
# 10
# 1
# ...
# 10
```

**Common mistakes:**
- Using `>` all three times (overwrites each time, leaving only one set)
- Forgetting to create the directory first
- Wrong range: `seq 1 9` (excludes 10)

---

### 4. Displaying Unique Lines with uniq

**Task:** Display only unique numbers from `~/lab2/numbers.txt` (remove duplicates).

**Solution:**
```bash
sort ~/lab2/numbers.txt | uniq
```

**Or using sort alone:**
```bash
sort -u ~/lab2/numbers.txt
```

**Output:**
```
1
2
3
4
5
6
7
8
9
10
```

**Explanation:**
- **Why `sort` first?** `uniq` only removes **consecutive** duplicates
- Without sorting, duplicates that aren't adjacent won't be removed
- `sort -u` combines sorting and uniqueness in one command

**Demonstration of why sorting matters:**
```bash
# File content:
# 1
# 2
# 1

# Wrong (doesn't remove non-consecutive duplicates):
uniq file.txt
# Output: 1, 2, 1

# Right (sort first):
sort file.txt | uniq
# Output: 1, 2
```

**To save the result:**
```bash
sort ~/lab2/numbers.txt | uniq > ~/lab2/unique_numbers.txt
```

**Common mistakes:**
- Using `uniq` without `sort` (doesn't remove all duplicates)
- Thinking `uniq` works like SQL DISTINCT (it doesn't - requires sorting)

---

### 5. Finding the Largest Number

**Task:** Find the largest number in `~/lab2/numbers.txt`.

**Solution:**

**Method 1 - Sort descending, take first:**
```bash
sort -nr ~/lab2/numbers.txt | head -n 1
```

**Method 2 - Sort ascending, take last:**
```bash
sort -n ~/lab2/numbers.txt | tail -n 1
```

**Output:**
```
10
```

**Explanation:**

**Method 1:**
- `sort -nr` sorts numerically (`-n`) in reverse order (`-r`)
- Largest number comes first
- `head -n 1` gets the first line

**Method 2:**
- `sort -n` sorts numerically in ascending order
- Largest number comes last
- `tail -n 1` gets the last line

**Why `-n` is crucial:**
```bash
# Without -n (alphabetical sort):
sort -r numbers.txt | head -n 1
# Might output: 9 (alphabetically, "9" > "10")

# With -n (numeric sort):
sort -nr numbers.txt | head -n 1
# Outputs: 10 (numerically correct)
```

**Alternative (minimum and maximum):**
```bash
# Smallest number
sort -n ~/lab2/numbers.txt | head -n 1

# Largest number
sort -n ~/lab2/numbers.txt | tail -n 1
```

**Common mistakes:**
- Forgetting `-n` flag (gets wrong result with multi-digit numbers)
- Using `head` with ascending sort (gets minimum, not maximum)

---

### 6. Creating Date-Based Journal Entries

**Task:** Create journal entries in `~/lab2/dziennik/` with filename YYYYMMDD and content "HH:MM Entry text".

**Solution:**

**First, create the directory:**
```bash
mkdir -p ~/lab2/dziennik
```

**Then create an entry:**
```bash
echo "$(date +%H:%M) To jest mój wpis" >> ~/lab2/dziennik/$(date +%Y%m%d)
```

**Explanation:**
1. `$(date +%Y%m%d)` - Generates filename: e.g., `20251216`
2. `$(date +%H:%M)` - Generates timestamp: e.g., `10:34`
3. `>>` - Appends to file (creates if doesn't exist)
4. Multiple runs on same day add multiple lines

**Example output:**

**If run on December 16, 2025 at 10:34:**
```bash
# File created: ~/lab2/dziennik/20251216
# Content:
10:34 To jest mój wpis
```

**Run again at 14:22:**
```bash
echo "$(date +%H:%M) Drugi wpis" >> ~/lab2/dziennik/$(date +%Y%m%d)

# File: ~/lab2/dziennik/20251216
# Content:
10:34 To jest mój wpis
14:22 Drugi wpis
```

**Generic command for easy reuse:**
```bash
# Function or alias for daily journal
journal() {
    echo "$(date +%H:%M) $*" >> ~/lab2/dziennik/$(date +%Y%m%d)
}

# Use it:
journal This is my entry
journal Another entry here
```

**Verification:**
```bash
# List journal files
ls ~/lab2/dziennik/

# View today's entries
cat ~/lab2/dziennik/$(date +%Y%m%d)
```

**Date format breakdown:**
- `%Y` - 4-digit year: 2025
- `%m` - 2-digit month: 12
- `%d` - 2-digit day: 16
- `%H` - Hour (00-23): 10
- `%M` - Minute (00-59): 34

**Common mistakes:**
- Using `>` instead of `>>` (overwrites previous entries)
- Wrong date format: `%Y%m%d` not `%Y-%m-%d` (no separators)
- Hardcoding the date (doesn't work on other days)

---

### 7. Copying Today's Journal to Tomorrow

**Task:** Copy today's journal entries to a file representing tomorrow, working dynamically.

**Solution:**
```bash
cp ~/lab2/dziennik/$(date +%Y%m%d) ~/lab2/dziennik/$(date -d tomorrow +%Y%m%d)
```

**Or on systems without `-d` (like macOS):**
```bash
cp ~/lab2/dziennik/$(date +%Y%m%d) ~/lab2/dziennik/$(date -v+1d +%Y%m%d)
```

**Explanation:**
1. `$(date +%Y%m%d)` - Today's filename
2. `$(date -d tomorrow +%Y%m%d)` - Tomorrow's filename
3. `cp` copies today's entries to tomorrow's file

**Example:**
```bash
# If today is 2025-12-16:
# Copies: 20251216 -> 20251217
```

**Alternative (more portable):**
```bash
# Calculate tomorrow's date
today=$(date +%Y%m%d)
tomorrow=$(date -d "+1 day" +%Y%m%d)
cp ~/lab2/dziennik/$today ~/lab2/dziennik/$tomorrow
```

**Verification:**
```bash
# Check both files exist and have same content
ls -l ~/lab2/dziennik/$(date +%Y%m%d)*
diff ~/lab2/dziennik/$(date +%Y%m%d) ~/lab2/dziennik/$(date -d tomorrow +%Y%m%d)
```

**For other relative dates:**
```bash
# Yesterday
date -d yesterday +%Y%m%d

# Next week
date -d "+7 days" +%Y%m%d

# Specific date
date -d "2025-12-25" +%Y%m%d
```

**Common mistakes:**
- Hardcoding dates (not dynamic)
- Wrong date arithmetic
- Not handling platform differences (Linux vs macOS date command)

---

### 8. Displaying Current Month's Journal Entries

**Task:** Display all journal entries from the current month.

**Solution:**
```bash
cat ~/lab2/dziennik/$(date +%Y%m)*
```

**Explanation:**
1. `$(date +%Y%m)` - Current year and month: `202512`
2. `*` - Wildcard matches any day: `20251201`, `20251202`, etc.
3. `cat` displays all matching files

**Example:**
```bash
# If current month is December 2025:
# Matches: 20251201, 20251202, ..., 20251216, ..., 20251231
# Shows all entries from these files
```

**To see filenames with content:**
```bash
cat ~/lab2/dziennik/$(date +%Y%m)* | less
```

**To show each file separately with headers:**
```bash
# Use cat with multiple files (shows filenames)
cat ~/lab2/dziennik/$(date +%Y%m)*

# Or more explicit with headers
for file in ~/lab2/dziennik/$(date +%Y%m)*; do
    echo "=== $file ==="
    cat "$file"
    echo
done
```

**To count entries in current month:**
```bash
cat ~/lab2/dziennik/$(date +%Y%m)* | wc -l
```

**Alternative using find:**
```bash
find ~/lab2/dziennik/ -name "$(date +%Y%m)*" -exec cat {} \;
```

**Common mistakes:**
- Forgetting the `*` wildcard (only shows one specific day)
- Wrong format: `$(date +%Y-%m)*` doesn't match format used in task 6

---

### 9. Creating Files with cat

**Task:** Use `cat` to interactively create a file.

**Solution:**

**Create the directory first:**
```bash
mkdir -p ~/lab2
```

**Then use cat:**
```bash
cat > ~/lab2/cat_test.txt
```

**Then type your content:**
```
This is line 1
This is line 2
This is line 3
```

**Press CTRL+D to finish**

**Verification:**
```bash
cat ~/lab2/cat_test.txt
```

**Explanation:**
1. `cat > file` redirects your keyboard input to a file
2. Each line you type is written to the file
3. `CTRL+D` sends EOF (End of File) signal
4. File is saved with all the content you typed

**What CTRL+D does:**
- Sends EOF (End of File) character
- Tells `cat` there's no more input
- Closes the file
- Returns to shell prompt

**Other `cat` creation patterns:**

**Append mode:**
```bash
cat >> ~/lab2/cat_test.txt
More lines here...
CTRL+D
```

**From multiple inputs:**
```bash
cat > combined.txt
First part from keyboard
CTRL+D
```

**Practical uses:**
- Quick file creation without opening editor
- Adding content to files in scripts
- Creating test files

**Common mistakes:**
- Using CTRL+C instead of CTRL+D (cancels without saving)
- Forgetting the `>` (just displays what you type)
- Trying to edit existing content (can't - only append or overwrite)

---

### 10. Creating a Report of Journal Entry Counts

**Task:** Create CSV report showing entry count per journal file (format: `count,YYYYMMDD`).

**Solution:**

**First, ensure you have at least 2 journal files:**
```bash
# If needed, create sample journals
echo "10:00 Test entry" > ~/lab2/dziennik/20241010
echo "11:00 Another" >> ~/lab2/dziennik/20241010
echo "09:00 Morning" > ~/lab2/dziennik/20241011
echo "12:00 Noon" >> ~/lab2/dziennik/20241011
echo "18:00 Evening" >> ~/lab2/dziennik/20241011
```

**Generate the report:**
```bash
wc -l ~/lab2/dziennik/* | head -n -1 | tr -s ' ' | cut -d ' ' -f 2,3 | tr ' ' ','
```

**Explanation (step by step):**

**1. `wc -l ~/lab2/dziennik/*`**
```
2 /home/user/lab2/dziennik/20241010
3 /home/user/lab2/dziennik/20241011
5 total
```

**2. `head -n -1`** (remove last line - the total)
```
2 /home/user/lab2/dziennik/20241010
3 /home/user/lab2/dziennik/20241011
```

**3. `tr -s ' '`** (squeeze multiple spaces to single space)
```
2 /home/user/lab2/dziennik/20241010
3 /home/user/lab2/dziennik/20241011
```

**4. `cut -d ' ' -f 2,3`** (extract fields 2 and 3: count and filename)
```
2 20241010
3 20241011
```

Note: Actually we want count and basename only. Let me refine:

**Better solution:**
```bash
wc -l ~/lab2/dziennik/* | head -n -1 | tr -s ' ' | cut -d ' ' -f 2,3 | rev | cut -d '/' -f 1 | rev | tr ' ' ','
```

**Actually, cleaner approach:**
```bash
cd ~/lab2/dziennik && wc -l * | head -n -1 | tr -s ' ' | tr ' ' ','
```

**Output:**
```
2,20241010
3,20241011
```

**Alternative using awk (more elegant):**
```bash
wc -l ~/lab2/dziennik/* | head -n -1 | awk '{print $1","$2}' | sed 's|.*/||'
```

**Or simplest with basename in the pipeline:**
```bash
for file in ~/lab2/dziennik/*; do
    echo "$(wc -l < $file),$(basename $file)"
done
```

**Expected output format:**
```
2,20241010
3,20241011
```

**Common mistakes:**
- Including the "total" line from `wc -l` output
- Wrong field extraction with `cut`
- Not converting spaces to commas
- Including full path instead of just filename

---

### 11. Adding CSV Header to Report

**Task:** Add header "ile,data" to the report and save to `~/lab2/raport1`.

**Solution:**
```bash
(echo "ile,data"; wc -l ~/lab2/dziennik/* | head -n -1 | tr -s ' ' | awk '{n=split($2,a,"/"); print $1","a[n]}') > ~/lab2/raport1
```

**Simpler alternative:**
```bash
echo "ile,data" > ~/lab2/raport1
cd ~/lab2/dziennik && wc -l * | head -n -1 | tr -s ' ' | tr ' ' ',' >> ~/lab2/raport1
```

**Or step by step:**
```bash
# Create header
echo "ile,data" > ~/lab2/raport1

# Append data
for file in ~/lab2/dziennik/*; do
    count=$(wc -l < "$file")
    name=$(basename "$file")
    echo "$count,$name" >> ~/lab2/raport1
done
```

**Output in ~/lab2/raport1:**
```
ile,data
2,20241010
3,20241011
```

**Explanation:**
1. `echo "ile,data"` - Creates header
2. `>` - Writes header to file (overwrites if exists)
3. Data generation (from task 10)
4. `>>` - Appends data rows

**Using grouping with parentheses:**
```bash
(
    echo "ile,data"
    cd ~/lab2/dziennik
    wc -l * | head -n -1 | tr -s ' ' | tr ' ' ','
) > ~/lab2/raport1
```

**Verification:**
```bash
cat ~/lab2/raport1
# Should show:
# ile,data
# 2,20241010
# 3,20241011
```

**Common mistakes:**
- Appending header (should use `>` for header, `>>` for data)
- Wrong order (data before header)
- Not formatting as proper CSV

---

### 12. Displaying Report Without Header

**Task:** Show content of `~/lab2/raport1` without the first line (header).

**Solution:**
```bash
tail -n +2 ~/lab2/raport1
```

**Or:**
```bash
sed '1d' ~/lab2/raport1
```

**Or:**
```bash
head -n -1 ~/lab2/raport1 | tail -n +2
```

**Output:**
```
2,20241010
3,20241011
```

**Explanation:**

**Method 1: `tail -n +2`**
- `+2` means "starting from line 2"
- Shows line 2 through end
- Most efficient method

**Method 2: `sed '1d'`**
- `1d` means "delete line 1"
- Shows all lines except first

**Alternative approaches:**
```bash
# Skip first line with awk
awk 'NR>1' ~/lab2/raport1

# Skip first line with grep (hacky)
grep -v "^ile,data$" ~/lab2/raport1  # Only if you know exact header
```

**Use case:**
Useful for piping data to other commands that expect just data rows:
```bash
tail -n +2 ~/lab2/raport1 | cut -d ',' -f 1 | sort -n
```

**Common mistakes:**
- Using `tail -n 2` (shows last 2 lines, not skipping first)
- Using `head` without understanding what you want

---

### 13. Extracting First Column from CSV

**Task:** Show only the counts (first column) from `~/lab2/raport1`.

**Solution:**
```bash
cut -d ',' -f 1 ~/lab2/raport1
```

**Output:**
```
ile
2
3
```

**To exclude header:**
```bash
tail -n +2 ~/lab2/raport1 | cut -d ',' -f 1
```

**Output:**
```
2
3
```

**Explanation:**
- `cut -d ','` - Use comma as delimiter
- `-f 1` - Extract first field
- `tail -n +2` - Skip header line

**Alternative methods:**
```bash
# Using awk
awk -F ',' '{print $1}' ~/lab2/raport1

# Without header
awk -F ',' 'NR>1 {print $1}' ~/lab2/raport1
```

**To get sum of all entries:**
```bash
tail -n +2 ~/lab2/raport1 | cut -d ',' -f 1 | awk '{sum+=$1} END {print sum}'
```

**Common mistakes:**
- Not specifying delimiter (cut defaults to TAB)
- Including header in output when you don't want it

---

### 14. Finding Date with Most Entries

**Task:** Find the date with the highest entry count from `~/lab2/raport1`.

**Solution:**
```bash
tail -n +2 ~/lab2/raport1 | sort -t ',' -k 1 -nr | head -n 1 | cut -d ',' -f 2
```

**Or get both count and date:**
```bash
tail -n +2 ~/lab2/raport1 | sort -t ',' -k 1 -nr | head -n 1
```

**Output:**
```
3,20241011
```

**Or just the date:**
```
20241011
```

**Explanation (step by step):**

**1. `tail -n +2 ~/lab2/raport1`** - Skip header
```
2,20241010
3,20241011
```

**2. `sort -t ',' -k 1 -nr`** - Sort by first field, numeric, descending
- `-t ','` - Comma delimiter
- `-k 1` - Sort by field 1
- `-n` - Numeric sort
- `-r` - Reverse (descending)
```
3,20241011
2,20241010
```

**3. `head -n 1`** - Get first line (highest count)
```
3,20241011
```

**4. `cut -d ',' -f 2`** - Extract date only
```
20241011
```

**Alternative using awk:**
```bash
tail -n +2 ~/lab2/raport1 | awk -F ',' '$1 > max {max=$1; date=$2} END {print date}'
```

**To get all dates with maximum count (if there are ties):**
```bash
tail -n +2 ~/lab2/raport1 | sort -t ',' -k 1 -nr | awk -F ',' 'NR==1{max=$1} $1==max{print $2}'
```

**Common mistakes:**
- Not using numeric sort (`-n`) - alphabetical gives wrong results
- Wrong field specified in `cut`
- Not removing header before processing

---

## Summary Questions

### Q1: What numbers does `seq 1 10` display?

**Answer:**

`seq 1 10` displays numbers **from 1 to 10, inclusive** (both 1 and 10 are included).

**Output:**
```
1
2
3
4
5
6
7
8
9
10
```

**General rule:** `seq START END` includes both START and END values.

**Examples:**
```bash
seq 1 5        # 1, 2, 3, 4, 5
seq 5 8        # 5, 6, 7, 8
seq 0 3        # 0, 1, 2, 3
seq 1 1        # 1 (single number)
```

**With increment:**
```bash
seq 1 2 10     # 1, 3, 5, 7, 9 (increment by 2)
seq 0 5 20     # 0, 5, 10, 15, 20
```

---

### Q2: How do we remove duplicates from a file?

**Answer:**

**Method 1: `sort` + `uniq` (most common)**
```bash
sort file.txt | uniq
```

**Method 2: `sort -u` (combines both)**
```bash
sort -u file.txt
```

**Explanation:**

**Why sorting is necessary:**
- `uniq` only removes **consecutive** duplicate lines
- Without sorting, non-adjacent duplicates remain

**Example demonstrating the need for sorting:**

**File content:**
```
apple
banana
apple
cherry
banana
```

**Wrong (uniq without sort):**
```bash
uniq file.txt
# Output: apple, banana, apple, cherry, banana
# (non-consecutive duplicates not removed!)
```

**Right (sort first):**
```bash
sort file.txt | uniq
# Output: apple, banana, cherry
```

**Additional options:**

**Count occurrences:**
```bash
sort file.txt | uniq -c
# Output:
#   2 apple
#   2 banana
#   1 cherry
```

**Show only duplicates:**
```bash
sort file.txt | uniq -d
# Output: apple, banana
```

**Show only unique (non-repeated):**
```bash
sort file.txt | uniq -u
# Output: cherry
```

**Case-insensitive:**
```bash
sort file.txt | uniq -i
```

**Save to file:**
```bash
sort file.txt | uniq > unique.txt
```

**Summary:** Always use `sort` before `uniq`, or use `sort -u` directly.

---

## Quick Reference

### Essential Patterns for Lab 2

**Dynamic filenames with dates:**
```bash
command > file_$(date +%Y%m%d).txt
```

**Remove duplicates:**
```bash
sort file.txt | uniq
```

**Extract and count:**
```bash
cut -d ',' -f 2 data.csv | sort | uniq -c
```

**Generate sequences:**
```bash
seq 1 100 > numbers.txt
```

**Skip header line:**
```bash
tail -n +2 file.csv
```

**Sort numerically:**
```bash
sort -n numbers.txt
```

**CSV processing:**
```bash
cut -d ',' -f 1 data.csv | sort -n
```

---

**End of Model Answers for Lab 2**
