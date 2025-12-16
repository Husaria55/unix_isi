# Model Answers - Lab 10: Regular Expressions, sed, awk

## Introduction Questions

**Q1: Jak przy pomocy polecenia `grep` znaleźć dowolną literę?**

**Answer:** 
Use the `.` (dot) metacharacter to match any single character:

```bash
grep '.' file.txt
```

However, this matches ANY character. To match specifically letters:

```bash
# Any single letter (lowercase or uppercase)
grep '[a-zA-Z]' file.txt

# Or using character classes
grep '[[:alpha:]]' file.txt
```

The `.` in regex means "any character except newline", so `grep '.'` matches all non-empty lines.

**Q2: Jak przy pomocy polecenia `sed` zamienić jeden tekst na inny w każdej linii?**

**Answer:** 
Use the substitute command with the `g` (global) flag:

```bash
sed 's/old_text/new_text/g' file.txt
```

**Breakdown:**
- `s` - substitute command
- `/old_text/` - pattern to find
- `/new_text/` - replacement text  
- `/g` - global flag (all occurrences in each line)

Without `g`, only the first occurrence on each line is replaced:
```bash
sed 's/old/new/' file.txt  # Only first occurrence per line
```

**Q3: Czy wyrażenia regularne polecenia `grep` mają zastosowanie do jednej linii czy wielu?**

**Answer:** 
Regex in `grep` applies to **one line at a time**. Each line is matched independently.

**Example:**
```bash
# This matches "start" at beginning of any single line
grep '^start' file.txt

# This does NOT match "start" on one line and "end" on another
grep '^start.*end$' file.txt
# It only matches if "start" and "end" are on the SAME line
```

For multi-line matching, use tools like:
- `pcregrep -M` (PCRE grep with multiline)
- `awk` with record separator manipulation
- `perl` with multiline mode

**Q4: Co daje opcja '-E' w poleceniu `grep`?**

**Answer:** 
The `-E` flag enables **Extended Regular Expressions (ERE)**, which provides additional metacharacters without escaping:

**Without `-E` (BRE - Basic):**
```bash
grep 'one\|two' file.txt        # Need to escape |
grep '\(ab\)+' file.txt         # Need to escape ( ) +
```

**With `-E` (ERE - Extended):**
```bash
grep -E 'one|two' file.txt      # Alternation
grep -E '(ab)+' file.txt        # Grouping and +
grep -E '[0-9]+' file.txt       # One or more digits
grep -E 'colou?r' file.txt      # Optional character
```

**Extended features:**
- `+` - One or more
- `?` - Zero or one
- `|` - Alternation (OR)
- `()` - Grouping
- `{n,m}` - Quantifiers (work without escaping)

Equivalent: `egrep` is the same as `grep -E`.

---

## Task Solutions

### Task 1: Find 4-letter Student Logins

**Requirement:** Find lines in /etc/passwd with 4-letter logins in the first field.

**Solution:**

```bash
# Basic solution
grep -E '^[a-z]{4}:' /etc/passwd

# More specific (lowercase letters only)
grep -E '^[a-z]{4}:' /etc/passwd

# Include uppercase possibility
grep -E '^[a-zA-Z]{4}:' /etc/passwd

# Only lowercase letters, exactly 4
grep -E '^[a-z]{4}:' /etc/passwd
```

**Explanation:**

