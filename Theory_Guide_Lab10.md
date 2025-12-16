# Theory Guide - Lab 10: Regular Expressions, grep, sed, awk

## Table of Contents
1. [Regular Expressions Basics](#regular-expressions-basics)
2. [grep - Pattern Searching](#grep-pattern-searching)
3. [sed - Stream Editor](#sed-stream-editor)
4. [awk - Pattern Scanning and Processing](#awk-pattern-scanning-and-processing)
5. [Command Reference](#command-reference)

---

## Regular Expressions Basics

### What are Regular Expressions?

**Regular expressions (regex)** are patterns used to match character combinations in strings.

### Basic Metacharacters

| Character | Meaning | Example |
|-----------|---------|---------|
| `.` | Any single character | `a.c` matches "abc", "a1c" |
| `*` | Zero or more of previous | `ab*c` matches "ac", "abc", "abbc" |
| `+` | One or more (ERE) | `ab+c` matches "abc", "abbc" |
| `?` | Zero or one (ERE) | `ab?c` matches "ac", "abc" |
| `^` | Start of line | `^abc` matches "abc" at start |
| `$` | End of line | `abc$` matches "abc" at end |
| `[]` | Character class | `[abc]` matches "a", "b", or "c" |
| `[^]` | Negated class | `[^abc]` matches any except a,b,c |
| `\` | Escape special char | `\.` matches literal "." |

### Character Classes

| Class | Meaning | Example |
|-------|---------|---------|
| `[abc]` | Any of a, b, or c | `[aeiou]` - vowels |
| `[a-z]` | Range a through z | `[A-Z]` - uppercase |
| `[0-9]` | Any digit | `[0-9]+` - number |
| `[^abc]` | Not a, b, or c | `[^0-9]` - non-digit |

### Quantifiers

| Quantifier | Meaning |
|------------|---------|
| `*` | 0 or more |
| `+` | 1 or more (ERE) |
| `?` | 0 or 1 (ERE) |
| `{n}` | Exactly n |
| `{n,}` | n or more |
| `{n,m}` | Between n and m |

### Anchors

| Anchor | Meaning | Example |
|--------|---------|---------|
| `^` | Start of line | `^hello` |
| `$` | End of line | `world$` |
| `\<` | Start of word | `\<word` |
| `\>` | End of word | `word\>` |
| `\b` | Word boundary | `\bword\b` |

### Special Sequences

| Sequence | Meaning (Extended) |
|----------|-------------------|
| `\d` | Digit [0-9] |
| `\D` | Non-digit [^0-9] |
| `\w` | Word character [a-zA-Z0-9_] |
| `\W` | Non-word character |
| `\s` | Whitespace [ \t\n\r] |
| `\S` | Non-whitespace |

### Examples

```bash
# Match any letter
[a-zA-Z]

# Match 4-letter username
^[a-z]{4}:

# Match lines ending with 'n'
n$

# Match 2 or 3 digit numbers
^[0-9]{2,3}$

# Match email address
[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}

# Match IP address
[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}
```

---

## grep - Pattern Searching

### Basic Usage

**Search for pattern in file:**
```bash
grep pattern file.txt
```

**Common options:**

| Option | Meaning |
|--------|---------|
| `-i` | Ignore case |
| `-v` | Invert match (show non-matching) |
| `-n` | Show line numbers |
| `-c` | Count matches |
| `-l` | List filenames only |
| `-r` | Recursive search |
| `-E` | Extended regex (ERE) |
| `-w` | Match whole word |
| `-x` | Match whole line |
| `-A n` | Show n lines after match |
| `-B n` | Show n lines before match |
| `-C n` | Show n lines context |

### Basic Regex (BRE)

```bash
# Lines containing "user"
grep user /etc/passwd

# Lines starting with "root"
grep ^root /etc/passwd

# Lines ending with "bash"
grep bash$ /etc/passwd

# Any single character
grep "r..t" /etc/passwd  # root, r00t, etc.

# Character class
grep "[aeiou]" file.txt  # Lines with vowels
```

### Extended Regex (ERE)

**Use `-E` flag:**

```bash
# One or more digits
grep -E '[0-9]+' file.txt

# Optional character
grep -E 'colou?r' file.txt  # color or colour

# Alternation
grep -E 'cat|dog' file.txt

# Grouping
grep -E '(ab)+' file.txt  # ab, abab, ababab

# Multiple words
grep -E 'error|warning|fail' logfile
```

### Practical Examples

**Find 4-letter usernames:**
```bash
grep -E '^[a-z]{4}:' /etc/passwd
```

**Lines ending with specific letter:**
```bash
grep 'n$' /etc/passwd
```

**2 or 3 digit user IDs:**
```bash
grep -E ':[0-9]{2,3}:' /etc/passwd
```

**IP addresses:**
```bash
grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' logfile
```

**Email addresses:**
```bash
grep -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' file.txt
```

### grep Variants

**fgrep (fixed strings, no regex):**
```bash
fgrep 'string' file.txt
# or
grep -F 'string' file.txt
```

**egrep (extended regex):**
```bash
egrep 'pattern' file.txt
# or
grep -E 'pattern' file.txt
```

---

## sed - Stream Editor

### Basic Syntax

```bash
sed 'command' file.txt
sed -e 'command1' -e 'command2' file.txt
```

### Print Commands

**Print all lines (like cat):**
```bash
sed '' file.txt
# or
sed 'p' file.txt  # Prints each line twice
sed -n 'p' file.txt  # -n suppresses default output
```

**Print specific lines:**
```bash
sed -n '2p' file.txt      # Line 2
sed -n '2,5p' file.txt    # Lines 2-5
sed -n '1p;3p;5p' file.txt  # Lines 1, 3, 5
sed -n '$p' file.txt      # Last line
```

**Print range:**
```bash
sed -n '10,20p' file.txt  # Lines 10-20
sed -n '1,5p' file.txt    # First 5 lines
```

### Substitution

**Basic substitution:**
```bash
sed 's/old/new/' file.txt
```
- Replaces first occurrence on each line

**Global substitution:**
```bash
sed 's/old/new/g' file.txt
```
- Replaces all occurrences

**Case-insensitive:**
```bash
sed 's/old/new/gi' file.txt
```

**Nth occurrence:**
```bash
sed 's/old/new/2' file.txt  # Replace 2nd occurrence
```

### Delimiters

**Use different delimiter:**
```bash
sed 's/\/usr\/bin/\/usr\/local\/bin/' file.txt  # Confusing
sed 's|/usr/bin|/usr/local/bin|' file.txt       # Clear
sed 's#/usr/bin#/usr/local/bin#' file.txt       # Also works
```

### Delete Commands

**Delete lines:**
```bash
sed '2d' file.txt        # Delete line 2
sed '2,5d' file.txt      # Delete lines 2-5
sed '/pattern/d' file.txt  # Delete lines matching pattern
```

### Address Ranges

**Pattern ranges:**
```bash
sed -n '/start/,/end/p' file.txt  # Print from start to end
```

**Line number to pattern:**
```bash
sed -n '5,/pattern/p' file.txt  # From line 5 to pattern
```

### Multiple Commands

**Using -e:**
```bash
sed -e 's/foo/bar/' -e 's/baz/qux/' file.txt
```

**Using semicolon:**
```bash
sed 's/foo/bar/; s/baz/qux/' file.txt
```

**Using script file:**
```bash
cat > script.sed <<EOF
s/foo/bar/
s/baz/qux/
EOF

sed -f script.sed file.txt
```

### Practical Examples

**Display file contents:**
```bash
sed '' /etc/passwd
```

**Replace separator:**
```bash
sed 's/:/ /g' /etc/passwd
```

**Extract first field (login):**
```bash
sed 's/:.*//' /etc/passwd
```

**Display specific lines:**
```bash
sed -n '2,5p' /etc/passwd
```

**Display lines starting with 'w' or 'z':**
```bash
sed -n '/^[wz]/p' /etc/passwd
```

**Add line numbers:**
```bash
sed = file.txt | sed 'N; s/\n/\t/'
```

**Remove blank lines:**
```bash
sed '/^$/d' file.txt
```

**Remove comments:**
```bash
sed '/^#/d' file.txt
```

---

## awk - Pattern Scanning and Processing

### Basic Syntax

```bash
awk 'pattern { action }' file.txt
```

### Built-in Variables

| Variable | Meaning |
|----------|---------|
| `$0` | Entire line |
| `$1, $2, ...` | Fields 1, 2, etc. |
| `NF` | Number of fields |
| `NR` | Current record (line) number |
| `FS` | Field separator (default: whitespace) |
| `OFS` | Output field separator |
| `RS` | Record separator (default: newline) |
| `ORS` | Output record separator |
| `FILENAME` | Current filename |

### Print Commands

**Print entire line:**
```bash
awk '{print}' file.txt
# or
awk '{print $0}' file.txt
```

**Print specific fields:**
```bash
awk '{print $1}' file.txt       # First field
awk '{print $1, $3}' file.txt   # Fields 1 and 3
awk '{print $NF}' file.txt      # Last field
```

**Print with text:**
```bash
awk '{print "User:", $1}' /etc/passwd
```

### Field Separator

**Default (whitespace):**
```bash
awk '{print $1}' file.txt
```

**Custom separator:**
```bash
awk -F: '{print $1}' /etc/passwd
# or
awk 'BEGIN {FS=":"} {print $1}' /etc/passwd
```

### Pattern Matching

**Match pattern:**
```bash
awk '/pattern/ {print}' file.txt
```

**Regex in specific field:**
```bash
awk '$1 ~ /pattern/ {print}' file.txt
```

**Not matching:**
```bash
awk '$1 !~ /pattern/ {print}' file.txt
```

**Numeric comparisons:**
```bash
awk '$3 > 100 {print}' file.txt
awk '$2 == "value" {print}' file.txt
```

### BEGIN and END

**BEGIN block (before processing):**
```bash
awk 'BEGIN {print "Starting..."} {print $1}' file.txt
```

**END block (after processing):**
```bash
awk '{sum += $1} END {print "Total:", sum}' file.txt
```

### Variables and Operators

**Assignment:**
```bash
awk '{count++} END {print count}' file.txt
```

**Arithmetic:**
```bash
awk '{sum += $1; count++} END {print sum/count}' file.txt
```

**String concatenation:**
```bash
awk '{full = $1 " " $2; print full}' file.txt
```

### Conditional Statements

**if-else:**
```bash
awk '{
    if ($3 > 100)
        print $1, "large"
    else
        print $1, "small"
}' file.txt
```

### Loops

**for loop:**
```bash
awk '{
    for (i=1; i<=NF; i++)
        print $i
}' file.txt
```

**while loop:**
```bash
awk '{
    i = 1
    while (i <= NF) {
        print $i
        i++
    }
}' file.txt
```

### Arrays (Associative)

**Count occurrences:**
```bash
awk '{count[$1]++} END {
    for (item in count)
        print item, count[item]
}' file.txt
```

**Sum by category:**
```bash
awk '{sum[$1] += $2} END {
    for (category in sum)
        print category, sum[category]
}' file.txt
```

### String Functions

| Function | Description |
|----------|-------------|
| `length(s)` | Length of string s |
| `substr(s, p, n)` | Substring from position p, length n |
| `index(s, t)` | Position of t in s |
| `split(s, a, sep)` | Split s into array a |
| `match(s, r)` | Match regex r in s |
| `sub(r, s)` | Replace first match |
| `gsub(r, s)` | Replace all matches |
| `tolower(s)` | Convert to lowercase |
| `toupper(s)` | Convert to uppercase |

**Examples:**

```bash
# Extract substring
awk '{print substr($1, 1, 4)}' file.txt

