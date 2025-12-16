# Model Answers - Lab 5: Process Management

## Introduction Question

### What does `kill` actually do?

**Common Misconception:** `kill` terminates processes.

**Reality:** `kill` sends **signals** to processes. The default signal happens to be SIGTERM (terminate), but `kill` can send any signal.

**Examples:**
```bash
kill 1234           # Send SIGTERM (terminate) to PID 1234
kill -STOP 1234     # Send SIGSTOP (suspend) to PID 1234
kill -CONT 1234     # Send SIGCONT (resume) to PID 1234
kill -9 1234        # Send SIGKILL (force terminate) to PID 1234
```

**Analogy:** Think of `kill` as a messenger that delivers different types of messages (signals) to processes. The message could be "please stop" (SIGTERM), "freeze" (SIGSTOP), "continue" (SIGCONT), or "die now" (SIGKILL).

---

## Task Solutions

### Task 1: Basic Job Control with vim

**Objective:** Learn Ctrl+Z, jobs, and fg commands.

**Step-by-step solution:**

**a) Open file and move cursor:**
```bash
vim muz4
```
Once in vim:
- Use arrow keys or `h` to move cursor before the word "utwory" in line 1
- Or use `/utwory` then press Enter, then `h` to move back one character

**b) Suspend vim with Ctrl+Z:**
- Press `Ctrl+Z`
- You'll see message: `[1]+ Stopped    vim muz4`
- You're back at the terminal prompt

**c) Try opening vim again:**
```bash
vim muz4
```

**What happens?**
Vim shows a warning message:

```
Found a swap file by the name ".muz4.swp"
...
(1) Another program may be editing the same file...
(2) An edit session for this file crashed.
...
```

**Why?**
- The first vim instance is **suspended** (stopped), not closed
- Vim creates a **swap file** (.muz4.swp) when editing
- The swap file still exists because the first vim is still running
- Vim warns you that the file might already be open

**What to do:**
- Press `q` to quit this second vim attempt
- Don't open multiple vim instances on the same file

**d) Check job list:**
```bash
jobs
```

**Output:**
```
[1]+ Stopped    vim muz4
```

**Explanation:**
- `[1]` - Job number 1
- `+` - Current job (default for fg/bg)
- `Stopped` - Job state
- `vim muz4` - The command

**e) Restore suspended process:**
```bash
fg
```

**What happens:**
- The stopped vim returns to foreground
- You're back in vim exactly where you left off
- Cursor is still before "utwory"

**f) Close vim:**
```
:q
```
(Or `:wq` if you made changes)

**g) Check jobs again:**
```bash
jobs
```

**Output:**
(Empty - no output)

**Why?**
- Closing vim terminated the process
- Terminated processes are removed from job list
- Job list only shows active (running/stopped) jobs in current shell

**Key Lessons:**
- `Ctrl+Z` suspends (doesn't close) programs
- Suspended jobs remain in memory
- Suspended vim locks the file (swap file persists)
- `fg` resumes exactly where you left off
- Closing a program removes it from job list

---

### Task 2: Background Processes and Resource Usage

**Objective:** Understand the difference between stopped and background-running processes.

**Step-by-step solution:**

**a) Download program:**
```bash
cd ~
wget https://gitlab.kis.agh.edu.pl/miroman/unix_isi/-/raw/main/Lab/zasoby/calc.c
```

**b) Compile:**
```bash
gcc -o prog calc.c
```

**Explanation:**
- `gcc` - GNU C Compiler
- `-o prog` - Output file named "prog"
- `calc.c` - Source file

**c) Run program:**
```bash
./prog
```

**What you see:**
- Nothing! The terminal appears frozen
- Program is running CPU-intensive calculations
- No output to screen
- Terminal is occupied

**d) Check resource usage (second terminal):**

Open new terminal/PuTTY session, then:
```bash
top -b -n 1 -u `whoami`
```

**Command breakdown:**
- `top` - Show process activity
- `-b` - Batch mode (non-interactive)
- `-n 1` - One iteration only
- `-u `whoami`` - Filter to your username
  - `` `whoami` `` - Command substitution: inserts your username

