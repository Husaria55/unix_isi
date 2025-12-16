# Theory Guide - Lab 8: Permissions, Links, and File System

## Table of Contents
1. [File Permissions](#file-permissions)
2. [Changing Permissions](#changing-permissions)
3. [Ownership](#ownership)
4. [Special Permissions](#special-permissions)
5. [Links (Hard and Symbolic)](#links)
6. [Named Pipes (FIFO)](#named-pipes)
7. [User Information](#user-information)
8. [Command Reference](#command-reference)

---

## File Permissions

### Understanding Permissions

**View permissions:**
```bash
ls -l file.txt
-rw-r--r-- 1 user group 1234 Dec 16 14:30 file.txt
```

**Permission string breakdown:**
```
-rw-r--r--
│├─┼─┼─┼─┘
││ │ │ └── Others permissions (r--)
││ │ └──── Group permissions (r--)
││ └────── Owner permissions (rw-)
│└──────── Special permissions
└───────── File type (- = regular file)
```

### File Types

| Symbol | Type |
|--------|------|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `p` | Named pipe (FIFO) |
| `s` | Socket |
| `b` | Block device |
| `c` | Character device |

### Permission Types

| Permission | File | Directory |
|------------|------|-----------|
| **r** (read) | View file contents | List directory contents (ls) |
| **w** (write) | Modify file | Create/delete files in directory |
| **x** (execute) | Run as program | Enter directory (cd) |

**Key insight for directories:**
- **r** - Can list files (`ls`)
- **x** - Can enter and access files (`cd`)
- **w** - Can create/delete files

**Examples:**

**Directory with r but no x:**
```bash
chmod 644 mydir  # r-- for all, no x
ls mydir         # Works - can list
cd mydir         # Fails - cannot enter
cat mydir/file   # Fails - cannot access files
```

**Directory with x but no r:**
```bash
chmod 711 mydir  # --x for all
ls mydir         # Fails - cannot list
cd mydir         # Works - can enter
cat mydir/file   # Works if you know filename!
```

**Directory with w but no x:**
```bash
chmod 622 mydir  # -w- for all, no x
cd mydir         # Fails - cannot enter
rm mydir/file    # Fails - need x to access
```

---

## Changing Permissions

### The chmod Command

**Symbolic Mode:**
```bash
chmod [who][operator][permissions] file
```

**Who:**
- `u` - User (owner)
- `g` - Group
- `o` - Others
- `a` - All (ugo)

**Operator:**
- `+` - Add permission
- `-` - Remove permission
- `=` - Set exactly

**Examples:**
```bash
chmod u+x file       # Add execute for user
chmod g-w file       # Remove write for group
chmod o=r file       # Set others to read only
chmod a+r file       # Add read for all
chmod ug+rw file     # Add read+write for user and group
chmod go-rwx file    # Remove all for group and others
```

**Numeric Mode:**

Each permission has a value:
- **r** = 4
- **w** = 2
- **x** = 1

Sum values for each category:

| Permissions | Calculation | Octal |
|-------------|-------------|-------|
| `---` | 0+0+0 | 0 |
| `--x` | 0+0+1 | 1 |
| `-w-` | 0+2+0 | 2 |
| `-wx` | 0+2+1 | 3 |
| `r--` | 4+0+0 | 4 |
| `r-x` | 4+0+1 | 5 |
| `rw-` | 4+2+0 | 6 |
| `rwx` | 4+2+1 | 7 |

**Common numeric permissions:**
```bash
chmod 644 file   # rw-r--r-- (owner: rw, others: r)
chmod 755 file   # rwxr-xr-x (owner: rwx, others: rx)
chmod 700 file   # rwx------ (owner only)
chmod 777 file   # rwxrwxrwx (all permissions - dangerous!)
chmod 600 file   # rw------- (owner read/write only)
```

**Examples:**
```bash
chmod 644 document.txt   # Readable by all, writable by owner
chmod 755 script.sh      # Executable by all, writable by owner
chmod 700 private/       # Owner-only access
chmod 600 ~/.ssh/id_rsa  # Private key permissions
```

---

## Ownership

### The chown Command

**Change owner:**
```bash
sudo chown newowner file
```

**Change owner and group:**
```bash
sudo chown newowner:newgroup file
```

**Change group only:**
```bash
sudo chown :newgroup file
# Or use chgrp:
chgrp newgroup file
```

**Recursive:**
```bash
sudo chown -R user:group directory/
```

**Examples:**
```bash
# Change owner to alice
sudo chown alice file.txt

# Change owner and group
sudo chown alice:developers project/

# Change group only
chgrp developers file.txt

# Recursive change
sudo chown -R alice:alice /home/alice/data
```

### Viewing Ownership

```bash
ls -l file.txt
-rw-r--r-- 1 alice developers 1234 Dec 16 14:30 file.txt
           │ │     │
           │ │     └── Group
           │ └──────── Owner
           └────────── Link count
```

**Check ownership:**
```bash
stat file.txt | grep -E 'Uid|Gid'
```

---

## Special Permissions

### SUID (Set User ID)

**Effect:** File executes with owner's permissions, not executor's.

**Symbol:** `s` in user execute position

**Example:**
```bash
ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root ...
```

**Set SUID:**
```bash
chmod u+s file
# Or numeric:
chmod 4755 file
```

**Use case:** Allow non-root users to change passwords (needs root access to /etc/shadow)

### SGID (Set Group ID)

**On files:** Execute with group's permissions

**On directories:** New files inherit directory's group

**Symbol:** `s` in group execute position

**Set SGID:**
```bash
chmod g+s directory
# Or numeric:
chmod 2755 directory
```

**Example:**
```bash
mkdir shared
chmod 2775 shared
chgrp developers shared

# Any file created in shared/ will have group 'developers'
```

### Sticky Bit

**Effect:** Only owner can delete their files (even if directory is writable)

**Symbol:** `t` in others execute position

**Example:** `/tmp` directory
```bash
ls -ld /tmp
drwxrwxrwt 10 root root ...
```

**Set sticky bit:**
```bash
chmod +t directory
# Or numeric:
chmod 1777 directory
```

**Use case:** Shared directories where users shouldn't delete each other's files

### Numeric Mode with Special Permissions

**Format:** `[special][user][group][others]`

| Value | Permission |
|-------|------------|
| 4000 | SUID |
| 2000 | SGID |
| 1000 | Sticky bit |

**Examples:**
```bash
chmod 4755 file  # SUID + rwxr-xr-x
chmod 2755 dir   # SGID + rwxr-xr-x
chmod 1777 dir   # Sticky + rwxrwxrwx
chmod 6755 file  # SUID + SGID + rwxr-xr-x
```

---

## Links

### Hard Links

**What:** Another name for the same file (same inode)

**Create:**
```bash
ln source target
```

**Characteristics:**
- Same inode number
- Same content
- Deleting one doesn't affect the other
- Cannot cross file systems
- Cannot link directories (usually)

**Example:**
```bash
echo "content" > original
ln original hardlink

ls -li
# 123456 -rw-r--r-- 2 user user 8 Dec 16 14:30 original
# 123456 -rw-r--r-- 2 user user 8 Dec 16 14:30 hardlink
#  ^same inode        ^link count = 2

cat hardlink  # Shows "content"
rm original   # hardlink still works!
```

**Link count:**
```bash
ls -l file
-rw-r--r-- 3 user user ...
           ^ number of hard links to this inode
```

**Why directories have count ≥ 2:**
```bash
mkdir parent
mkdir parent/child

ls -ld parent
drwxr-xr-x 3 user user ...
           ^ count = 3
```

**Three links:**
1. `parent` (the directory name itself)
2. `parent/.` (self-reference)
3. `parent/child/..` (reference from child)

### Symbolic (Soft) Links

**What:** Pointer to another file (different inode)

**Create:**
```bash
ln -s target linkname
```

**Characteristics:**
- Different inode
- Points to target path
- If target deleted, link breaks (dangling)
- Can cross file systems
- Can link directories

**Example:**
```bash
echo "content" > original
ln -s original symlink

ls -li
# 123456 -rw-r--r-- 1 user user 8 Dec 16 14:30 original
# 789012 lrwxrwxrwx 1 user user 8 Dec 16 14:30 symlink -> original
#  ^different inode   ^link type  ^points to original

cat symlink   # Shows "content"
rm original   # symlink breaks!
cat symlink   # Error: No such file or directory
```

**Absolute vs relative paths:**
```bash
# Absolute (works from anywhere)
ln -s /home/user/file /tmp/link

# Relative (works from specific location)
cd /home/user
ln -s ../other/file link
```

**Find symlink target:**
```bash
readlink symlink          # Shows target path
readlink -f symlink       # Shows absolute path
ls -l symlink             # Shows link arrow
stat symlink              # Detailed info
```

**Find broken symlinks:**
```bash
find . -type l ! -exec test -e {} \; -print
```

### Comparison: Hard vs Symbolic

| Feature | Hard Link | Symbolic Link |
|---------|-----------|---------------|
| Inode | Same | Different |
| Target deleted | Still works | Breaks |
| Cross filesystem | No | Yes |
| Link directories | No | Yes |
| Size | Same as original | Small (path length) |
| Command | `ln` | `ln -s` |

---

## Named Pipes (FIFO)

### What are FIFOs?

**Named pipe** - Special file that acts as a queue between processes.

**Characteristics:**
- First In, First Out (FIFO)
- Blocks until both reader and writer connected
- No data stored in pipe itself

### Creating FIFOs

```bash
mkfifo pipe_name
```

**Example:**
```bash
mkfifo mypipe

ls -l mypipe
prw-r--r-- 1 user user 0 Dec 16 14:30 mypipe
^pipe type
```

### Using FIFOs

**Terminal 1 (writer):**
```bash
echo "Hello" > mypipe
# Blocks until someone reads
```

**Terminal 2 (reader):**
```bash
cat < mypipe
# Output: Hello
```

**Practical example - logging:**
```bash
# Create pipe
mkfifo /tmp/logpipe

# Reader (background)
cat < /tmp/logpipe >> logfile.txt &

# Writers can send to pipe
echo "Event 1" > /tmp/logpipe
echo "Event 2" > /tmp/logpipe
```

---

## User Information

### Commands

**users** - Show logged-in usernames
```bash
users
# Output: alice bob charlie
```

**who** - Show who is logged in with details
```bash
who
# alice pts/0  2024-12-16 14:00 (192.168.1.10)
# bob   pts/1  2024-12-16 14:15 (192.168.1.20)
```

**w** - Show who and what they're doing
```bash
w
#  14:30:00 up 10 days,  3:42,  2 users,  load average: 0.15, 0.18, 0.20
# USER  TTY    FROM       LOGIN@   IDLE   JCPU   PCPU WHAT
# alice pts/0  192.168..  14:00    0.00s  0.05s  0.01s vim file.txt
# bob   pts/1  192.168..  14:15   10:00   0.02s  0.02s -bash
```

**id** - Show user/group IDs
```bash
id
# uid=1000(alice) gid=1000(alice) groups=1000(alice),27(sudo),1001(developers)

id alice
# uid=1000(alice) gid=1000(alice) groups=1000(alice),27(sudo)

id -u      # UID only
id -g      # Primary GID only
id -G      # All group IDs
id -un     # Username
id -gn     # Group name
```

### The stat Command

**Show detailed file information:**
```bash
stat file.txt
```

**Output:**
```
  File: file.txt
  Size: 1234      	Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d	Inode: 123456      Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/   alice)   Gid: ( 1000/   alice)
Access: 2024-12-16 14:00:00.000000000 +0000
Modify: 2024-12-16 14:30:00.000000000 +0000
Change: 2024-12-16 14:30:00.000000000 +0000
 Birth: -
```

**Specific format:**
```bash
stat -c '%a' file     # Permissions (octal)
stat -c '%U' file     # Owner name
stat -c '%G' file     # Group name
stat -c '%i' file     # Inode number
stat -c '%h' file     # Link count
```

---

## The umask Command

### What is umask?

**umask** sets default permissions for newly created files/directories.

**How it works:**
- Files: Start with 666, subtract umask
- Directories: Start with 777, subtract umask

**View current umask:**
```bash
umask
# Output: 0022
```

**Set umask:**
```bash
umask 0077  # Restrictive (owner only)
umask 0022  # Default (owner write, others read)
```

**Examples:**

**umask 0022:**
```
Files:      666 - 022 = 644 (rw-r--r--)
Directories: 777 - 022 = 755 (rwxr-xr-x)
```

**umask 0077:**
```
Files:      666 - 077 = 600 (rw-------)
Directories: 777 - 077 = 700 (rwx------)
```

**Permanent umask:**
```bash
echo "umask 0077" >> ~/.bashrc
```

---

## Command Reference

### Permissions

```bash
chmod 644 file              # Set permissions (numeric)
chmod u+x file              # Add execute (symbolic)
chmod -R 755 dir/           # Recursive
ls -l file                  # View permissions
stat -c '%a' file           # Permission octal
```

### Ownership

```bash
chown user file             # Change owner
chown user:group file       # Change owner and group
chgrp group file            # Change group only
chown -R user:group dir/    # Recursive
```

### Special Permissions

```bash
chmod u+s file              # Set SUID
chmod g+s dir               # Set SGID
chmod +t dir                # Set sticky bit
chmod 4755 file             # SUID (numeric)
chmod 2755 dir              # SGID (numeric)
chmod 1777 dir              # Sticky (numeric)
```

### Links

```bash
ln source target            # Hard link
ln -s target linkname       # Symbolic link
readlink link               # Show link target
readlink -f link            # Absolute path
stat file                   # Show inode, links
```

### FIFOs

```bash
mkfifo pipename             # Create named pipe
echo "data" > pipe          # Write to pipe
cat < pipe                  # Read from pipe
```

### Users

```bash
users                       # Logged-in users
who                         # Who with details
w                           # Who and what doing
id                          # User/group IDs
id username                 # Specific user info
```

### umask

```bash
umask                       # View current
umask 0077                  # Set restrictive
umask 0022                  # Set default
```

---

## Common Permission Scenarios

### Private File

```bash
chmod 600 private.txt       # rw------- (owner only)
```

### Executable Script

```bash
chmod 755 script.sh         # rwxr-xr-x (all can run)
```

### Shared Group Directory

```bash
mkdir /shared
chmod 2770 /shared          # rwxrws--- (group access, SGID)
chgrp developers /shared
```

### Web Directory

```bash
chmod 755 /var/www/html/    # rwxr-xr-x (public read)
chmod 644 index.html        # rw-r--r-- (public read file)
```

### SSH Keys

```bash
chmod 700 ~/.ssh            # rwx------ (directory)
chmod 600 ~/.ssh/id_rsa     # rw------- (private key)
chmod 644 ~/.ssh/id_rsa.pub # rw-r--r-- (public key)
```

---

## Summary

Lab 8 covers permissions and file system:

**Permissions:**
- **r** (4), **w** (2), **x** (1)
- Symbolic: `chmod u+x`, `chmod go-w`
- Numeric: `chmod 644`, `chmod 755`

**Directory permissions:**
- **r** - List contents
- **w** - Create/delete files
- **x** - Enter and access

**Ownership:**
- `chown user:group file`
- `chgrp group file`

**Special permissions:**
- **SUID** (4000) - Run as owner
- **SGID** (2000) - Inherit group
- **Sticky** (1000) - Owner-delete only

**Links:**
- **Hard:** Same inode, same file
- **Symbolic:** Pointer, can break

**Tools:**
- `chmod`, `chown`, `chgrp`
- `ln`, `ln -s`
- `mkfifo`
- `stat`, `id`, `umask`

**Key Principle:** Proper permissions ensure security and collaboration.