# String length
awk '{print length($0)}' file.txt

# Match and extract
awk 'match($0, /[0-9]+/) {print substr($0, RSTART, RLENGTH)}' file.txt

# Replace
awk '{gsub(/old/, "new"); print}' file.txt
```

### Practical Examples

**Sum column:**
```bash
awk '{sum += $3} END {print sum}' file.txt
```

**Average:**
```bash
awk '{sum += $1; n++} END {print sum/n}' file.txt
```

**Count lines:**
```bash
awk 'END {print NR}' file.txt
```

**CSV processing:**
```bash
awk -F, '{print $1, $3}' file.csv
```

**Swap columns:**
```bash
awk '{print $2, $1}' file.txt
```

**Filter and format:**
```bash
awk '$3 > 1000 {print $1 ": $" $3}' sales.txt
```

**Complex log processing:**
```bash
awk '
    /200|404/ {
        if (match($0, /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/)) {
            ip = substr($0, RSTART, RLENGTH)
            requests[ip]++
            bytes[ip] += $(NF)
            if (!(ip in first_seen))
                first_seen[ip] = NR
        }
    }
    END {
        for (ip in requests)
            print "IP:", ip, "Requests:", requests[ip], 
                  "Bytes:", bytes[ip], "First:", first_seen[ip]
    }