**Output excerpt:**
```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
12345 student   20   0    2236    752    672 R  99.9   0.0   0:15.23 prog
```

**Key observation:**
- **%CPU: 99.9%** - Program uses nearly 100% of one CPU core
- **S: R** - State is "Running"
- Program is consuming significant resources

**e) Suspend prog (first terminal):**
- Return to first terminal (where prog is running)
- Press `Ctrl+Z`

**Output:**
```
^Z
[1]+ Stopped    ./prog
```

Now you can type commands again.

**f) Check resources again (second terminal):**
```bash
top -b -n 1 -u `whoami`
```

**Output excerpt:**
```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
12345 student   20   0    2236    752    672 T   0.0   0.0   0:15.23 prog
```

**Key observations:**
- **%CPU: 0.0%** - No CPU usage!
- **S: T** - State is "Stopped" (not Running)
- Program is suspended - not using resources

**Why?**
- Stopped processes receive no CPU time
- OS doesn't schedule them
- They remain in memory but frozen

**g) Resume in background (first terminal):**
```bash
bg
```

**Output:**
```
[1]+ ./prog &
```

**What changed:**
- `&` indicates background job
- Process resumed execution
- Terminal is **free** - you can type commands
- Program continues calculating

**Verify with top (second terminal):**
```bash
top -b -n 1 -u `whoami`
```

**Output:**
```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
12345 student   20   0    2236    752    672 R  99.9   0.0   0:32.45 prog
```

**Key observations:**
- **%CPU: 99.9%** - Back to high usage!
- **S: R** - Running again
- **TIME+** increased - program continued from where it stopped

**h) Bring to foreground and terminate:**
```bash
fg
```
(prog returns to foreground, terminal blocked)

Press `Ctrl+C`

**Output:**
```
^C
```

Program terminates.

**Summary of states:**

| Action | State | %CPU | Terminal Free? |
|--------|-------|------|----------------|
| `./prog` | Running | 99.9% | No (blocked) |
| `Ctrl+Z` | Stopped | 0.0% | Yes |
| `bg` | Running | 99.9% | Yes |
| `fg` | Running | 99.9% | No (blocked) |

**Key Lesson:** Background running (`bg`) ≠ Stopped (`Ctrl+Z`)
- **Stopped:** Process frozen, no CPU
- **Background:** Process runs, CPU used, terminal free

---

### Task 3: Controlling Multiple Jobs

**Objective:** Experiment with multiple jobs and job specifications.

**Experiment:**

**Start multiple processes:**
```bash
vim file1.txt       # Started first
Ctrl+Z              # Suspend
# [1]+ Stopped    vim file1.txt

vim file2.txt       # Started second
Ctrl+Z              # Suspend
# [2]+ Stopped    vim file2.txt

sleep 300 &         # Started third (background)
# [3] 12345
```

**Check jobs:**
```bash
jobs
```

**Output:**
```
[1]-  Stopped    vim file1.txt
[2]+  Stopped    vim file2.txt
[3]   Running    sleep 300 &
```

**Symbols explained:**
- `[1]`, `[2]`, `[3]` - Job numbers
- `+` - **Current job** - default for `fg` and `bg` without arguments
- `-` - **Previous job** - second most recent
- No symbol - Older jobs

**Test default fg:**
```bash
fg
```
Brings job `[2]` (marked with `+`) to foreground.

**Quit vim:**
```
:q
```

**Check jobs again:**
```bash
jobs
```

**Output:**
```
[1]+  Stopped    vim file1.txt
[3]-  Running    sleep 300 &
```

**Notice:** Job `[1]` is now current (`+`), job `[3]` is previous (`-`)

**Control specific jobs:**

**Bring job 1 to foreground:**
```bash
fg %1
```
Vim file1.txt appears.

**Quit vim:**
```
:q
```

**Resume sleep in background (already running, but as example):**
```bash
bg %3
```
(No visible effect since already running)

**Kill job 3:**
```bash
kill %3
```

**Verify:**
```bash
jobs
```
Should be empty.

**Advanced job specifications:**

