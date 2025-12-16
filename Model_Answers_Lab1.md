# Model Answers & Walkthrough - Lab 1: Introduction to Unix Systems

## Overview

This document provides complete, step-by-step solutions to all Lab 1 exercises. Each answer includes:
- The exact command(s) to use
- Explanation of why specific options were chosen
- Common mistakes to avoid
- Additional context where helpful

---

## Exercise Solutions

### 1. Logging into the Unix Shell

**Task**: Log into the Unix shell via SSH.

**Solution:**

**On Linux/MacOS:**
```bash
ssh your_username@lab.kis.agh.edu.pl
```

**On Windows (using PuTTY):**
1. Download and install PuTTY from https://putty.org/
2. Open PuTTY
3. Enter hostname: `lab.kis.agh.edu.pl`
4. Port: `22`
5. Connection type: SSH
6. Click "Open"
7. Enter username when prompted
8. Enter password when prompted

**On Windows (using built-in SSH client, Windows 10+):**
```powershell
ssh your_username@lab.kis.agh.edu.pl
```

**Explanation:**
- `ssh` establishes a secure, encrypted connection to a remote server
- Replace `your_username` with your actual KIS account username
- You'll be prompted for your password (characters won't be visible while typing - this is normal)

**Common mistakes:**
- Forgetting to register an account at panel.kis.agh.edu.pl:3000 first
- Not being connected to AGH network (use VPN if remote)
- Mistyping the hostname

---

### 2. Using `apropos` to Find the Password Command

**Task**: Find the command to change password using `apropos` with keyword "authentication", paginate with `less`, and search for "update".

**Solution:**
```bash
apropos authentication | less
```

Then inside `less`, press `/` and type:
```
update
```

Press `n` to find next match if needed.

**Expected result**: You should find `passwd` command in the results.

**Explanation:**
- `apropos authentication` searches all man page descriptions for the word "authentication"
- `| less` pipes the output to the `less` pager for comfortable viewing
- `/update` searches within `less` for the word "update"
- The `passwd` command appears with description mentioning "update" authentication tokens

**Alternative solution:**
```bash
apropos authentication | grep update
```
This directly filters for lines containing "update".

**Common mistakes:**
- Forgetting to pipe to `less` (output scrolls off screen)
- Not using `/` to search within `less`
- Typing commands in `less` (you're in view mode, not command mode)

---

### 3. Browsing Lecture Materials

**Task**: Browse materials in `/home/wojnicki/wyk/`

**Solution:**
```bash
cd /home/wojnicki/wyk/
ls -la
```

**Or without changing directory:**
```bash
ls -la /home/wojnicki/wyk/
```

**Explanation:**
- `cd` changes your current working directory
- `ls -la` lists all files including hidden ones (`-a`) in long format (`-l`)
- Using the full path with `ls` lets you view without changing directories

**To view structure:**
```bash
ls -lR /home/wojnicki/wyk/
```

The `-R` flag shows recursive listing of all subdirectories.

**Common mistakes:**
- Not having permission to access (should be readable)
- Trying to modify files (you should only read)

---

### 4. Reading Session Files from Lecture

**Task**: Read the session file `/home/wojnicki/wyk/script/w1` and identify issues with character display.

**Solution:**
```bash
less /home/wojnicki/wyk/script/w1
```

**Or:**
```bash
more /home/wojnicki/wyk/script/w1
```

**Or to display entire file:**
```bash
cat /home/wojnicki/wyk/script/w1
```

**Explanation:**
- `less` is the best choice for viewing - allows scrolling both directions
- `more` is simpler but only scrolls forward
- `cat` displays everything at once (good for short files)

**Character display issues:**
Session files (created with the `script` command) may contain:
- Control characters
- ANSI color codes
- Terminal escape sequences

These might not display correctly in all viewers. They show as strange characters like `^M`, `[01;32m`, etc.

**To see raw control characters:**
```bash
cat -A /home/wojnicki/wyk/script/w1
```

The `-A` flag shows all characters including non-printable ones.

**Common mistakes:**
- Using a text editor instead of a viewer (could accidentally modify the file)
- Not recognizing that control characters are normal in session files

---

### 5. Finding Information About /etc/passwd Using man

**Task**: Use `whatis` and `man` to find what the third column in `/etc/passwd` means.

**Solution:**

**Step 1 - Use whatis:**
```bash
whatis passwd
```

**Output:**
```
passwd (1)           - change user password
passwd (5)           - the password file
```

**Step 2 - Open the correct manual section:**
```bash
man 5 passwd
```

**Explanation:**
- `whatis passwd` shows there are two sections: (1) for the command, (5) for the file format
- Section 5 documents file formats and conventions
- `man 5 passwd` opens the documentation for the `/etc/passwd` file format
- Inside the man page, look for the "DESCRIPTION" section

**Answer**: The third column is **UID (User ID)** - a unique numerical identifier for the user.

**Alternative approach:**
```bash
man -a passwd
```
Then press `q` to skip the first man page and view the second (section 5).

**Common mistakes:**
- Only reading `man passwd` (section 1) which describes the command, not the file
- Not understanding that man pages have multiple sections
- Looking at the wrong man page section

---

### 6. Understanding `man -a` Option

**Task**: What does the `-a` option do in the `man` command?

**Solution:**
```bash
man man
```

Then search for `-a` option by typing `/-a` in the man viewer.

**Answer**: The `-a` option displays **all** available manual pages for a command, one after another. After viewing one man page, press `q` to view the next section.

**Example:**
```bash
man -a passwd
```
This will show:
1. First: `passwd(1)` - the password command
2. After pressing `q`: `passwd(5)` - the password file format

**Explanation:**
- By default, `man` shows only the first matching manual page
- `-a` (all) sequentially displays every matching page across all sections
- Useful when a name appears in multiple sections (commands, system calls, file formats, etc.)

**Common mistakes:**
- Expecting all man pages to appear simultaneously (they appear sequentially)
- Not pressing `q` to move to the next section

---

### 7. Finding Your Home Directory Path and Current Directory

**Task**: 
a) What is the path to your home directory?
b) What command shows the full path of your current directory?