' logfile
```

---

## Command Reference

### grep

```bash
# Basic search
grep pattern file.txt

# Options
grep -i pattern file.txt      # Case insensitive
grep -v pattern file.txt      # Invert match
grep -n pattern file.txt      # Line numbers
grep -c pattern file.txt      # Count matches
grep -E 'pat1|pat2' file.txt  # Extended regex
grep -r pattern directory/    # Recursive

# Regex examples
grep '^start' file.txt        # Lines starting with "start"
grep 'end$' file.txt          # Lines ending with "end"
grep '[0-9]' file.txt         # Lines with digits
grep -E '[0-9]{2,3}' file.txt # 2-3 digit numbers
```

### sed

```bash
# Print
sed '' file.txt              # Display all
sed -n '2,5p' file.txt       # Lines 2-5

# Substitute
sed 's/old/new/' file.txt    # First occurrence
sed 's/old/new/g' file.txt   # All occurrences
sed 's|/path|/new|' file.txt # Different delimiter

# Delete
sed '/pattern/d' file.txt    # Delete matching lines
sed '2d' file.txt            # Delete line 2

# Multiple commands
sed -e 's/a/A/' -e 's/b/B/' file.txt

# In-place editing
sed -i 's/old/new/g' file.txt
```

### awk

```bash
# Print
awk '{print}' file.txt          # All lines
awk '{print $1}' file.txt       # First field
awk -F: '{print $1}' /etc/passwd # Custom separator