```bash
vim myprogram.c
Ctrl+Z
vim otherfile.txt
Ctrl+Z
sleep 100 &

jobs
# [1]   Stopped    vim myprogram.c
# [2]-  Stopped    vim otherfile.txt
# [3]+  Running    sleep 100 &
```

**Various ways to refer to jobs:**
```bash
fg %1               # Job 1 by number
fg %vim             # Job starting with "vim"
fg %?program        # Job containing "program"
fg %%               # Current job (same as %+)
fg %+               # Current job
fg %-               # Previous job
```

**Plus/minus meaning:**

| Symbol | Meaning | Use |
|--------|---------|-----|
| `+` | Current job | Used by `fg` or `bg` without arguments |
| `-` | Previous job | Becomes current when current job finishes |
| (none) | Older job | Must specify explicitly |

**How + and - change:**
1. Most recently stopped/backgrounded job gets `+`
2. Previous `+` job becomes `-`
3. When current (`+`) job terminates, previous (`-`) becomes current (`+`)

**Example flow:**
```bash
vim a
Ctrl+Z
# [1]+ Stopped    vim a

vim b
Ctrl+Z
# [1]- Stopped    vim a
# [2]+ Stopped    vim b    <- Most recent

fg        # Brings job [2]
:q

jobs
# [1]+ Stopped    vim a    <- Now current
```

---

### Task 4: Finding Memory-Intensive Process

**Objective:** Identify process using most memory; understand RSS vs VSZ.

**Solution:**

**Method 1: Using ps with sorting:**
```bash
ps aux --sort=-%mem | head -5
```

**Explanation:**
- `ps aux` - All processes, detailed
- `--sort=-%mem` - Sort by memory (descending)
- `head -5` - Show top 5

**Example output:**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      1234  0.0 15.3 245678 123456 ?      Ssl  Jan15  10:23 /usr/bin/mysqld
www-data  5678  0.1 12.1 198765 98765 ?       S    Jan16   5:42 /usr/sbin/apache2
...
```

**Method 2: Using top:**
```bash
top
```
- Press `M` (uppercase) to sort by memory
- Press `q` to quit

**Method 3: Using htop:**
```bash
htop
```
- Press `F6`, select `PERCENT_MEM`, press Enter
- Or click "MEM%" column header with mouse

**Answer:** The process with highest %MEM value (in example above: mysqld at 15.3%)

---

**RSS vs VSZ Difference:**

**VSZ (Virtual Memory Size):**
- **Total virtual memory** allocated to process
- Includes:
  - Code
  - Data
  - Shared libraries
  - Memory mapped files
  - **Swap space**
- May **not be in physical RAM**
- Can be larger than physical RAM
- **Not actual resource usage**

**RSS (Resident Set Size):**
- **Actual physical RAM** used by process
- Currently **in RAM** (not swapped)
- **Real memory usage**
- More important for performance
- Cannot exceed physical RAM

**Analogy:**
- **VSZ:** Size of all books you own (some in storage, some at home)
- **RSS:** Books actually on your desk right now

**Example:**
```
  PID    VSZ   RSS COMMAND
 1234 245678 123456 mysqld
```
- VSZ: 245 MB total virtual memory allocated
- RSS: 123 MB actually in physical RAM
- Difference (122 MB): Could be shared libraries, mapped files, or swapped

**Which matters more?**
- **RSS** - For understanding current RAM pressure
- **VSZ** - For understanding total memory requirements

**Checking specific process:**
```bash
ps -o pid,vsz,rss,comm -p PID
```

**Example:**
```bash
ps -o pid,vsz,rss,comm -p 1234

  PID    VSZ   RSS COMMAND
 1234 245678 123456 mysqld
```

---

### Task 5: Finding CPU-Intensive Process

**Objective:** Identify process using most CPU.

**Solution:**

**Method 1: Using ps with sorting:**
```bash
ps aux --sort=-%cpu | head -5
```

**Explanation:**
- `--sort=-%cpu` - Sort by CPU descending
- `head -5` - Top 5 processes

**Example output:**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
student  12345 99.8  0.0   2236   752 pts/0    R+   14:23   1:34 ./prog
root      5678  2.1  1.5 456789 12345 ?        Ssl  Jan15  45:23 /usr/bin/Xorg
```

