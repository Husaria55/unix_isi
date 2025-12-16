# Model Answers - Lab 8: Permissions, Links, and File System

## Introduction Questions

**Key concepts before starting tasks:**

1. **Proper permissions principle:** Owner can read/write their files, group can read if needed, others have minimal or no access
2. **Check permissions:** `ls -l` or `stat`
3. **Interpret:** `-rwxr-xr-x` = owner(rwx), group(r-x), others(r-x)
4. **Set permissions:** `chmod u+rwx,g+rx,o+rx file` or `chmod 755 file`
5. **Directory permissions:** **x** allows entry, **w** allows file creation/deletion
6. **Check owner:** `ls -l` shows owner and group
7. **Change owner:** `chown newowner file` (requires sudo)
8. **Check/change group:** `ls -l` to check, `chgrp newgroup file` to change

---

## Task Solutions

### Task 1: Directory Link Count

**Command:**
```bash
mkdir -p a/{b,c,d}
ls -ld a | cut -f 2 -d' '
```

**Output:** `5`

**Why does directory 'a' have 5 links?**

**Answer:** Directory link count includes:
1. **a** - The directory name itself
2. **a/.** - Self-reference inside the directory
3. **a/b/..** - Parent reference from subdirectory b
4. **a/c/..** - Parent reference from subdirectory c
5. **a/d/..** - Parent reference from subdirectory d

**Verification:**
```bash
ls -lid a a/. a/b/.. a/c/.. a/d/..
```

**All show same inode number!**

**General rule:** Directory link count = 2 + number of subdirectories
- a has 3 subdirectories → 2 + 3 = 5

---

### Task 2: Symbolic Link Loops

**Commands:**
```bash
mkdir -p a/{b,c,d}
ln -sf `pwd`/a a/b/link
cd a/b/link/b/link/b
```

**Which directory are you in?**

**Check with pwd:**
```bash
pwd
# Output: /home/student/a/b/link/b/link/b
```

**Check with pwd -P (physical path):**
```bash
pwd -P
# Output: /home/student/a/b
```

**Explanation:**
- `pwd` - Shows path with symlinks (logical)
- `pwd -P` - Resolves symlinks to actual location (physical)

The symlink `a/b/link` points to `a`, creating a loop:
```
a/b/link → a
a/b/link/b/link → a/b/link → a
a/b/link/b/link/b → a/b
```

Physically, you're in `/home/student/a/b`, but the path includes symlink references.

---

### Task 3: Hard Link to .bashrc

**Command:**
```bash
ln ~/.bashrc ~/konfiguracja_bash
ls -l ~/.bashrc
```

**Output:**
```
-rw-r--r-- 2 student student 4567 Dec 16 14:00 /home/student/.bashrc
           ^ link count = 2
```

**What does the value 2 mean?**

**Answer:** The **link count** shows how many hard links point to this file's inode.
- Originally 1 (just `.bashrc`)
- After creating hard link: 2 (`.bashrc` and `konfiguracja_bash`)

**Verification:**
```bash
ls -li ~/.bashrc ~/konfiguracja_bash
# 123456 -rw-r--r-- 2 student student 4567 Dec 16 14:00 .bashrc
# 123456 -rw-r--r-- 2 student student 4567 Dec 16 14:00 konfiguracja_bash
#  ^same inode number
```

Both files are the same file (same inode). Deleting one doesn't affect the other.

---

### Task 4: Hard Link to Directory

**Attempt:**
```bash
mkdir testdir
ln testdir testdir_link
```

**Output:**
```
ln: testdir: hard link not allowed for directory
```

**Can you create a hard link to a directory?**

**Answer:** **No** (in most Unix systems).

**Status:** Error (non-zero exit code)

**Why not allowed?**
- Would create circular references
- Could break file system traversal
- Handled specially by kernel (`.` and `..` are exceptions)

**Alternative:** Use symbolic link
```bash
ln -s testdir testdir_link  # This works
```

---

### Task 5: Symbolic Link Exploration

**Commands:**
```bash
cd ~
ln -s /tmp smietnik
```

**In which directory are you if you enter 'smietnik'?**
```bash
cd smietnik
pwd
# Output: /home/student/smietnik

pwd -P
# Output: /tmp
```

**Answer:** Logically in `~/smietnik`, physically in `/tmp`.

---

**Three ways to find where symlink points:**

**Method 1: ls -l**
```bash
ls -l smietnik
# lrwxrwxrwx 1 student student 4 Dec 16 14:30 smietnik -> /tmp
```

**Method 2: readlink**
```bash
readlink smietnik
# Output: /tmp
```

**Method 3: stat**
```bash
stat smietnik | grep File:
# File: smietnik -> /tmp
```