# Pattern matching
awk '/pattern/ {print}' file.txt
awk '$3 > 100 {print}' file.txt

# Calculations
awk '{sum += $1} END {print sum}' file.txt
awk '{count++} END {print count}' file.txt

# Arrays
awk '{arr[$1]++} END {for (i in arr) print i, arr[i]}' file.txt

# Formatted output
awk '{printf "%-10s %5d\n", $1, $2}' file.txt
```

---

## Best Practices

### 1. Quote Patterns

```bash
# Good
grep 'pattern' file.txt

# Bad (can break with special chars)
grep pattern file.txt
```

### 2. Use Extended Regex When Needed

```bash
# Extended features
grep -E 'pat1|pat2' file.txt
grep -E '[0-9]+' file.txt

# Or use egrep
egrep 'pat1|pat2' file.txt
```

### 3. Test on Sample Data

```bash
# Test first
head -20 large_file.txt | grep pattern

# Then apply to full file
grep pattern large_file.txt
```

### 4. Combine Tools

```bash
# grep + cut
grep pattern file.txt | cut -d: -f1

# sed + awk
sed 's/,/ /g' file.csv | awk '{print $1, $3}'

# Pipeline
cat file.txt | grep pattern | sed 's/old/new/' | awk '{print $1}'
```

### 5. Use -n with sed for Specific Output

```bash
# Without -n, sed prints all lines
sed '2p' file.txt  # Prints line 2 twice

# With -n, only explicit prints
sed -n '2p' file.txt  # Prints line 2 once
```

---

## Common Patterns

### Extract Fields

```bash
# First field (username) from /etc/passwd
awk -F: '{print $1}' /etc/passwd
sed 's/:.*//' /etc/passwd
cut -d: -f1 /etc/passwd
```

### Count Occurrences

```bash
# Count unique IPs
awk '{print $1}' access.log | sort | uniq -c

# Count with awk
awk '{count[$1]++} END {for (ip in count) print ip, count[ip]}' access.log
```

### Filter and Transform

```bash
# Lines with 200 status, extract IP and bytes
awk '/200/ {print $1, $NF}' access.log

# Replace multiple patterns
sed -e 's/foo/bar/' -e 's/baz/qux/' file.txt
```

### Log Analysis

```bash
# Sum bytes transferred
awk '{sum += $NF} END {print sum}' access.log

# Average response time
awk '{sum += $5; n++} END {print sum/n}' metrics.log

# Group by status code
awk '{count[$9]++} END {for (s in count) print s, count[s]}' access.log
```

---

## Summary

**Regular Expressions:** Pattern matching language for text  
**grep:** Search for patterns in files  
**sed:** Stream editor for transforming text  
**awk:** Pattern scanning and text processing language  

**Key Concepts:**
- Regex applies to single lines (per match)
- `-E` enables extended regex features (+, ?, |, etc.)
- sed transforms streams without modifying files
- awk excels at field-based processing and calculations
- Combine tools in pipelines for complex processing

**Common Pipeline:**
```bash
grep pattern file | sed 's/old/new/' | awk '{print $1, $3}' | sort | uniq
```