**Answer:** Process with highest %CPU (in example: ./prog at 99.8%)

**Method 2: Using top:**
```bash
top
```
- By default, sorted by %CPU
- Or press `P` (uppercase) to ensure CPU sorting
- Look at %CPU column

**Method 3: Using htop:**
```bash
htop
```
- Press `F6`, select `PERCENT_CPU`, Enter
- Or click "CPU%" column

**Understanding %CPU:**
- **< 100%:** Using less than one full CPU core
- **= 100%:** Using one full CPU core
- **> 100%:** Multi-threaded, using multiple cores
  - 200% = 2 cores
  - 400% = 4 cores

**Example:**
```
%CPU
99.8    # Single-threaded, almost 1 core
345.2   # Multi-threaded, ~3.5 cores
```

**Snapshot vs continuous:**
```bash
# One snapshot
ps aux --sort=-%cpu | head

# Continuous monitoring
top
# or
watch -n 1 'ps aux --sort=-%cpu | head -10'
```

---

### Task 6: Total Number of Processes

**Objective:** Count all processes in system.

**Solution:**

**Method 1: Using ps and wc:**
```bash
ps aux | wc -l
```

**Explanation:**
- `ps aux` - All processes
- `wc -l` - Count lines

**Important:** Subtract 1 for header line!

**Example output:**
```
234
```

**Actual process count:** 234 - 1 = **233 processes**

**Method 2: Without header:**
```bash
ps aux --no-headers | wc -l
```

**Output:**
```
233
```

**Direct count without adjustment needed.**

**Method 3: Using ps with specific output:**
```bash
ps -e | wc -l
```

**Subtract 1 for header.**

**Method 4: From /proc:**
```bash
ls /proc | grep '^[0-9]*$' | wc -l
```

**Explanation:**
- Each process has `/proc/PID` directory
- `grep '^[0-9]*$'` - Only numeric directories (PIDs)
- Counts PID directories

**Method 5: Using pgrep:**
```bash
pgrep -c '.'
```

**Explanation:**
- `pgrep` - Process grep
- `-c` - Count matches
- `'.'` - Match anything (all processes)

**Verification with top:**
```bash
top -b -n 1 | head -3
```

**Look for line:**
```
Tasks: 233 total, 2 running, 231 sleeping, 0 stopped, 0 zombie
```

**Answer:** Check "Tasks: XXX total"

---

### Task 7: Counting Your Processes

**Objective:** Count processes belonging to you; identify programs.

**Solution:**

**Count my processes:**

**Method 1: Using ps and wc:**
```bash
ps -u $USER --no-headers | wc -l
```

**Explanation:**
- `ps -u $USER` - Processes by current user
- `$USER` - Environment variable with username
- `--no-headers` - No header line
- `wc -l` - Count

**Example output:**
```
15
```

**Method 2: Using explicit username:**
```bash
ps -u student --no-headers | wc -l
```

**Method 3: Using whoami:**
```bash
ps -u `whoami` --no-headers | wc -l
```

---

**List programs:**

**Method 1: Show command names:**
```bash
ps -u $USER -o comm
```

**Example output:**
```
COMMAND
bash
bash
vim
sleep
top
ps
```

**Method 2: Unique programs only:**
```bash
ps -u $USER -o comm --no-headers | sort | uniq
```

**Example output:**
```
bash
ps
sleep
top
vim
```

**Explanation:**
- `-o comm` - Show only command name
- `--no-headers` - Remove header
- `sort` - Alphabetize
- `uniq` - Remove duplicates

**Method 3: With count of each:**
```bash
ps -u $USER -o comm --no-headers | sort | uniq -c
```

**Example output:**
```
      2 bash
      1 ps
      1 sleep
      1 top
      1 vim
```

**Shows:** 2 bash instances, 1 of each other program

**Method 4: Full command with arguments:**
```bash
ps -u $USER -o pid,comm,args
```

**Example output:**
```
  PID COMMAND         COMMAND
 1234 bash            -bash
 5678 vim             vim file.txt
 9012 sleep           sleep 300
```

**Difference:**
- `comm` - Program name only
- `args` - Full command with arguments

---