**Pattern breakdown: `^[a-z]{4}:`**
- `^` - Start of line (ensures we're checking the first field)
- `[a-z]` - Any lowercase letter
- `{4}` - Exactly 4 times
- `:` - Followed by colon (field separator in /etc/passwd)

**Why each part is necessary:**

1. **`^`** - Without this, would match 4 letters anywhere:
   ```bash
   # Without ^:
   grep -E '[a-z]{4}:' /etc/passwd
   # Would match: root:x:0:0:root:/root:/bin/bash
   # Because "root" has 4 letters
   
   # With ^:
   grep -E '^[a-z]{4}:' /etc/passwd
   # Only matches lines starting with 4 letters
   ```

2. **`{4}`** - Ensures exactly 4 letters:
   ```bash
   grep -E '^[a-z]{4}:' /etc/passwd
   # Matches: john:x:1001:...
   # Doesn't match: root:x:0:...     (only 3 letters)
   # Doesn't match: admin:x:1002:... (5 letters)
   ```

3. **`:`** - Ensures we're at field boundary:
   ```bash
   # Without colon:
   grep -E '^[a-z]{4}' /etc/passwd
   # Would match: john123:...  (john + other chars)
   
   # With colon:
   grep -E '^[a-z]{4}:' /etc/passwd
   # Only matches: john:...    (exactly 4, then colon)
   ```

**Testing:**
```bash
# Create test file
cat > test_users.txt <<EOF
abc:x:100:100::/home/abc:/bin/bash
abcd:x:101:101::/home/abcd:/bin/bash
abcde:x:102:102::/home/abcde:/bin/bash
john:x:1000:1000::/home/john:/bin/bash
admin:x:1001:1001::/home/admin:/bin/bash
EOF

# Test the pattern
grep -E '^[a-z]{4}:' test_users.txt
# Output:
# abcd:x:101:101::/home/abcd:/bin/bash
# john:x:1000:1000::/home/john:/bin/bash
```

**Alternative approaches:**

```bash
# Using awk (more precise)
awk -F: 'length($1) == 4 && $1 ~ /^[a-z]+$/' /etc/passwd

# Using grep with word boundary
grep -E '^\<[a-z]{4}\>:' /etc/passwd
```

---

### Task 2: Lines Ending with Letter 'n'

**Requirement:** Find lines in /etc/passwd ending with letter 'n'.

**Solution:**

```bash
grep 'n$' /etc/passwd
```

**Explanation:**

**Pattern: `n$`**
- `n` - Literal letter 'n'
- `$` - End of line anchor

**Why it works:**
```bash
# $ matches end of line
grep 'n$' /etc/passwd
# Matches lines like:
# root:x:0:0:root:/root:/bin/bash     ← ends with 'h', NOT matched
# user:x:1000:1000::/home/user:/bin/zsh  ← ends with 'h', NOT matched
# daemon:x:1:1:daemon:/usr/sbin:/bin/sh  ← ends with 'h', NOT matched
# But if any line ends with 'n', it's matched
```

**Common /etc/passwd entries ending with 'n':**
```bash
# Lines ending with /bin/false/nologin might exist
# Or custom entries ending with 'n'

# Example matches:
# myuser:x:1001:1001::/home/myuser:/bin/bashn
# admin:x:1002:1002:Admin User:/home/admin
```

**Testing:**
```bash
# Create test data
cat > test.txt <<EOF
This line ends with bash
This line ends with n
Another line with python
Final line
EOF

grep 'n$' test.txt
# Output:
# This line ends with n
# Another line with python
```

**Case-insensitive version:**
```bash
grep -i 'n$' /etc/passwd
```

**Count matches:**
```bash
grep -c 'n$' /etc/passwd
```

**Alternative - last character is 'n':**
```bash
awk '/n$/' /etc/passwd
sed -n '/n$/p' /etc/passwd
```

---

### Task 3: Users with 2 or 3-digit IDs

**Requirement:** Using cut, grep, and regex, find users with 2 or 3-digit user IDs.

**Solution:**

```bash
# Extract UID field (3rd field), then filter
cut -d: -f3 /etc/passwd | grep -E '^[0-9]{2,3}$'

# Better: show full lines
grep -E '^[^:]+:[^:]+:[0-9]{2,3}:' /etc/passwd

# Or with cut and grep combined to show usernames
cut -d: -f1,3 /etc/passwd | grep -E ':[0-9]{2,3}$'
```

**Explanation:**

**Method 1: Extract UIDs, filter, but lose context**
```bash
cut -d: -f3 /etc/passwd | grep -E '^[0-9]{2,3}$'
```
- `cut -d: -f3` - Extract 3rd field (UID)
- `grep -E '^[0-9]{2,3}$'` - Match 2-3 digit numbers

**Output:**
```
99
100
120
999
```

**Method 2: Keep full lines**
```bash
grep -E '^[^:]+:[^:]+:[0-9]{2,3}:' /etc/passwd
```
- `^` - Start of line
- `[^:]+` - Username (anything except colon)
- `:` - Separator
- `[^:]+` - Password field
- `:` - Separator
- `[0-9]{2,3}` - UID (2-3 digits)
- `:` - Separator

**Output:**
```
systemd-timesync:x:100:102::/run/systemd:/bin/false
messagebus:x:101:103::/var/run/dbus:/bin/false
```

**Method 3: Show username and UID**
```bash
cut -d: -f1,3 /etc/passwd | grep -E ':[0-9]{2,3}$'
```

**Output:**
```
systemd-timesync:100
messagebus:101
```

**Breakdown of regex `^[0-9]{2,3}$`:**
- `^` - Start of line (entire line is just the number)
- `[0-9]` - Any digit
- `{2,3}` - Between 2 and 3 occurrences
- `$` - End of line

**Why `{2,3}`:**
- `{2,3}` matches: 10, 99, 100, 999
- Does NOT match: 9 (only 1 digit), 1000 (4 digits)

**Complete example with filtering:**

```bash
# Show username, UID, and home directory for 2-3 digit UIDs
awk -F: '$3 ~ /^[0-9]{2,3}$/ {print $1, $3, $6}' /etc/passwd

# Or with grep and cut
grep -E '^[^:]+:[^:]+:[0-9]{2,3}:' /etc/passwd | cut -d: -f1,3,6
```

**Testing:**
```bash
# Create test file
cat > test_passwd.txt <<EOF
root:x:0:0:root:/root:/bin/bash
user:x:50:50::/home/user:/bin/bash
john:x:100:100::/home/john:/bin/bash
admin:x:1000:1000::/home/admin:/bin/bash
guest:x:9:9::/home/guest:/bin/bash
EOF

# Test the pattern
cut -d: -f3 test_passwd.txt | grep -E '^[0-9]{2,3}$'
# Output:
# 50
# 100
```

---

### Task 4: Display /etc/passwd with sed

**Requirement:** Display contents of /etc/passwd using sed.

**Solution:**

```bash
# Method 1: Empty command (simplest)
sed '' /etc/passwd

# Method 2: Print command (but see note)
sed 'p' /etc/passwd

# Method 3: With -n (suppresses default, then explicit print)
sed -n 'p' /etc/passwd

# Method 4: Match all and print
sed -n '1,$p' /etc/passwd
```

**Explanation:**

**`sed '' /etc/passwd`**
- Empty command = no transformation
- sed reads each line, applies no changes, prints
- Equivalent to `cat /etc/passwd`

**`sed 'p' /etc/passwd`**
- `p` command prints current line
- But sed ALSO prints by default
- Result: each line printed TWICE

```bash
# Example:
echo -e "line1\nline2" | sed 'p'
# Output:
# line1
# line1
# line2
# line2
```

**`sed -n 'p' /etc/passwd`**
- `-n` suppresses default printing
- `p` explicitly prints
- Result: each line printed once (correct)

**Why use sed for display?**
Usually you don't! Use `cat`. But understanding `sed ''` helps when building more complex commands:

```bash
# Start simple
sed '' file.txt

# Add transformation
sed 's/old/new/' file.txt

# Add filtering
sed -n '/pattern/p' file.txt
```

**Verification:**
```bash
# These should be identical:
cat /etc/passwd
sed '' /etc/passwd
sed -n 'p' /etc/passwd

# This is different (duplicates):
sed 'p' /etc/passwd
```

---

### Task 5: Replace Colons with Spaces

**Requirement:** Replace separator (colon) in /etc/passwd with space.

**Solution:**

```bash
# Basic substitution
sed 's/:/ /g' /etc/passwd

# Alternative: replace with tab
sed 's/:/\t/g' /etc/passwd

# Replace only first colon
sed 's/:/ /' /etc/passwd
```

**Explanation:**

**Command: `sed 's/:/ /g' /etc/passwd`**
- `s` - Substitute command
- `:` - Search pattern (literal colon)
- ` ` - Replacement (space)
- `g` - Global flag (all occurrences on each line)

**Without `g` flag:**
```bash
sed 's/:/ /' /etc/passwd
# Only replaces FIRST colon on each line
# Input:  root:x:0:0:root:/root:/bin/bash
# Output: root x:0:0:root:/root:/bin/bash
```

**With `g` flag:**
```bash
sed 's/:/ /g' /etc/passwd
# Replaces ALL colons on each line
# Input:  root:x:0:0:root:/root:/bin/bash
# Output: root x 0 0 root /root /bin/bash
```

**Example output:**
```bash
sed 's/:/ /g' /etc/passwd | head -3
# Possible output:
# root x 0 0 root /root /bin/bash
# daemon x 1 1 daemon /usr/sbin /usr/sbin/nologin
# bin x 2 2 bin /bin /usr/sbin/nologin
```

**Variations:**

```bash
# Replace with tab for better alignment
sed 's/:/\t/g' /etc/passwd

# Replace with multiple spaces
sed 's/:/ - /g' /etc/passwd

# Replace colon with newline (one field per line)
sed 's/:/\n/g' /etc/passwd

# Replace only first 3 colons
sed 's/:/ /; s/:/ /; s/:/ /' /etc/passwd
```

**Practical use:**
```bash
# Make more readable
sed 's/:/ /g' /etc/passwd | column -t

# Extract fields easily
sed 's/:/ /g' /etc/passwd | awk '{print $1, $3, $6}'
```

---

### Task 6: Extract Logins with sed

**Requirement:** Display only usernames (logins) from /etc/passwd using sed.

**Solution:**

```bash
# Remove everything after first colon
sed 's/:.*//' /etc/passwd

# Alternative: explicit field extraction
sed -n 's/^\([^:]*\):.*/\1/p' /etc/passwd

# Another approach: delete from first colon to end
sed 's/:.*$//' /etc/passwd
```

**Explanation:**

**Command: `sed 's/:.*//' /etc/passwd`**
- `s` - Substitute
- `:` - Match literal colon
- `.*` - Match any character (`.`), zero or more times (`*`)
- `//` - Replace with nothing (delete)

**How it works:**
```bash
# Input line:
root:x:0:0:root:/root:/bin/bash

# Pattern `:.*` matches:
:x:0:0:root:/root:/bin/bash
# (from first colon to end of line)

# After substitution (removed):
root
```

**Step-by-step:**
```
Original: root:x:0:0:root:/root:/bin/bash
Pattern:  root[:x:0:0:root:/root:/bin/bash]  ← This part matched by :.*
Result:   root                                ← This part remains
```

**Alternative using capture groups:**
```bash
sed -n 's/^\([^:]*\):.*/\1/p' /etc/passwd
```
- `^` - Start of line
- `\([^:]*\)` - Capture group: any non-colon chars
- `:.*` - Colon and rest of line
- `\1` - Replace with captured group (username)

**Why this works:**
- `/etc/passwd` format: `username:password:UID:GID:info:home:shell`
- Username is always before first `:`
- `.*` is greedy - matches everything to end of line
- So `:.*` matches from first `:` to end

**Comparison with other tools:**

```bash
# sed
sed 's/:.*//' /etc/passwd

# awk
awk -F: '{print $1}' /etc/passwd

# cut
cut -d: -f1 /etc/passwd
```

All produce same output:
```
root
daemon
bin
sys
...
```

**Verification:**
```bash
# Should be identical
sed 's/:.*//' /etc/passwd > /tmp/sed_output.txt
cut -d: -f1 /etc/passwd > /tmp/cut_output.txt
diff /tmp/sed_output.txt /tmp/cut_output.txt
# No output = identical
```

---

### Task 7: Display Lines 2-5

**Requirement:** Display lines 2 through 5 from /etc/passwd using sed.

**Solution:**

```bash
# Basic range
sed -n '2,5p' /etc/passwd

# Alternative: delete all except 2-5
sed '2,5!d' /etc/passwd

# Using addresses
sed -n '2p;3p;4p;5p' /etc/passwd
```

**Explanation:**

**Command: `sed -n '2,5p' /etc/passwd`**
- `-n` - Suppress default printing
- `2,5` - Address range (lines 2 through 5)
- `p` - Print command

**Without `-n`:**
```bash
sed '2,5p' /etc/passwd
# Prints ALL lines (default behavior)
# PLUS prints lines 2-5 again
# Result: lines 2-5 appear twice
```

**With `-n`:**
```bash
sed -n '2,5p' /etc/passwd
# Suppresses default printing
# Only explicit 'p' commands print
# Result: lines 2-5 printed once
```

**Alternative - delete inverse:**
```bash
sed '2,5!d' /etc/passwd
```
- `2,5!` - Everything EXCEPT lines 2-5
- `d` - Delete
- Result: only lines 2-5 remain

**Testing:**
```bash
# Create test file
cat > test.txt <<EOF
Line 1
Line 2
Line 3
Line 4
Line 5
Line 6
EOF

# Extract lines 2-5
sed -n '2,5p' test.txt
# Output:
# Line 2
# Line 3
# Line 4
# Line 5
```

**Other range examples:**

```bash
# First 5 lines
sed -n '1,5p' /etc/passwd

# Lines 10 to end
sed -n '10,$p' /etc/passwd

# Only line 3
sed -n '3p' /etc/passwd

# Lines 2, 4, 6
sed -n '2p;4p;6p' /etc/passwd

# Every other line starting from 2
sed -n '2~2p' /etc/passwd  # GNU sed
```

**Comparison with other tools:**

```bash
# sed
sed -n '2,5p' /etc/passwd

# head and tail
head -5 /etc/passwd | tail -4

# awk
awk 'NR>=2 && NR<=5' /etc/passwd

# tail
tail -n +2 /etc/passwd | head -4
```

---

### Task 8: Lines Starting with 'w' or 'z'

**Requirement:** Display lines from /etc/passwd for users with login starting with 'w' or 'z'.

**Solution:**

```bash
# Using character class
sed -n '/^[wz]/p' /etc/passwd

# Using alternation (extended regex)
sed -n -E '/^(w|z)/p' /etc/passwd

# Basic regex with alternation (escaped)
sed -n '/^\(w\|z\)/p' /etc/passwd

# Using grep (simpler)
grep -E '^[wz]' /etc/passwd
```

**Explanation:**

**Command: `sed -n '/^[wz]/p' /etc/passwd`**
- `-n` - Suppress default output
- `/^[wz]/` - Pattern match
  - `^` - Start of line
  - `[wz]` - Character class: 'w' or 'z'
- `p` - Print matching lines

**Pattern breakdown:**
```bash
# ^[wz] matches:
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin  ← starts with 'w'
zabbix:x:110:118:Zabbix:/var/lib/zabbix:/bin/false    ← starts with 'z'

# Does NOT match:
root:x:0:0:root:/root:/bin/bash     ← starts with 'r'
admin:x:1000:1000::/home/admin:/bin/bash ← starts with 'a'
```

**Alternative patterns:**

```bash
# Using alternation (| means OR)
sed -n -E '/^(w|z)/p' /etc/passwd

# Case insensitive
sed -n '/^[wzWZ]/p' /etc/passwd

# More letters
sed -n '/^[a-z]/p' /etc/passwd  # Any lowercase letter
```

**Why use sed instead of grep?**
Usually grep is simpler for filtering:
```bash
grep -E '^[wz]' /etc/passwd
```

But sed allows chaining transformations:
```bash
sed -n '/^[wz]/p' /etc/passwd | sed 's/:/ /g'
# Or in one command:
sed -n '/^[wz]/{s/:/ /g; p}' /etc/passwd
```

**Testing:**
```bash
# Create test data
cat > test_users.txt <<EOF
root:x:0:0:root:/root:/bin/bash
www-data:x:33:33:www-data:/var/www:/bin/false
admin:x:1000:1000:Admin:/home/admin:/bin/bash
zabbix:x:110:118:Zabbix:/var/lib/zabbix:/bin/false
wojtek:x:1001:1001::/home/wojtek:/bin/bash
EOF

# Test the pattern
sed -n '/^[wz]/p' test_users.txt
# Output:
# www-data:x:33:33:www-data:/var/www:/bin/false
# zabbix:x:110:118:Zabbix:/var/lib/zabbix:/bin/false
# wojtek:x:1001:1001::/home/wojtek:/bin/bash
```

**Extract just usernames:**
```bash
sed -n '/^[wz]/p' /etc/passwd | cut -d: -f1
# Or in one command:
sed -n '/^[wz]/{s/:.*//; p}' /etc/passwd
```

---

### Task 9: Transform Date Output Format

**Requirement:** Transform Lab 2, Task 10 command to swap column order (date,count format).

**Context from Lab 2, Task 10:**
Original command produced output like:
```
ile,data
2,20201010
3,20201011
```

Need to transform to:
```
data,ile
20201010,2
20201011,3
```

**Solution:**

```bash
# Original command (assumed from Lab 2):
# [some command producing: ile,data format]

# Transform with awk
[original_command] | awk -F, 'NR==1 {print "data,ile"; next} {print $2 "," $1}'

# Or with sed (swap two fields)
[original_command] | sed -E 's/^([^,]+),([^,]+)$/\2,\1/'

# If original command was something like:
cut -d, -f1,2 file.csv | awk '{count[$2]++} END {print "ile,data"; for (d in count) print count[d] "," d}'

# Transform to:
cut -d, -f1,2 file.csv | awk '{count[$2]++} END {print "data,ile"; for (d in count) print d "," count[d]}'
```

**Explanation:**

**Using awk to swap columns:**

```bash
awk -F, 'NR==1 {print "data,ile"; next} {print $2 "," $1}'
```

**Breakdown:**
- `-F,` - Field separator is comma
- `NR==1` - First row (header)
- `{print "data,ile"; next}` - Print new header, skip rest
- `{print $2 "," $1}` - For other rows, print field 2, comma, field 1

**Using sed to swap:**

```bash
sed -E 's/^([^,]+),([^,]+)$/\2,\1/'
```

**Breakdown:**
- `-E` - Extended regex
- `^` - Start of line
- `([^,]+)` - Capture group 1: non-comma characters (first field)
- `,` - Literal comma
- `([^,]+)` - Capture group 2: non-comma characters (second field)
- `$` - End of line
- `\2,\1` - Replace with group 2, comma, group 1 (swapped)

**Complete example:**

```bash
# Input data (original format)
cat > data.txt <<EOF
ile,data
2,20201010
3,20201011
5,20201012
EOF

# Method 1: awk
awk -F, 'NR==1 {print "data,ile"; next} {print $2 "," $1}' data.txt

# Method 2: sed
sed -E '1s/.*/data,ile/; 2,$s/^([^,]+),([^,]+)$/\2,\1/' data.txt

# Method 3: awk one-liner for all lines
awk -F, '{print $2 "," $1}' data.txt | sed '1s/.*/data,ile/'

# Output (all methods):
# data,ile
# 20201010,2
# 20201011,3
# 20201012,5
```

**If original Lab 2 Task 10 command was:**

```bash
# Hypothetical original:
cat access.log | awk '{print $4}' | cut -d: -f1 | 
    sed 's/\[//' | awk '{count[$1]++} END {
        print "ile,data"
        for (date in count)
            print count[date] "," date
    }'

# Modified to swap columns:
cat access.log | awk '{print $4}' | cut -d: -f1 | 
    sed 's/\[//' | awk '{count[$1]++} END {
        print "data,ile"
        for (date in count)
            print date "," count[date]
    }'
```

**Key change:** In the `print` statement, swap order of `date` and `count[date]`.

---

### Task 10: Advanced awk Log Analysis

**Requirement:** Process web server log, track requests and bytes by IP for status codes 200 or 404.

**Log format:**
```
192.168.1.10 - - [01/Jan/2024:10:00:00 +0000] "GET /index.html HTTP/1.1" 200 1234
```

**Solution:**

```bash
awk '
# Match lines with HTTP status 200 or 404
/200|404/ && /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/ {
    # Extract IP address (first field with IP pattern)
    if (match($0, /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/)) {
        ip = substr($0, RSTART, RLENGTH)
        
        # Track request count
        requests[ip]++
        
        # Track total bytes (last field)
        bytes[ip] += $NF
        
        # Record first occurrence line number
        if (!(ip in first_seen)) {
            first_seen[ip] = NR
        }
    }
}

# After processing all lines, print summary
END {
    for (ip in requests) {
        print "Adres IP: " ip ", Całkowita liczba żądań: " requests[ip] ", Całkowita liczba wysłanych bajtów: " bytes[ip]
        print "Pierwszy raz zauważony w wierszu: " first_seen[ip]
        print ""
    }
}
' logfile.txt
```

**Explanation:**

**Pattern matching:**
```bash
/200|404/ && /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/
```
- `/200|404/` - Line contains status 200 OR 404
- `&&` - AND
- `/[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/` - Line contains IP pattern

**IP extraction:**
```bash
if (match($0, /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/)) {
    ip = substr($0, RSTART, RLENGTH)
}
```
- `match($0, pattern)` - Find pattern in current line
- Returns 1 if found, sets `RSTART` and `RLENGTH`
- `RSTART` - Starting position of match
- `RLENGTH` - Length of match
- `substr($0, RSTART, RLENGTH)` - Extract matched text

**Tracking data with associative arrays:**
```bash
requests[ip]++        # Count requests per IP
bytes[ip] += $NF      # Sum bytes per IP
first_seen[ip] = NR   # Record line number
```

**Arrays:**
- `requests[ip]` - How many requests from this IP
- `bytes[ip]` - Total bytes sent to this IP
- `first_seen[ip]` - Line number of first occurrence

**First occurrence tracking:**
```bash
if (!(ip in first_seen)) {
    first_seen[ip] = NR
}
```
- `ip in first_seen` - Checks if key exists in array
- `!(ip in first_seen)` - True if IP NOT yet recorded
- `NR` - Current record (line) number

**Complete working example:**

```bash
# Create test logfile
cat > logfile.txt <<'EOF'
Log start [01/Jan/2024:09:00:00 +0000]
192.168.1.10 - - [01/Jan/2024:10:00:00 +0000] "GET /index.html HTTP/1.1" 200 1234
192.168.1.10 - - [01/Jan/2024:11:00:00 +0000] ERROR
10.0.0.5 - - [01/Jan/2024:10:01:00 +0000] "POST /login HTTP/1.1" 404 567
192.168.1.10 - - [01/Jan/2024:10:02:00 +0000] "GET /style.css HTTP/1.1" 200 876
172.16.0.1 - - [01/Jan/2024:10:03:00 +0000] "GET /image.jpg HTTP/1.1" 304 0
192.168.1.10 - - [01/Jan/2024:10:04:00 +0000] "GET /index.html HTTP/1.1" 200 1500
Log end [01/Jan/2024:10:09:00 +0000]
EOF

# Run the awk script
awk '
/200|404/ && /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/ {
    if (match($0, /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/)) {
        ip = substr($0, RSTART, RLENGTH)
        requests[ip]++
        bytes[ip] += $NF
        if (!(ip in first_seen)) {
            first_seen[ip] = NR
        }
    }
}
END {
    for (ip in requests) {
        print "Adres IP: " ip ", Całkowita liczba żądań: " requests[ip] ", Całkowita liczba wysłanych bajtów: " bytes[ip]
        print "Pierwszy raz zauważony w wierszu: " first_seen[ip]
        print ""
    }
}
' logfile.txt
```

**Expected output:**
```
Adres IP: 192.168.1.10, Całkowita liczba żądań: 3, Całkowita liczba wysłanych bajtów: 3610
Pierwszy raz zauważony w wierszu: 2

Adres IP: 10.0.0.5, Całkowita liczba żądań: 1, Całkowita liczba wysłanych bajtów: 567
Pierwszy raz zauważony w wierszu: 4
```

**Breakdown of calculation for 192.168.1.10:**

```
Line 2: 200 1234 → count=1, bytes=1234
Line 3: ERROR (skipped, no status 200/404)
Line 5: 200 876  → count=2, bytes=1234+876=2110
Line 7: 200 1500 → count=3, bytes=2110+1500=3610

Result: 3 requests, 3610 bytes, first seen at line 2
```

**Alternative - simpler IP extraction:**
```bash
awk '
/200|404/ {
    ip = $1
    requests[ip]++
    bytes[ip] += $NF
    if (!(ip in first_seen)) {
        first_seen[ip] = NR
    }
}
END {
    for (ip in requests) {
        print "Adres IP: " ip ", Całkowita liczba żądań: " requests[ip] ", Całkowita liczba wysłanych bajtów: " bytes[ip]
        print "Pierwszy raz zauważony w wierszu: " first_seen[ip]
        print ""
    }
}
' logfile.txt
```

This assumes IP is always first field. Works if log format is consistent.

**Script file version:**

```bash
# Save as log_analyzer.awk
cat > log_analyzer.awk <<'EOF'
/200|404/ && /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/ {
    if (match($0, /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/)) {
        ip = substr($0, RSTART, RLENGTH)
        requests[ip]++
        bytes[ip] += $NF
        if (!(ip in first_seen)) {
            first_seen[ip] = NR
        }
    }
}

END {
    for (ip in requests) {
        printf "Adres IP: %s, Całkowita liczba żądań: %d, Całkowita liczba wysłanych bajtów: %d\n", ip, requests[ip], bytes[ip]
        printf "Pierwszy raz zauważony w wierszu: %d\n\n", first_seen[ip]
    }
}
EOF

# Use it
awk -f log_analyzer.awk logfile.txt
```

---

### Task 11: Extract Speech Recognition Text

**Requirement:** Extract recognized text from speech recognition output file.

**Context:** Lines starting with `sentence1` contain the recognized text.

**Solution:**

```bash
# Basic extraction
grep '^sentence1' sample_awk_sed_grep_data.txt

# Extract just the text (remove "sentence1" prefix)
grep '^sentence1' sample_awk_sed_grep_data.txt | sed 's/^sentence1 //'

# Using awk
awk '/^sentence1/ {sub(/^sentence1 /, ""); print}' sample_awk_sed_grep_data.txt

# One line output
grep '^sentence1' sample_awk_sed_grep_data.txt | sed 's/^sentence1 //' | tr '\n' ' '
```

**Explanation:**

**Method 1: grep + sed**
```bash
grep '^sentence1' sample_awk_sed_grep_data.txt | sed 's/^sentence1 //'
```
- `grep '^sentence1'` - Filter lines starting with "sentence1"
- `sed 's/^sentence1 //'` - Remove "sentence1 " prefix

**Method 2: awk**
```bash
awk '/^sentence1/ {sub(/^sentence1 /, ""); print}' sample_awk_sed_grep_data.txt
```
- `/^sentence1/` - Match lines starting with "sentence1"
- `sub(/^sentence1 /, "")` - Remove prefix (first occurrence)
- `print` - Print modified line

**Method 3: Join lines**
```bash
grep '^sentence1' sample_awk_sed_grep_data.txt | sed 's/^sentence1 //' | tr '\n' ' '
```
- Extracts sentences and joins them into one line

**Example with sample data:**

```bash
# Create sample file
cat > sample_awk_sed_grep_data.txt <<EOF
header info
metadata
sentence1 The quick brown fox
debug info
sentence1 jumps over the lazy dog
more metadata
sentence1 This is a test
end of file
EOF

# Extract sentences
grep '^sentence1' sample_awk_sed_grep_data.txt | sed 's/^sentence1 //'
# Output:
# The quick brown fox
# jumps over the lazy dog
# This is a test

# Join into one line
grep '^sentence1' sample_awk_sed_grep_data.txt | sed 's/^sentence1 //' | tr '\n' ' '
# Output:
# The quick brown fox jumps over the lazy dog This is a test
```

**More sophisticated - clean up spacing:**

```bash
grep '^sentence1' sample_awk_sed_grep_data.txt | 
    sed 's/^sentence1 //' | 
    tr '\n' ' ' | 
    sed 's/  / /g; s/^ //; s/ $//'
```

**Alternative - sed only:**

```bash
sed -n 's/^sentence1 //p' sample_awk_sed_grep_data.txt
```
- `-n` - Suppress default output
- `s/^sentence1 //` - Remove prefix
- `p` - Print if substitution succeeded

**Script version:**

```bash
#!/bin/bash
# extract_speech.sh

if [ $# -ne 1 ]; then
    echo "Usage: $0 <speech_recognition_file>"
    exit 1
fi

input_file=$1

# Extract and format
grep '^sentence1' "$input_file" | \
    sed 's/^sentence1 //' | \
    tr '\n' ' ' | \
    sed 's/  */ /g; s/^ //; s/ $//'

echo  # Final newline
```

**Usage:**
```bash
chmod +x extract_speech.sh
./extract_speech.sh sample_awk_sed_grep_data.txt
```

**Advanced - with line numbers:**

```bash
grep -n '^sentence1' sample_awk_sed_grep_data.txt | sed 's/^\([0-9]*\):sentence1 /[\1] /'
```

Output format:
```
[3] The quick brown fox
[5] jumps over the lazy dog
[7] This is a test
```

---

## Summary

Lab 10 covered essential text processing tools:

**Regular Expressions:** Pattern matching language
- `.` any character, `*` zero or more, `+` one or more (ERE)
- `^` start of line, `$` end of line
- `[abc]` character class, `{n,m}` quantifiers

**grep:** Search and filter
- Basic and extended regex (`-E`)
- Pattern matching applies to single lines
- Essential for finding patterns in files

**sed:** Stream editor
- Substitute: `s/old/new/g`
- Print ranges: `sed -n '2,5p'`
- Delete patterns: `sed '/pattern/d'`
- Transform text without modifying files

**awk:** Programming language for text
- Field-based processing: `$1`, `$2`, `$NF`
- Built-in variables: `NR`, `NF`, `FS`
- Associative arrays for counting and aggregation
- BEGIN/END blocks for initialization and summary
- Powerful for log analysis and data transformation

**Key principle:** Combine tools in pipelines for complex text processing tasks.