**Additional method: file command**
```bash
file smietnik
# smietnik: symbolic link to /tmp
```

---

### Task 6: Sharing Files (Working in Pairs)

**Part A: Private home directory**

**Configure home directory for owner-only access:**
```bash
chmod 700 ~
```

**Verify:**
```bash
ls -ld ~
# drwx------ ... student student ... /home/student
```

**Part B: Public readable directory**

**Create and configure public directory:**
```bash
mkdir ~/pub-r
chmod 755 ~/pub-r
```

**Explanation:**
- Home directory is `700` (no access for others)
- But need execute permission up to the shared directory
- Solution: `755` allows others to traverse

**Better approach - grant execute on home:**
```bash
chmod 711 ~         # Allow others to traverse (--x)
chmod 755 ~/pub-r   # Allow others to read directory
```

**Test with partner:**

**Partner tries:**
```bash
ls ~student         # Fails (no read on home)
cd ~student         # Works if 711
ls ~student/pub-r   # Works - can list
cat ~student/pub-r/file.txt  # Works - can read
```

---

### Task 7: Group-Based Sharing

**Find common group:**
```bash
groups
# Output: student developers staff

# Partner's groups
groups partner
# Output: partner developers staff
```

**Common group:** `developers`

---

**Set up shared directory:**
```bash
mkdir ~/s
chgrp developers ~/s
chmod 750 ~/s   # rwxr-x--- (Symbolic method)
# OR
chmod u=rwx,g=rx,o= ~/s
```

**Numeric method:**
```bash
chmod 750 ~/s
# 7 (rwx) for owner
# 5 (r-x) for group
# 0 (---) for others
```

**Create file:**
```bash
touch ~/s/1
ls -l ~/s/1
# -rw-r--r-- 1 student student 0 Dec 16 14:30 1
```

**Partner tests:**
```bash
ls ~/student/s       # Works - can list
cd ~/student/s       # Works - can enter
cat ~/student/s/1    # Works - can read
echo "test" > ~/student/s/1  # Fails - no write permission
```

---

**Make file readable by group:**
```bash
chmod 640 ~/s/1
# OR
chmod g+r ~/s/1
```

Now partner can read the file content.

---

**Set SGID bit:**
```bash
chmod g+s ~/s
# OR
chmod 2750 ~/s

ls -ld ~/s
# drwxr-s--- ... student developers ... s
#       ^SGID (s instead of x)
```

**Create new file:**
```bash
touch ~/s/2
ls -l ~/s/2
# -rw-r--r-- 1 student developers 0 Dec 16 14:35 2
#                       ^inherited group from directory!
```