### Task 8: Processes Without Terminal

**Objective:** Count processes without controlling terminal.

**Solution:**

**Understanding TTY column:**
- **pts/0, pts/1, tty1**: Process has terminal
- **?**: No controlling terminal (daemons, services)

**Count processes with ? in TTY:**

**Method 1: Using grep:**
```bash
ps aux | grep '?' | wc -l
```

**⚠️ Problem:** This is **incorrect** - `grep '?'` matches literally everything because `?` is a regex wildcard!

**Correct method 1:**
```bash
ps aux | grep ' ? ' | wc -l
```

**Explanation:**
- Space before and after `?` matches exact TTY column value
- Subtract 1 for header (or use --no-headers)

**Correct method 2: Using awk:**
```bash
ps aux --no-headers | awk '$7 == "?" {print}' | wc -l
```

**Explanation:**
- `$7` - 7th column (TTY)
- `== "?"` - Exactly equals ?
- Counts matching lines

**Correct method 3: Direct ps filtering:**
```bash
ps -e -o tty,comm | grep '?' | wc -l
```

Subtract 1 for header.

**Example output:**
```
145
```

**What are these processes?**

**View some:**
```bash
ps aux | grep ' ? ' | head -10
```

**Example:**
```
USER       PID TTY   STAT  COMMAND
root         1 ?     Ss    /sbin/init
root         2 ?     S     [kthreadd]
root      1234 ?     Ssl   /usr/sbin/sshd
root      5678 ?     Ss    /usr/sbin/cron
mysql     9012 ?     Ssl   /usr/sbin/mysqld
```

**Common daemon processes:**
- **sshd** - SSH server
- **cron** - Scheduled tasks
- **mysqld** - Database server
- **apache2** - Web server
- **systemd** - Init system
- Kernel threads (in brackets)

**Why no terminal?**
- Started at boot (before user login)
- Background services
- Not associated with user session
- Continue running after logout

---

### Task 9: Background Process with Output Redirection

**Objective:** Understand process states and output with datecnt program.

**Setup:**

**a) Download:**
```bash
wget https://gitlab.kis.agh.edu.pl/miroman/unix_isi/-/raw/main/Lab/zasoby/datecnt
```

**b) Make executable:**
```bash
chmod +x datecnt
```

**c) Test run (foreground):**
```bash
./datecnt
```

**Output:**
```
2024-01-20 14:30:15
2024-01-20 14:30:16
2024-01-20 14:30:17
...
```

Press `Ctrl+C` to stop.

---

**Task steps:**

**1) Run in background with output redirection:**
```bash
./datecnt > dates.txt &
```

**Output:**
```
[1] 12345
```

**Explanation:**
- `./datecnt` - Run program
- `> dates.txt` - Redirect stdout to file
- `&` - Background execution
- `[1] 12345` - Job 1, PID 12345

**2) Check if data appears:**
```bash
# Wait a few seconds
sleep 3

# View file
cat dates.txt
```

**Expected output:**
```
2024-01-20 14:35:01
2024-01-20 14:35:02
2024-01-20 14:35:03
...
```

**Or watch in real-time:**
```bash
tail -f dates.txt
```

**Answer:** YES, data appears continuously. Press `Ctrl+C` to stop tail.

**3) Bring to foreground:**
```bash
fg
```

**What you see:**
- datecnt is running in foreground
- But output still goes to file, not screen!
- Terminal appears frozen (occupied by datecnt)

**4) Suspend with Ctrl+Z:**
```bash
Ctrl+Z
```

**Output:**
```
^Z
[1]+ Stopped    ./datecnt > dates.txt
```

**Check file:**
```bash
tail dates.txt
```

**Latest timestamp is old (stopped when you pressed Ctrl+Z).**

**Wait and check again:**
```bash
sleep 5
tail dates.txt
```

**Same timestamps - no new data!**

**Answer:** NO, data does NOT appear when stopped.

**Why?**
- Process is suspended
- Not executing any code
- Not producing output
- File remains unchanged

**5) Resume in background:**
```bash
bg
```

**Output:**
```
[1]+ ./datecnt > dates.txt &
```

**Check file immediately:**
```bash
tail -f dates.txt
```