**Solution:**

**Part a - Home directory path:**
```bash
echo $HOME
```

**Or:**
```bash
cd
pwd
```

**Or:**
```bash
pwd
```
(if you're already in your home directory)

**Typical output:**
```
/home/your_username
```

**Part b - Current directory path:**
```bash
pwd
```

**Explanation:**
- `pwd` stands for "Print Working Directory"
- It always shows the **absolute path** (from root `/`)
- `$HOME` is an environment variable storing your home directory path
- `echo $HOME` displays the value of that variable

**Common mistakes:**
- Confusing relative paths with absolute paths
- Not understanding that `~` is shorthand for `$HOME`

---

### 8. Understanding Tilde Expansion

**Task**: 
a) Find in `bash` documentation what `~` means
b) What does `~wojnicki` mean?

**Solution:**

**To find documentation:**
```bash
man bash
```

Then search for "Tilde Expansion" by typing `/Tilde Expansion` and pressing Enter.

**Or search more broadly:**
```bash
man bash | grep -A 10 "Tilde Expansion"
```

**Answers:**

**Part a - What `~` means:**
The tilde (`~`) expands to the **current user's home directory**.

**Example:**
```bash
cd ~
# Equivalent to:
cd /home/your_username
```

**Part b - What `~wojnicki` means:**
`~username` expands to the **home directory of the specified user**.

**Example:**
```bash
cd ~wojnicki
# Expands to:
cd /home/wojnicki

ls ~wojnicki/wyk
# Expands to:
ls /home/wojnicki/wyk
```

**Explanation:**
- Tilde expansion is performed by the **shell**, not by individual commands
- It happens before the command executes
- Only works at the start of a word
- Very useful for portable scripts that work for any user