**What changed?**
- File `1`: Group is `student` (creator's primary group)
- File `2`: Group is `developers` (inherited from directory with SGID)

**Benefit:** All files created in shared directory automatically belong to the collaboration group.

---

### Task 8: Renaming Files - Owner vs Directory Permissions

**Question:** Do you need to be file owner to rename it?

**Answer:** **No!** You need **write permission on the DIRECTORY**, not the file.

**Test:**

**Create test structure:**
```bash
mkdir testdir
touch testdir/myfile
chmod 444 testdir/myfile  # File: read-only
chmod 777 testdir         # Directory: full access
```

**Try renaming (as non-owner or with read-only file):**
```bash
mv testdir/myfile testdir/renamed
# Works! Even though file is read-only
```

**Why?** Renaming changes directory entry, not file content.

---

**Directory `w` permission allows:**
- Creating files
- Deleting files
- Renaming files
- Moving files in/out

**File `w` permission allows:**
- Modifying file content

**Experiment:**
```bash
# File writable, directory read-only
mkdir testdir2
touch testdir2/file
chmod 666 testdir2/file   # File writable
chmod 555 testdir2        # Directory read-only

# Try modifying file content
echo "new content" > testdir2/file  # Works! File is writable

# Try renaming
mv testdir2/file testdir2/newname   # Fails! Directory not writable

# Try deleting
rm testdir2/file                     # Fails! Directory not writable
```

**Key insight:** File operations on **content** need file permissions. Operations on **name/existence** need directory permissions.

---

### Task 9: Attendance List

**Objective:** Create sorted, unique list of logged-in users with timestamp filename.

**Solution:**
```bash
mkdir -p ~/lab1/obecnosc

# Get epoch timestamp
timestamp=$(date +%s)

# Create attendance list
users | tr ' ' '\n' | sort -u > ~/lab1/obecnosc/$timestamp
```

**Breakdown:**

**Step 1: Get users**
```bash
users
# Output: alice bob alice charlie bob
```

**Step 2: One user per line**
```bash
users | tr ' ' '\n'
# alice
# bob
# alice
# charlie
# bob
```

**Step 3: Sort and remove duplicates**
```bash
users | tr ' ' '\n' | sort -u
# alice
# bob
# charlie
```

**Step 4: Save with epoch timestamp**
```bash
timestamp=$(date +%s)  # e.g., 1702745400
users | tr ' ' '\n' | sort -u > ~/lab1/obecnosc/$timestamp
```

**Alternative using who:**
```bash
who | cut -d' ' -f1 | sort -u > ~/lab1/obecnosc/$(date +%s)
```

**Alternative using awk:**
```bash
who | awk '{print $1}' | sort -u > ~/lab1/obecnosc/$(date +%s)
```

---

### Task 10: Email System with Named Pipe

**Objective:** Create simple messaging system using named pipe.

**Setup:**

**Create named pipe:**
```bash
mkfifo ~/in
chmod 622 ~/in  # -w--w--w- (write for all)
# OR
chmod a+w ~/in  # Add write for everyone
```

**Verify:**
```bash
ls -l ~/in
# prw--w--w- 1 student student 0 Dec 16 14:40 in
# ^pipe type
```

---

**Reader script:**
```bash
vim ~/mail_reader.sh
```

**Script content:**
```bash
#!/bin/bash

INBOX=~/inbox
PIPE=~/in

# Create inbox if doesn't exist
touch "$INBOX"
chmod 600 "$INBOX"  # Private inbox

echo "Mail reader started (PID: $$)"
echo "Messages will be saved to: $INBOX"

# Infinite loop reading from pipe
while true; do
    if read line; then
        timestamp=$(date '+%Y-%m-%d %H:%M:%S')
        echo "[$timestamp] $line" >> "$INBOX"
        echo "Message received at $timestamp"
    fi
done < "$PIPE"
```

**Make executable and run:**
```bash
chmod +x ~/mail_reader.sh
~/mail_reader.sh &
[1] 12345
```

---

**Sending messages:**

**From same user:**
```bash
echo "Hello to myself!" > ~/in
```

**From other user:**
```bash
echo "Message from Bob" > ~student/in
```

**From script:**
```bash
cat << EOF > ~student/in
This is a longer message
spanning multiple lines
from automated script
EOF
```

---

**Reading inbox:**
```bash
cat ~/inbox
```

**Output:**
```
[2024-12-16 14:45:00] Hello to myself!
[2024-12-16 14:46:15] Message from Bob
[2024-12-16 14:47:30] This is a longer message
[2024-12-16 14:47:30] spanning multiple lines
[2024-12-16 14:47:30] from automated script
```

---

**Secure inbox:**
```bash
chmod 600 ~/inbox  # Only you can read
```

**Test:**
```bash
# Partner tries to read
cat ~student/inbox
# Permission denied
```

---

**Stop mail reader:**
```bash
# Find process
jobs -l

# Kill it
kill %1
# OR
kill 12345
```

---

## Common Mistakes

### 1. Wrong chmod Syntax

```bash
# Wrong
chmod 755 = file

# Correct
chmod 755 file
chmod u=rwx,g=rx,o=rx file
```

### 2. Forgetting Directory x Permission

```bash
# Wrong - can't access files
chmod 644 mydir

# Correct
chmod 755 mydir  # Need x to enter
```

### 3. Symbolic Link with Wrong Path

```bash
# Wrong - relative path breaks when accessing from different location
cd /home/user
ln -s file.txt /tmp/link
cd /tmp
cat link  # Fails - looks for /tmp/file.txt

# Correct - absolute path
ln -s /home/user/file.txt /tmp/link
```

### 4. Hard Link Across Filesystems

```bash
# Fails if /tmp on different filesystem
ln ~/file.txt /tmp/link

# Use symbolic link instead
ln -s ~/file.txt /tmp/link
```

### 5. FIFO Blocking

```bash
# This blocks forever (no reader)
echo "test" > mypipe
# Terminal hangs

# Solution: reader in background first
cat < mypipe &
echo "test" > mypipe
```

---

## Key Takeaways

1. **Permissions:** `rwx` for user, group, others
2. **chmod:** Symbolic (`u+x`) or numeric (`755`)
3. **Directory x:** Required to enter and access files
4. **Directory w:** Allows file creation/deletion/renaming
5. **Hard links:** Same inode, same file
6. **Symbolic links:** Pointer, can break
7. **Link count:** Number of hard links to inode
8. **SGID:** Directory inherits group to new files
9. **Named pipes:** Communication between processes
10. **File operations:** Need file permissions. Name operations need directory permissions.

Proper permissions are fundamental to Unix security and collaboration!