**You see:**
- New timestamps appearing
- Data resumes from where it stopped
- Continuous updates

**Answer:** YES, data appears when running in background.

**6) Using kill with signals (second terminal):**

**Find PID:**
```bash
jobs -l
# [1]+ 12345 Running    ./datecnt > dates.txt &
```

**Or:**
```bash
pidof datecnt
# 12345
```

**Send SIGSTOP:**
```bash
kill -STOP 12345
```

**Check in first terminal:**
```bash
jobs
# [1]+ Stopped    ./datecnt > dates.txt
```

**Check file (first terminal):**
```bash
tail dates.txt
# Timestamps stopped
```

**Wait and check:**
```bash
sleep 5
tail dates.txt
# Same timestamps - no new data
```

**Send SIGCONT (second terminal):**
```bash
kill -CONT 12345
```

**Check jobs (first terminal):**
```bash
jobs
# [1]+ Running    ./datecnt > dates.txt &
```

**Check file:**
```bash
tail -f dates.txt
# New timestamps appearing!
```

**Cleanup:**
```bash
kill %1
# or
kill 12345
```

---

**7) Repeat without redirection:**

**Run in background:**
```bash
./datecnt &
```

**What happens:**
```
[1] 12346
2024-01-20 14:40:01
2024-01-20 14:40:02
2024-01-20 14:40:03
...
```

**Timestamps appear on your terminal!**

**Problem:** Output mixes with your commands:
```
$ ls2024-01-20 14:40:04

dates.txt  datecnt  2024-01-20 14:40:05
prog
$2024-01-20 14:40:06
```

**Messy and confusing!**

**Suspend:**
```bash
fg
Ctrl+Z
```

**Output stops.**

**Resume in background:**
```bash
bg
```

**Output appears on terminal again (messy).**

**Kill:**
```bash
kill %1
```

---

**Summary - Data appears when:**

| State | Data Appears? |
|-------|---------------|
| Foreground | Yes (to file if redirected) |
| Background running | Yes (to file if redirected) |
| Stopped | **No** |
| Killed (SIGSTOP) | **No** |
| Resumed (SIGCONT) | Yes |

**Key Insight:** Only **running** processes produce output. Stopped/suspended processes are frozen.

---

### Task 10: Background Process and Terminal Closure

**Objective:** Understand what happens to background jobs when terminal closes.

**Part A: Process survives while terminal open**

**1) Run command:**
```bash
(sleep 10 && echo "Ble") &
```

**Output:**
```
[1] 12347
```

**Explanation:**
- `( )` - Subshell grouping
- `sleep 10` - Wait 10 seconds
- `&&` - Then (if successful)
- `echo "Ble"` - Print message
- `&` - Run in background

**2) Wait for completion:**

**Check status:**
```bash
jobs
# [1]+ Running    (sleep 10 && echo "Ble") &
```

**After ~10 seconds:**
```bash
Ble
[1]+ Done    (sleep 10 && echo "Ble")
```

**Command completed successfully.**

---

**Part B: What happens when terminal closes?**

**1) Run command:**
```bash
(sleep 10 && echo "Ble") &
# [1] 12348
```

**2) Get PID:**
```bash
jobs -l
# [1]+ 12348 Running    (sleep 10 && echo "Ble") &
```

**3) Immediately close terminal:**
- Click X button
- Type `exit`
- Close PuTTY window

**4) Open new terminal and check:**
```bash
ps -p 12348
# PID TTY          TIME CMD
# (no output)
```

**Or:**
```bash
ps aux | grep 12348
# (only grep process itself)
```

**Answer:** Process is **NOT running**. It was **terminated**.

---

**Why did it die?**

**When terminal closes:**
1. Shell sends **SIGHUP** (hangup) signal to all jobs
2. Default action for SIGHUP: **terminate**
3. Background jobs receive signal
4. Process terminates

**Exception:** Some programs handle SIGHUP differently (e.g., `nohup`, daemons).

---

**How to make process survive terminal closure?**

**Method 1: Using `nohup`:**
```bash
nohup sleep 30 &
```

**Output:**
```
nohup: ignoring input and appending output to 'nohup.out'
[1] 12349
```