**Common mistakes:**
- Thinking `~` is part of the filesystem (it's a shell expansion)
- Not understanding that `~` alone means YOUR home, `~user` means THAT user's home
- Trying to use `~` in the middle of a path without proper context

---

### 9. Getting Today's Date

**Task**: What is today's date?

**Solution:**
```bash
date
```

**Sample output:**
```
Tue Dec 16 10:45:23 CET 2025
```

**For just the date (no time):**
```bash
date +%Y-%m-%d
```

**Output:**
```
2025-12-16
```

**Or more readable:**
```bash
date "+%A, %B %d, %Y"
```

**Output:**
```
Tuesday, December 16, 2025
```

**Explanation:**
- `date` without arguments shows current date and time in default format
- `date +FORMAT` allows custom formatting using format specifiers
- `%Y` = 4-digit year, `%m` = month (01-12), `%d` = day of month

**Common mistakes:**
- None really - this is straightforward!

---

### 10. Getting Only the Current Time (Hour)

**Task**: Display only the time (specifically the hour), not the date.

**Solution:**

**For hour and minute:**
```bash
date +%H:%M
```

**Output:**
```
10:45
```

**For hour, minute, and second:**
```bash
date +%H:%M:%S
```

**Output:**
```
10:45:23
```

**For just the hour:**
```bash
date +%H
```

**Output:**
```
10
```

**Explanation:**
- `+%H:%M` formats output as hours:minutes (24-hour format)
- `%H` = hours (00-23), `%M` = minutes (00-59), `%S` = seconds (00-59)
- The `+` sign indicates a custom format follows
- Quote the format if it contains spaces: `date "+%H:%M %p"`

**Alternative (12-hour format with AM/PM):**
```bash
date "+%I:%M %p"
```

**Output:**
```
10:45 AM
```

**Common mistakes:**
- Forgetting the `+` before the format string
- Using lowercase `%h` (which means abbreviated month, not hours)

---

### 11. Counting Home Directories with `wc`

**Task**: Count how many user home directories exist on the server.

**Solution:**
```bash
ls /home | wc -l
```

**Sample output:**
```
42
```

**Explanation:**
- `ls /home` lists all entries in the `/home` directory (each user has a directory there)
- `|` pipes that output to the next command
- `wc -l` counts the number of lines
- Each directory name is one line, so the count equals the number of home directories

**Alternative (more detailed):**
```bash
ls -1 /home | wc -l
```

The `-1` flag explicitly forces one item per line (though `ls` already does this when piping).

**Why this works:**
- By convention, each user gets a home directory in `/home`
- Directory names = usernames
- Counting directories in `/home` = counting users

**Common mistakes:**
- Using `wc` without `-l` (shows lines, words, and bytes - confusing)
- Forgetting to pipe: `ls /home wc -l` (wrong syntax)
- Including hidden files: `ls -a /home | wc -l` (adds `.` and `..`)

---

### 12. Creating Nested Directories Recursively

**Task**: Create `~/lab1/dziennik` structure in one command, working from any directory.

**Solution:**
```bash
mkdir -p ~/lab1/dziennik
```

**Explanation:**
- `mkdir` creates directories
- `-p` flag creates **parent directories as needed** (no error if they exist)
- `~` expands to your home directory, so command works from anywhere
- Without `-p`, you'd need two commands:
  ```bash
  mkdir ~/lab1
  mkdir ~/lab1/dziennik
  ```

**Why `-p` is essential:**
- If `lab1` doesn't exist, `mkdir ~/lab1/dziennik` would fail
- `-p` solves this: creates both `lab1` and `lab1/dziennik` in one go
- `-p` also doesn't error if directories already exist (safe to run multiple times)

**Verification:**
```bash
ls -lR ~/lab1
```

**Common mistakes:**
- Omitting `-p` (command fails if parent doesn't exist)
- Using relative path without `-p` and `~` (doesn't work from all directories)
- Typo: using `-r` instead of `-p`

---

### 13. Creating a Directory Named `tmp`

**Task**: Create a directory called `tmp` in your home directory.

**Solution:**
```bash
mkdir ~/tmp
```

**Or from home directory:**
```bash
cd
mkdir tmp
```

**Or using full path:**
```bash
mkdir /home/your_username/tmp
```

**Explanation:**
- Simple directory creation
- `~/tmp` ensures it's created in home directory regardless of current location
- No `-p` needed since we're only creating one level

**Verification:**
```bash
ls -ld ~/tmp
```

The `-d` flag shows the directory itself, not its contents.

**Common mistakes:**
- Creating it in the wrong location (not in home directory)
- Confusing it with system `/tmp` directory

---

### 14. Copying `/etc/passwd` to `~/tmp`

**Task**: Copy `/etc/passwd` to your `~/tmp` directory.

**Solution:**
```bash
cp /etc/passwd ~/tmp
```

**Or more explicit:**
```bash
cp /etc/passwd ~/tmp/passwd
```

**Explanation:**
- `cp source destination`
- When destination is a directory, file is copied **into** that directory
- File keeps its original name: `passwd`
- The file ends up at: `~/tmp/passwd`

**Verification:**
```bash
ls -l ~/tmp
```

**Common mistakes:**
- Reversing source and destination: `cp ~/tmp /etc/passwd` (wrong!)
- Not having read permission on `/etc/passwd` (rare - it's world-readable)

---

### 15. Copying with a New Name

**Task**: Copy `/etc/passwd` to `~/tmp` with name `passwd.system.orig`.

**Solution:**
```bash
cp /etc/passwd ~/tmp/passwd.system.orig
```

**Explanation:**
- When destination is a filename (not existing directory), that becomes the new name
- Source: `/etc/passwd`
- Destination: `~/tmp/passwd.system.orig`
- Now you have two files in `~/tmp`: `passwd` (from exercise 14) and `passwd.system.orig`

**Verification:**
```bash
ls -l ~/tmp/passwd*
```

This lists all files starting with "passwd" in `~/tmp`.

**Common mistakes:**
- Forgetting to include the full destination path
- Thinking the file is automatically renamed in place (no, it's a copy operation)

---

### 16. Renaming a File

**Task**: Rename `passwd.system.orig` to `passwd.orig`.

**Solution:**
```bash
mv ~/tmp/passwd.system.orig ~/tmp/passwd.orig
```

**Or from within ~/tmp:**
```bash
cd ~/tmp
mv passwd.system.orig passwd.orig
```

**Explanation:**
- `mv` is used for **both** moving and renaming
- When source and destination are in the same directory, it's a rename
- The file `passwd.system.orig` no longer exists; it's now `passwd.orig`

**Verification:**
```bash
ls -l ~/tmp/passwd*
```

You should see:
- `passwd`
- `passwd.orig`

But NOT `passwd.system.orig` (it was renamed).

**Common mistakes:**
- Using `cp` instead of `mv` (creates a copy instead of renaming)
- Forgetting to specify the full path

---

### 17. Overwriting with `cp`

**Task**: Copy `/etc/passwd-` to `~/tmp/passwd` (overwriting). Compare file sizes to see what happened.

**Solution:**

**Copy the file:**
```bash
cp /etc/passwd- ~/tmp/passwd
```

**Check what happened:**
```bash
ls -l /etc/passwd*
ls -l ~/tmp/passwd*
```

**Or compare directly:**
```bash
ls -l /etc/passwd /etc/passwd- ~/tmp/passwd
```

**Explanation:**
- `~/tmp/passwd` already existed (from exercise 14)
- `cp` **overwrites without warning** by default
- The old content is **lost forever** (no undo!)
- Comparing sizes shows they're different:
  - `/etc/passwd` - current password file
  - `/etc/passwd-` - backup (often slightly different in size/content)

**What happened:**
The original copy of `/etc/passwd` in `~/tmp/passwd` was replaced with the contents of `/etc/passwd-`.

**Common mistakes:**
- Not realizing that files were overwritten silently
- Assuming there's a way to recover the old content (there isn't)

---

### 18. Using Interactive Mode (`-i`)

**Task**: Repeat exercise 17 but with `-i` option. What's the difference?

**Solution:**
```bash
cp -i /etc/passwd- ~/tmp/passwd
```

**Output:**
```
cp: overwrite '~/tmp/passwd'? 
```

**Response:**
- Type `y` and press Enter to overwrite
- Type `n` and press Enter to cancel

**Explanation:**
- `-i` stands for "interactive"
- Prompts before overwriting existing files
- Gives you a chance to avoid accidental data loss
- Essential when learning or working with important files

**What `-i` does (from man page):**
"Prompt before overwrite"

**Comparison:**

| Command | Behavior |
|---------|----------|
| `cp source dest` | Silently overwrites if dest exists |
| `cp -i source dest` | Prompts before overwriting |

**Best practice:**
- Use `-i` when learning or when files are important
- Consider aliasing: `alias cp='cp -i'` in your `.bashrc`

**Common mistakes:**
- Typing full words like "yes" instead of just "y"
- Not understanding that without `-i`, there's no warning

---

### 19. Recursive Copy of Directory

**Task**: Copy entire `~/lab1` directory and all its contents to `~/tmp`.

**Solution:**
```bash
cp -r ~/lab1 ~/tmp
```

**Or more explicit:**
```bash
cp -r ~/lab1 ~/tmp/lab1
```

**Explanation:**
- `-r` (or `-R`) means "recursive" - copies directories and their contents
- Without `-r`, you get an error: "cp: -r not specified; omitting directory"
- After this command, you'll have:
  - Original: `~/lab1/dziennik/`
  - Copy: `~/tmp/lab1/dziennik/`

**Alternative (with more options):**
```bash
cp -rv ~/lab1 ~/tmp
```

The `-v` (verbose) flag shows what's being copied.

**Verification:**
```bash
ls -lR ~/tmp/lab1
```

**Common mistakes:**
- Forgetting `-r` (command fails for directories)
- Using `-r` with file copy (unnecessary but harmless)
- Not understanding that it copies the directory itself, not just its contents

---

### 20. Moving Files to Subdirectory

**Task**: 
a) Create `~/tmp/to_remove` directory
b) Move all files from `~/tmp` to `~/tmp/to_remove`

**Solution:**

**Part a - Create directory:**
```bash
mkdir ~/tmp/to_remove
```

**Part b - Move all files:**
```bash
mv ~/tmp/* ~/tmp/to_remove/
```

**Important:** This moves only files in `~/tmp`, not subdirectories (except `lab1` which is also moved).

**More careful approach (files only, not directories):**
```bash
cd ~/tmp
mv passwd passwd.orig ~/tmp/to_remove/
mv lab1 ~/tmp/to_remove/
```

**Explanation:**
- `*` is a wildcard matching all files/directories in `~/tmp`
- `mv ~/tmp/* ~/tmp/to_remove/` moves everything to the subdirectory
- Exception: Can't move `to_remove` into itself (you'll get an error for that)

**Verification:**
```bash
ls -la ~/tmp
ls -la ~/tmp/to_remove
```

You should see most items moved to `to_remove` subdirectory.

**Common mistakes:**
- Forgetting the trailing `/` (not required but good practice)
- Moving the `to_remove` directory into itself (causes error)
- Using `cp` instead of `mv` (leaves originals behind)

---

### 21. Listing Files in Home Directory from Anywhere

**Task**: Display all files in your home directory, regardless of current directory.

**Solution:**
```bash
ls ~
```

**Or:**
```bash
ls /home/your_username
```

**Or:**
```bash
ls $HOME
```

**More detailed listing:**
```bash
ls -la ~
```

**Explanation:**
- Using `~` or `$HOME` or absolute path makes the command work from **anywhere**
- Without these, `ls` would list the current directory
- `-la` shows all files (including hidden) in long format

**Examples from different directories:**
```bash
cd /etc
ls ~        # Still lists your home directory, not /etc

cd /tmp
ls -l ~     # Still lists your home directory, not /tmp
```

**Common mistakes:**
- Using just `ls` without path (lists current directory)
- Not understanding that `~` makes the command location-independent

---

### 22. Recursive Listing of Home Directory

**Task**: Display all files in all subdirectories of your home directory.

**Solution:**
```bash
ls -R ~
```

**Or more detailed:**
```bash
ls -lR ~
```

**Or with all files including hidden:**
```bash
ls -laR ~
```

**Explanation:**
- `-R` means "recursive" - lists directory contents, then recurses into subdirectories
- Shows the entire directory tree starting from home
- Output can be very long - consider piping to `less`:

```bash
ls -laR ~ | less
```

**Sample output structure:**
```
/home/student:
lab1  tmp  trash

/home/student/lab1:
dziennik

/home/student/lab1/dziennik:

/home/student/tmp:
to_remove
...
```

**Common mistakes:**
- Using `-r` (sort reverse) instead of `-R` (recursive)
- Not piping long output to `less`
- Confusing `-R` with `-r` (lowercase vs uppercase matters!)

---

### 23. Renaming a Directory

**Task**: Rename `~/tmp` to `~/trash`.

**Solution:**
```bash
mv ~/tmp ~/trash
```

**Or from home directory:**
```bash
cd
mv tmp trash
```

**Explanation:**
- `mv` renames directories the same way it renames files
- No special flag needed (unlike `cp` which requires `-r` for directories)
- After this, `~/tmp` no longer exists; it's now `~/trash`
- All contents are preserved

**Verification:**
```bash
ls -ld ~/trash
ls -ld ~/tmp  # Should give "No such file or directory"
```

**Common mistakes:**
- Thinking you need a special flag for directories (you don't with `mv`)
- Using `cp -r` then `rm -r` instead of just `mv` (inefficient)

---

### 24. Deleting Directory Tree

**Task**: Delete `~/trash` and all its contents (files and subdirectories).

**Solution:**
```bash
rm -r ~/trash
```

**Or more forceful (no prompts):**
```bash
rm -rf ~/trash
```

**Or interactive (prompts for each file):**
```bash
rm -ri ~/trash
```

**Explanation:**
- `rm` removes files
- `-r` (recursive) required to delete directories and their contents
- `-f` (force) suppresses prompts and errors
- `-i` (interactive) asks confirmation for each deletion

**⚠️ WARNING:**
- There is NO UNDO
- `-rf` is powerful and dangerous
- Triple-check before running `rm -rf`
- Never run `rm -rf /` or similar on important directories

**Verification:**
```bash
ls -l ~/trash  # Should say "No such file or directory"
```

**Common mistakes:**
- Forgetting `-r` (error: cannot remove directory)
- Using `rmdir` (only works on empty directories)
- Accidentally deleting wrong directory (check path carefully!)

---

### 25. Attempting to Remove `/tmp`

**Task**: Try to remove `/tmp` directory. What happens and why?

**Solution:**
```bash
rm -r /tmp
```

**Expected output:**
```
rm: cannot remove '/tmp': Permission denied
```

**Or:**
```
rm: cannot remove '/tmp/file1': Permission denied
rm: cannot remove '/tmp/file2': Permission denied
...
```

**Explanation:**

**Why it fails:**
1. **Permission protection**: You don't have write permission on `/tmp` itself
2. **System protection**: `/tmp` is a system directory, owned by root
3. **Other users' files**: `/tmp` contains files from multiple users; you can't delete others' files
4. **In use**: System processes may be using `/tmp`

**What `/tmp` is:**
- Temporary file storage for all users
- Cleared on reboot (usually)
- World-writable but with "sticky bit" (users can only delete their own files)
- Critical for system operation

**Permissions on `/tmp`:**
```bash
ls -ld /tmp
```

Output:
```
drwxrwxrwt 20 root root 4096 Dec 16 10:30 /tmp
```

The `t` at the end means "sticky bit" - special permission preventing users from deleting others' files.

**What you CAN do:**
- Create your own files in `/tmp`
- Delete your own files from `/tmp`
- Not affect other users' files

**Common mistakes:**
- Thinking you can delete system directories as a regular user
- Not understanding sticky bit permissions
- Trying to use `sudo` (which would work but is dangerous and inappropriate)

---

### 26. Exiting the Shell

**Task**: End your shell session. List at least 3 ways (1 keyboard shortcut, 2 commands).

**Solution:**

**Method 1 - Keyboard shortcut:**
```
CTRL + D
```

**Method 2 - exit command:**
```bash
exit
```

**Method 3 - logout command:**
```bash
logout
```

**Explanation:**

**CTRL + D:**
- Sends EOF (End of File) signal
- Shell interprets EOF on empty prompt as "end of input"
- Closes the shell session
- Most common and fastest method

**exit:**
- Built-in shell command
- Cleanly terminates the shell
- Can specify exit code: `exit 0` (success), `exit 1` (error)
- Works in all shells

**logout:**
- Similar to `exit` but specifically for login shells
- May not work in all shell types (sub-shells, scripts)
- Traditional Unix command

**Additional methods:**
- **Close terminal window**: (GUI method, not always clean)
- **Ctrl + Z then kill**: (not recommended - suspends rather than exits)

**Best practice:**
Use `CTRL + D` for speed, or `exit` for clarity in scripts.

**Common mistakes:**
- Using `CTRL + C` (interrupts command, doesn't exit shell)
- Closing window without proper logout (can leave processes running)
- Confusing `CTRL + D` with `CTRL + Z` (suspend vs exit)

---

## Summary Questions - Model Answers

### Q1: How does `apropos` differ from `man`?

**Answer:**

| Aspect | `apropos` | `man` |
|--------|-----------|-------|
| **Purpose** | Searches for commands | Displays documentation |
| **Input** | Keyword | Command name |
| **Output** | List of matching commands | Full manual page |
| **Use case** | "I need to find a command that does X" | "I know the command, need details" |

**Detailed explanation:**

**`apropos`:**
- Searches the NAME section of all man pages
- Returns a list of commands matching the keyword
- Used when you don't know the exact command name
- Example: `apropos "change password"` → finds `passwd`

**`man`:**
- Displays complete documentation for a specific command
- Shows syntax, options, examples, and detailed explanation
- Used when you know the command but need usage details
- Example: `man ls` → full documentation for `ls` command

**Workflow:**
1. Use `apropos` to discover relevant commands
2. Use `man` to learn how to use those commands

---

### Q2: Which tool is best for viewing text files with proper formatting?

**Answer:** **`less`** is the best choice for viewing text files while preserving formatting.

**Reasoning:**

**`less` advantages:**
- Bidirectional scrolling (forward and backward)
- Search functionality (`/` forward, `?` backward)
- Doesn't load entire file into memory (efficient for large files)
- Preserves formatting and line structure
- Doesn't modify the file

**Comparison:**

| Tool | Best for | Limitations |
|------|----------|-------------|
| `less` | Interactive viewing, large files | None significant |
| `more` | Simple viewing | Can't scroll backward |
| `cat` | Small files, piping | Scrolls off screen |
| Text editors | Editing files | Risk of accidental modification |

**Example:**
```bash
less /home/wojnicki/wyk/script/w1
```

**Alternative for special cases:**
- **Binary or complex formatting**: Use appropriate viewer (e.g., `hexdump` for binary)
- **Session files with control characters**: `less -r` (raw mode) to see colors

---

### Q3: Why is `/etc/passwd` so important in Linux?

**Answer:**

`/etc/passwd` is a **critical system file** containing essential user account information.

**What it contains:**
- Every user account on the system
- User IDs (UIDs) and primary group IDs (GIDs)
- Home directory paths
- Default shell for each user
- User information (GECOS field)

**Why it's critical:**

1. **Authentication**: Used during login process to verify user existence
2. **Authorization**: Maps usernames to UIDs (kernel uses UIDs for permissions)
3. **User environment**: Specifies home directory and default shell
4. **System operations**: Many programs read this file to resolve UID↔username

**Format:**
```
username:x:UID:GID:GECOS:home_directory:shell
```

**Example:**
```
student:x:1001:1001:John Smith:/home/student:/bin/bash
```

**Security note:**
- Historically stored encrypted passwords (hence the name)
- Modern systems use `/etc/shadow` for passwords (better security)
- `/etc/passwd` is world-readable but only root-writable
- Must be readable by all users (needed for basic system operations)

**Impact if corrupted:**
- Users can't log in
- Commands like `ls -l` can't show usernames
- System services may fail

---

### Q4: What happens when you copy a file to a location where a file with the same name exists?

**Answer:**

**Default behavior (`cp`):**
The existing file is **silently overwritten** without warning. The original content is **permanently lost**.

**Example:**
```bash
# ~/tmp/passwd exists with content A
cp /etc/passwd- ~/tmp/passwd
# ~/tmp/passwd now has content from /etc/passwd-
# Original content A is GONE FOREVER
```

**Prevention:**

**Option 1 - Interactive mode:**
```bash
cp -i source destination
# Prompts: "overwrite destination? y/n"
```

**Option 2 - No-clobber mode:**
```bash
cp -n source destination
# Skips if destination exists (no overwrite, no prompt)
```

**Option 3 - Backup mode:**
```bash
cp -b source destination
# Creates backup: destination~ before overwriting
```

**Best practices:**
1. **Use `-i` when learning** or working with important files
2. **Check destination** before copying: `ls -l destination`
3. **Consider backups** of important files
4. **Test with safe files** first

**Comparison with `mv`:**
Same behavior - `mv` also overwrites without warning unless you use `-i`.

---

### Q5: What does `|` (pipe) actually do?

**Answer:**

The **pipe operator (`|`)** connects the **standard output** of one command to the **standard input** of another command.

**Conceptual model:**
```
command1 | command2
```

1. `command1` produces output
2. Instead of displaying on screen, output goes into a "pipe"
3. `command2` reads from the pipe as if it were keyboard input
4. Final output appears on screen

**Real example:**
```bash
ls /home | wc -l
```

**What happens:**
1. `ls /home` produces list of directories (one per line)
2. That list goes into pipe (not shown on screen yet)
3. `wc -l` reads the list and counts lines
4. `wc -l` outputs the count: `42`

**Key characteristics:**

- **Concurrent execution**: Both commands run simultaneously (not sequential)
- **Efficient**: No temporary files needed
- **Composable**: Can chain multiple pipes: `cmd1 | cmd2 | cmd3 | cmd4`
- **Text-based**: Works with text streams (Unix philosophy)

**Complex example:**
```bash
cat /etc/passwd | grep student | cut -d: -f1 | sort | wc -l
```

This pipeline:
1. `cat` reads `/etc/passwd`
2. `grep` filters lines containing "student"
3. `cut` extracts first field (username)
4. `sort` sorts alphabetically
5. `wc -l` counts results

**Why pipes are powerful:**
They embody the Unix philosophy: small, specialized tools working together to solve complex problems.

---

## Final Tips for Lab 1

### Common Pitfalls to Avoid

1. **Not using tab completion**: Press Tab to complete filenames and commands
2. **Forgetting `-i` with destructive commands**: Use `-i` until comfortable
3. **Not checking current directory**: Use `pwd` frequently
4. **Ignoring error messages**: Read them carefully - they often explain the problem
5. **Not reading man pages**: Most answers are in the documentation

### Commands You Should Now Understand

✅ Documentation: `man`, `apropos`, `whatis`  
✅ Navigation: `pwd`, `cd`, `ls`  
✅ File operations: `cp`, `mv`, `rm`, `mkdir`, `rmdir`  
✅ Text viewing: `less`, `more`, `cat`, `wc`, `echo`  
✅ System: `date`, `passwd`  
✅ Concepts: pipes (`|`), tilde expansion (`~`), paths

### Practice Suggestions

1. **Experiment safely**: Create a test directory and practice commands there
2. **Read man pages**: Pick one command per day and read its full manual
3. **Use the history**: Press ↑ to recall previous commands
4. **Make mistakes**: Best way to learn (just not with `rm -rf`)
5. **Ask why**: Understand not just "what" but "why" commands work

---

## Quick Reference Card

### Essential Commands

```bash
# Documentation
man command          # Read manual
apropos keyword      # Find commands
whatis command       # Brief description

# Navigation
pwd                  # Current directory
cd path              # Change directory
ls -la               # List all files (long format)

# File Operations
cp source dest       # Copy file
cp -r dir dest       # Copy directory
mv source dest       # Move/rename
rm file              # Remove file
rm -r dir            # Remove directory
mkdir dir            # Create directory
mkdir -p path/to/dir # Create nested directories

# Text Viewing
less file            # View file (interactive)
cat file             # Display file
wc -l file           # Count lines

# Pipes
command1 | command2  # Connect output to input
```

### Important Flags

```bash
-i    # Interactive (prompt before overwrite)
-r    # Recursive (for directories)
-a    # All (including hidden files)
-l    # Long format
-p    # Create parents / preserve
-v    # Verbose
-f    # Force
```

---

**End of Model Answers for Lab 1**

You now have complete solutions with explanations for all exercises. Practice these commands until they become second nature - they form the foundation of all Unix work!