**Close terminal, reopen, check:**
```bash
ps aux | grep 'sleep 30'
# student  12349  0.0  0.0   7236   812 ?        S    14:50   0:00 sleep 30
```

**Still running!** TTY is `?` (no terminal).

**Method 2: Using `disown`:**
```bash
sleep 30 &
# [1] 12350

disown
```

**Close terminal, reopen, check:**
```bash
ps aux | grep 'sleep 30'
# Still running
```

**Method 3: Using `tmux` or `screen`:**
```bash
tmux
# Inside tmux:
sleep 30
# Detach: Ctrl+b, then d
```

**Close terminal, reopen:**
```bash
tmux attach
# Process still running
```

---

**Summary:**

| Scenario | Process Survives? |
|----------|-------------------|
| Wait for completion | Yes (completed before close) |
| Close terminal immediately | **No** (receives SIGHUP) |
| Use `nohup` | **Yes** |
| Use `disown` | **Yes** |
| Use `tmux`/`screen` | **Yes** |

---

## Summary Questions

### Question 1: Ctrl+Z vs Ctrl+C

**What does Ctrl+Z do?**

**Short answer:** **Suspends** (stops) the foreground process.

**Detailed explanation:**
- Sends **SIGTSTP** signal (Terminal Stop)
- Process enters **Stopped** state (T)
- Process remains in memory
- Does NOT terminate
- Job stays in job list
- Can be resumed with `fg` or `bg`
- State preserved exactly

**Example:**
```bash
vim file.txt
Ctrl+Z
# [1]+ Stopped    vim file.txt

jobs
# [1]+ Stopped    vim file.txt

fg  # Returns to vim exactly where you left off
```

**Use cases:**
- Temporarily pause task
- Access terminal for quick command
- Return to task later
- Switch between multiple tasks

---

**What does Ctrl+C do?**

**Short answer:** **Terminates** the foreground process.

**Detailed explanation:**
- Sends **SIGINT** signal (Interrupt)
- Process should terminate
- Clean shutdown (if program handles signal properly)
- Process removed from memory
- Job removed from job list
- Cannot be resumed (process ended)
- Data may be lost if not saved

**Example:**
```bash
./prog
Ctrl+C
^C

jobs
# (empty - process terminated)

# Cannot resume - process gone
```

**Use cases:**
- Stop unwanted process
- Cancel long-running command
- Exit hung program
- Interrupt infinite loop

---

**Comparison:**

| Aspect | Ctrl+Z | Ctrl+C |
|--------|--------|--------|
| Signal | SIGTSTP (20) | SIGINT (2) |
| Effect | Suspend | Terminate |
| Process state | Stopped (T) | Terminated |
| Memory | Retained | Freed |
| Resumable | Yes (fg/bg) | No |
| Job list | Remains | Removed |
| Data | Preserved | May be lost |
| Use | Temporary pause | End process |

**Analogy:**
- **Ctrl+Z:** Pause button - can press play later
- **Ctrl+C:** Stop button - restarts from beginning

---

### Question 2: Terminal Closure Effects

**What happens when you close a terminal with running command?**

**Short answer:** Background processes are **terminated** (by default).

**Detailed explanation:**

**1. Foreground process:**
- Already occupies terminal
- Immediately terminates when terminal closes
- No special signal needed (loses input/output)

**2. Background process:**
- **Receives SIGHUP** (Hangup) signal
- Default action: **terminate**
- Process ends even though it was "background"

**Mechanism:**
1. User closes terminal window
2. Terminal emulator sends SIGHUP to shell
3. Shell propagates SIGHUP to all child processes
4. Processes receive SIGHUP
5. Default handler terminates process

---

**Experiment demonstration:**

```bash
# Start long process
sleep 100 &
# [1] 12345

# Check it's running
ps -p 12345
# Running

# Close terminal
exit

# Open new terminal
ps -p 12345
# No such process (terminated)
```

---

**Why does this happen?**

**Historical reason:** "Hangup" refers to old telephone modems.
- When phone line disconnected, modem "hung up"
- Terminal sent SIGHUP signal
- Processes knew connection lost
- Clean termination expected

**Modern reason:** Prevents orphaned processes
- User logs out
- Background jobs should stop
- Prevents resource waste
- Security: user session ended

---

**How to keep processes running after logout?**

**Method 1: `nohup` (No Hangup)**
```bash
nohup command &
```

**Effect:**
- Ignores SIGHUP
- Continues after terminal closes
- Output to `nohup.out`

**Example:**
```bash
nohup sleep 100 &
# Close terminal
# Process survives
```

---

**Method 2: `disown`**
```bash
command &
disown
```

**Effect:**
- Removes job from shell's job table
- Shell won't send SIGHUP
- Process continues

**Example:**
```bash
sleep 100 &
disown
# Close terminal
# Process survives
```

---

**Method 3: `tmux` or `screen`**
```bash
tmux
command
# Detach: Ctrl+b, d
```

**Effect:**
- Terminal multiplexer session persists
- Processes run inside tmux
- Can reattach later

**Example:**
```bash
tmux
./long_process
# Ctrl+b, d (detach)
# Close terminal
# Open new terminal
tmux attach
# Process still running!
```

---

**Method 4: Systemd service (for servers)**
```bash
# Create service unit
sudo systemctl start myservice
```

**Effect:**
- Process managed by systemd
- Independent of terminal
- Survives reboots

---

**Summary table:**

| Scenario | Process Survives? | Solution |
|----------|-------------------|----------|
| Terminal closed (default) | **No** | Use workaround |
| `nohup command &` | **Yes** | Ignores SIGHUP |
| `command &` then `disown` | **Yes** | Removed from job table |
| Inside `tmux`/`screen` | **Yes** | Persistent session |
| Systemd service | **Yes** | System-managed |

---

**Best practices:**

**For temporary tasks:**
```bash
nohup long_task > output.log 2>&1 &
```

**For development work:**
```bash
tmux
# Work inside tmux session
```

**For production services:**
```bash
# Use systemd or similar init system
```

**Key principle:** Default behavior protects system, but you have options when needed.

---

## Common Mistakes

### 1. Confusing Stopped with Background

**Mistake:**
```bash
vim file.txt
Ctrl+Z
# Think file is being edited in background
```

**Why it's wrong:**
- Vim is **stopped**, not running
- No changes happening
- File not being saved

**Correct understanding:**
- Background running: Process executes
- Stopped: Process frozen

---

### 2. Using kill without checking

**Mistake:**
```bash
killall python    # Kills ALL python processes!
```

**Why dangerous:**
- Might kill important processes
- Other users affected on shared server
- Data loss

**Better approach:**
```bash
# Find specific PID
ps aux | grep myscript.py
# Kill specific process
kill PID
```

---

### 3. Expecting background processes to survive logout

**Mistake:**
```bash
./long_process &
# Log out
# Expect process to continue
```

**Why it fails:**
- Background ≠ Independent of terminal
- SIGHUP terminates process

**Solution:**
```bash
nohup ./long_process &
```

---

### 4. Force killing (kill -9) immediately

**Mistake:**
```bash
kill -9 PID    # First resort
```

**Why bad:**
- No cleanup
- Files not saved
- Connections not closed
- Database corruption possible

**Better:**
```bash
kill PID        # Try graceful first
sleep 2
kill -9 PID     # Only if necessary
```

---

### 5. Not redirecting nohup output

**Mistake:**
```bash
nohup command &
# nohup.out fills disk over time
```

**Better:**
```bash
nohup command > /dev/null 2>&1 &
# Or specify output file
nohup command > output.log 2>&1 &
```

---

## Key Takeaways

1. **Ctrl+Z suspends, Ctrl+C terminates**
2. **Stopped processes use no CPU**
3. **Background processes run but free terminal**
4. **jobs, fg, bg control job flow**
5. **kill sends signals (not just terminates)**
6. **SIGTERM (15) is graceful, SIGKILL (9) is forceful**
7. **Terminal closure kills background jobs (use nohup)**
8. **top/htop/ps for monitoring**
9. **RSS is physical memory, VSZ is virtual**
10. **nice/renice control priority**

Process management is essential for efficient system use and multitasking!
