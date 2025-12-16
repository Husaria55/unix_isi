# Model Answers & Walkthrough - Lab 4: VIM Text Editor

## Overview

This document provides solutions to Lab 4 exercises focused on mastering VIM text editor through vimtutor and practical exercises.

---

## Introductory Question

### Q: What is VIM, and what is VI?

**Answer:**

**VI (Visual Editor):**
- **Created:** 1976 by Bill Joy
- **Purpose:** Screen-oriented text editor for Unix
- **Features:** Modal editing, keyboard navigation
- **Availability:** Present on every Unix/Linux system
- **Standard:** POSIX-compliant

**VIM (Vi IMproved):**
- **Created:** 1991 by Bram Moolenaar
- **Purpose:** Enhanced version of vi with many improvements
- **Added features:**
  - Syntax highlighting
  - Multiple undo/redo
  - Visual mode
  - Split windows
  - Plugins and scripting
  - GUI version (gvim)
  - Better help system

**Relationship:**
- VIM is **backward compatible** with vi
- Most modern systems alias `vi` to `vim`
- Learning VIM teaches you vi as well

**Check what you have:**
```bash
vi --version        # Shows if it's vim or true vi
which vi            # Shows path (often -> vim)
vim --version       # Shows vim version
```

**Why it matters:**
- **vi** guaranteed on any Unix system (servers, routers, etc.)
- **vim** more user-friendly and powerful
- Skills transfer between both

---

## Exercise Solutions

### 1. Complete vimtutor Lessons 1-6

**Task:** Complete all exercises in vimtutor chapters 1-6.

**How to start:**
```bash
vimtutor
```

**What vimtutor covers:**

**Lesson 1: Basic Movement and Editing**
- Moving cursor: `h`, `j`, `k`, `l`
- Entering/exiting VIM: `vim filename`, `:q!`, `:wq`
- Deleting characters: `x`
- Inserting text: `i`

**Lesson 2: Deletion Commands**
- Delete word: `dw`
- Delete to end of line: `d$`
- Count + motion: `2w`, `3e`
- Count + delete: `d2w`
- Delete line: `dd`
- Undo: `u`
- Undo line: `U`
- Redo: `Ctrl+r`

**Lesson 3: Put, Replace, Change**
- Put (paste): `p`
- Replace: `r`
- Change: `cw`, `c$`
- Change more: `ce`, `c3w`

**Lesson 4: Search and Replace**
- Search: `/pattern`, `?pattern`
- Next/Previous: `n`, `N`
- Go to matching bracket: `%`
- Substitute: `:s/old/new`, `:%s/old/new/g`
- Execute external command: `:!command`

**Lesson 5: Writing Files**
- Write (save): `:w filename`
- Save part of file: `:'<,'>w filename`
- Insert file content: `:r filename`
- Read command output: `:r !ls`

**Lesson 6: Open, Append, Replace Mode**
- Open new line: `o`, `O`
- Append: `a`, `A`
- Replace mode: `R`
- Copy/yank: `y`, `yy`
- Set options: `:set xxx`

**Estimated time:** 25-30 minutes

**Tips for vimtutor:**
- Don't skip sections
- Practice each command multiple times
- Take notes of commands you forget
- Repeat lessons if needed

---

### 2. Creating a Music File

**Task:** Create file "muz" with 4 favorite music titles using vim or cat.

**Method 1: Using vim**

**Solution:**
```bash
vim muz
```

Then in vim:
1. Press `i` to enter insert mode
2. Type your favorite songs:
```
Song Title 1
Song Title 2
Song Title 3
Song Title 4
```
3. Press `Esc` to return to normal mode
4. Type `:wq` to save and quit

**Method 2: Using cat**

**Solution:**
```bash
cat > muz
Song Title 1
Song Title 2
Song Title 3
Song Title 4
[Press Ctrl+D]
```

**Verification:**
```bash
cat muz
# Should display your 4 songs
```

**Example content:**
```
Bohemian Rhapsody
Stairway to Heaven
Hotel California
Imagine
```

---

### 3. Adding Header Line in VIM

**Task:** Open "muz" with vim, add "ulubione utwory" as first line, save.

**Solution:**
```bash
vim muz
```

**VIM commands:**
```
gg          # Go to first line
O           # Open line above (enters insert mode)
ulubione utwory
<Esc>       # Exit insert mode
:w          # Save
```

**Alternative method:**
```
gg          # Go to first line
I           # Insert at start of line
ulubione utwory<Enter>
<Esc>       # Exit insert mode
:w          # Save
```

**Explanation:**
- `gg` - Jump to first line
- `O` - Opens new line **above** current line (enters insert mode)
- Type the text
- `Esc` - Return to normal mode
- `:w` - Write (save) file

**Resulting file structure:**
```
ulubione utwory
Song Title 1
Song Title 2
Song Title 3
Song Title 4
```

---

### 4. Advanced Editing in VIM

**Task:** Open muz starting at line 3, enable line numbers, add content, indent, save as muz1.

**Solution:**

**Open at line 3:**
```bash
vim +3 muz
```

**Or open normally and jump:**
```bash
vim muz
# Then: 3G or :3
```

**VIM command sequence:**

**1. Enable line numbers:**
```
:set number
# or short form:
:set nu
```

**2. Add new line below current:**
```
o               # Opens line below, enters insert mode
Song Title 5
<Esc>
```

**3. Add artist lines before each song:**

Position on song line, then:
```
O               # Open line above
Artist Name
<Esc>
j               # Move down to next song
```

Repeat for each song.

**4. Indent song titles:**

Position on first song title:
```
>>              # Indent current line (one tab right)
```

Repeat for other songs, or:
```
V               # Visual line mode
3j              # Select 4 lines
>               # Indent all
```

**5. Check line count:**
```
Ctrl+g          # Shows line count in status
# or
:set ruler      # Shows line/column in bottom right
```

**6. Save as muz1:**
```
:w muz1
```

**Expected structure:**
```
 1  ulubione utwory
 2  Artist 1
 3      Song Title 1
 4  Artist 1
 5      Song Title 2
 6  Artist 2
 7      Song Title 3
 8  Artist 2
 9      Song Title 4
10  Artist 3
11      Song Title 5
```

**Alternative for mass indenting:**
```
/Song           # Search for "Song"
>>              # Indent
n               # Next match
.               # Repeat indent
n               # Next match
.               # Repeat indent
```

---

### 5. Editing muz1 - Character Operations

**Task:** Various character-level edits in muz1, save as muz2.

**Solution:**

```bash
vim muz1
```

**1. Move to "utwory" in line 1:**
```
gg              # First line
/utwory         # Search for "utwory"
# or
0               # Start of line
w               # Move word by word until before "utwory"
```

**2. Delete to end of line:**
```
D               # Delete to end
# or
d$              # Delete to end (same result)
```

**3. Toggle case of previous word ("ulubione"):**
```
b               # Back one word
g~~             # Toggle case of entire line
# or
b               # Back one word
viw~            # Visual select inner word, toggle case
```

**4. Go to last line and delete it:**
```
G               # Last line
dd              # Delete line
```

**5. Save as muz2:**
```
:w muz2
```

**Explanation of case toggle:**
- `~` - Toggle case of character under cursor
- `g~~` - Toggle case of entire line
- `viw~` - Visual select inner word and toggle
- Result: "ulubione" → "ULUBIONE" (or vice versa)

**Expected result:**
Line 1 becomes:
```
ULUBIONE
```
(or similar depending on case toggle)

---

### 6. Converting Tabs to Commas and Joining Lines

**Task:** Open muz1, change tabs to commas, join lines so each line is "Artist, Songs...", save as muz3.

**Solution:**

```bash
vim muz1
```

**1. Replace tabs with commas:**
```
:%s/\t/,/g
# Explanation:
# % - entire file
# s - substitute
# \t - tab character
# , - comma
# g - global (all on each line)
```

**2. Join lines (Artist + Song):**

Position on artist line:
```
J               # Join with next line
J               # Join with next line if multiple songs
```

Repeat for each artist, or use macro:
```
qa              # Start recording macro in register 'a'
J               # Join lines
j               # Move down
q               # Stop recording
4@a             # Repeat 4 times (or however many needed)
```

**Alternative - Join multiple lines at once:**
```
V               # Visual line mode
2j              # Select 3 lines (artist + 2 songs)
J               # Join all selected
```

**3. Save as muz3:**
```
:w muz3
```

**Expected result:**
```
Artist 1, Song Title 1, Song Title 2
Artist 2, Song Title 3, Song Title 4
Artist 3, Song Title 5
```

**Note:** Original file had tabs for indentation. If there were no tabs, adjust the substitution or manually join lines.

---

### 7. Restructure to One Song Per Line

**Task:** Open muz3, copy and rearrange so each line is "Artist, Song" (one song per line), save as muz4.

**Solution:**

```bash
vim muz3
```

**Strategy:** For each line with multiple songs, split into separate lines.

**Example transformation:**
```
Before: Artist 1, Song Title 1, Song Title 2
After:  Artist 1, Song Title 1
        Artist 1, Song Title 2
```

**Method 1: Manual copying**

For line "Artist 1, Song Title 1, Song Title 2":

1. **Copy artist name:**
```
0               # Start of line
y/,             # Yank to first comma
```

2. **Position at second song:**
```
f,              # Find first comma
f,              # Find second comma
```

3. **Create new line with artist:**
```
o               # Open line below
<Esc>           # Exit insert
p               # Paste artist
A,<space>       # Append comma-space
```

4. **Cut second song from above line and paste:**
```
k               # Up to original line
f,              # To second comma
d$              # Delete to end
j               # Down to new line
$               # End of line
p               # Paste song
```

**Method 2: Using substitute and macros**

```
# Replace ", " after second occurrence with newline + artist
:%s/\(^[^,]*,\)\([^,]*\),/\1\2\r\1/g
```

This is complex regex - manual approach may be clearer.

**Method 3: Step by step**

For each line with multiple songs:
1. Copy artist name
2. Put cursor on second comma
3. Open new line below
4. Paste artist
5. Cut song from line above
6. Paste after artist

**Final save:**
```
:w muz4
```

**Expected muz4 content:**
```
Artist 1, Song Title 1
Artist 1, Song Title 2
Artist 2, Song Title 3
Artist 2, Song Title 4
Artist 3, Song Title 5
```

---

### 8. Check File Type

**Task:** Use `file` command to check muz4.

**Solution:**
```bash
file muz4
```

**Expected output:**
```
muz4: ASCII text
```

**Or:**
```
muz4: UTF-8 Unicode text
```

**Explanation:**
- `file` examines file content
- Determines it's a text file
- Shows character encoding

**Alternative checks:**
```bash
file -b muz4            # Brief (no filename)
# Output: ASCII text

file -i muz4            # MIME type
# Output: muz4: text/plain; charset=us-ascii
```

---

### 9. Sort Lines from Line 2 Onwards

**Task:** Open muz4, sort lines starting from line 2 (skip header).

**Solution:**

```bash
vim muz4
```

**VIM commands:**
```
:2,$sort
# Explanation:
# 2,$ - Range from line 2 to end ($)
# sort - Sort command
```

**Alternative method:**
```
gg              # Go to first line
j               # Move to line 2
V               # Visual line mode
G               # Select to end
:sort           # Sort selection
```

**To save:**
```
:w
```

**Result:**
Lines 2 onwards are sorted alphabetically:
```
ulubione utwory
Artist 1, Song Title 1
Artist 1, Song Title 2
Artist 2, Song Title 3
Artist 2, Song Title 4
Artist 3, Song Title 5
```

Becomes (after sort):
```
ulubione utwory
Artist 1, Song Title 1
Artist 1, Song Title 2
Artist 2, Song Title 3
Artist 2, Song Title 4
Artist 3, Song Title 5
```

**For numeric sort:**
```
:2,$sort n
```

**For reverse sort:**
```
:2,$sort!
```

---

### 10. Fast Search and Replace

**Task:** Open muz, find "utwory", replace with "kawałki", exit without saving - as fast as possible.

**Command provided:**
```bash
s=`date +'%s'`; vim +/utwory muz; expr `date +'%s'` - $s
```

**Solution sequence:**

**What the command does:**
1. Records start time
2. Opens vim at first "utwory" occurrence  
3. You edit
4. Calculates elapsed time

**Fastest VIM sequence:**

When vim opens (cursor at "utwory"):
```
cwkawałki<Esc>:q!<Enter>
```

**Breakdown:**
- `cw` - Change word (deletes "utwory", enters insert mode)
- `kawałki` - Type replacement
- `<Esc>` - Return to normal mode
- `:q!` - Quit without saving
- `<Enter>` - Execute command

**Alternative (using substitute):**
```
:%s/utwory/kawałki/g<Enter>:q!<Enter>
```

**Ultra-fast (if you're quick):**
```
Rcawałki<Esc>:q!<Enter>
```

**Timing yourself:**
- Under 5 seconds: Excellent
- Under 10 seconds: Good
- Under 15 seconds: Learning
- Over 15 seconds: Practice more!

**Tips for speed:**
- Know the command before starting
- Don't pause to think
- Muscle memory is key
- Practice the sequence multiple times

---

## Summary Questions

### Q1: How to exit VIM?

**Answer:**

**Normal mode commands:**

| Command | Effect |
|---------|--------|
| `:q` | Quit (fails if unsaved changes) |
| `:q!` | Quit without saving (force) |
| `:wq` | Write (save) and quit |
| `:x` | Save if changed, then quit |
| `ZZ` | Save and quit (normal mode, no colon) |
| `ZQ` | Quit without saving (normal mode) |

**Step by step:**
1. Press `Esc` (ensure you're in normal mode)
2. Type `:q!` (force quit, no save)
3. Press `Enter`

**Common mistakes:**
- Trying to quit while in insert mode (press `Esc` first)
- Typing `quit` instead of `:q`
- Forgetting the `!` when changes are unsaved

---

### Q2: How to add text in an existing line?

**Answer:**

**Multiple methods depending on where:**

**Insert before cursor:**
```
i         # Insert mode at cursor
```

**Append after cursor:**
```
a         # Append mode after cursor
```

**Insert at line start:**
```
I         # Insert at first non-blank character
```

**Append at line end:**
```
A         # Append at end of line
```

**Insert below current line:**
```
o         # Open new line below
```

**Insert above current line:**
```
O         # Open new line above
```

**Example scenarios:**

**Add to end of line:**
```
A           # Jump to end and enter insert mode
, more text
<Esc>
```

**Insert at beginning:**
```
I           # Jump to start and enter insert mode
prefix
<Esc>
```

**Insert at specific position:**
```
/word       # Find "word"
i           # Insert before "word"
new text
<Esc>
```

---

### Q3: How to copy and paste?

**Answer:**

**Basic copy (yank) and paste:**

| Command | Action |
|---------|--------|
| `yy` | Yank (copy) current line |
| `p` | Paste after cursor/line |
| `P` | Paste before cursor/line |
| `yw` | Yank word |
| `y$` | Yank to end of line |
| `yG` | Yank to end of file |

**Visual mode copy:**
```
v           # Enter visual mode
(move to select)
y           # Yank selection
p           # Paste
```

**Examples:**

**Copy and paste line:**
```
yy          # Copy line
j           # Move down
p           # Paste below
```

**Copy word and paste:**
```
yw          # Yank word
w           # Move to next word
P           # Paste before
```

**Copy multiple lines:**
```
3yy         # Copy 3 lines
```

**Copy to named register:**
```
"ayy        # Copy line to register 'a'
"ap         # Paste from register 'a'
```

**Visual block copy:**
```
Ctrl+v      # Block visual
(select)
y           # Yank
p           # Paste
```

---

### Q4: How to replace more than one occurrence?

**Answer:**

**Using substitute command:**

**Replace all in file:**
```
:%s/old/new/g
# % - entire file
# s - substitute
# old - pattern to find
# new - replacement
# g - global (all on each line)
```

**Replace with confirmation:**
```
:%s/old/new/gc
# c - confirm each replacement
# You'll be prompted: y/n/a/q/l
```

**Replace in range:**
```
:5,10s/old/new/g        # Lines 5-10
:.,+5s/old/new/g        # Current line + 5 more
:'<,'>s/old/new/g       # Visual selection
```

**Replace only first on each line (no 'g'):**
```
:%s/old/new/
```

**Case-insensitive replace:**
```
:%s/old/new/gi
```

**Examples:**

**Replace all "TODO" with "DONE":**
```
:%s/TODO/DONE/g
```

**Replace and confirm:**
```
:%s/color/colour/gc
```

**Replace in selected lines:**
```
V               # Visual line mode
5j              # Select 6 lines
:s/old/new/g    # Replace in selection
```

---

### Q5: How to undo and redo?

**Answer:**

| Command | Action |
|---------|--------|
| `u` | Undo last change |
| `U` | Undo all changes on current line |
| `Ctrl+r` | Redo (undo the undo) |
| `.` | Repeat last change |

**Examples:**

**Simple undo/redo:**
```
dd          # Delete line (oops!)
u           # Undo - line restored
Ctrl+r      # Redo - line deleted again
```

**Multiple undo:**
```
u           # Undo once
u           # Undo again
u           # Undo again
```

**Undo entire line:**
```
(make several changes on line)
U           # Reverts all changes to line
```

**Undo tree navigation:**
```
:earlier 1m     # Go back 1 minute
:later 30s      # Go forward 30 seconds
:undo 5         # Go to undo state 5
```

**Check undo history:**
```
:undolist       # Show undo tree
```

---

### Q6: How to navigate through search results?

**Answer:**

**Search forward:**
```
/pattern        # Search forward for "pattern"
```

**Search backward:**
```
?pattern        # Search backward for "pattern"
```

**Navigate matches:**
| Command | Action |
|---------|--------|
| `n` | Next match (same direction) |
| `N` | Previous match (opposite direction) |
| `*` | Search word under cursor (forward) |
| `#` | Search word under cursor (backward) |

**Example workflow:**

```
/error          # Search for "error"
# (cursor moves to first match)
n               # Next occurrence
n               # Next occurrence
N               # Previous occurrence (go back one)
```

**Case-insensitive search:**
```
/pattern\c      # This search only
:set ic         # Set ignore case globally
:set noic       # Unset ignore case
```

**Search and highlight:**
```
/pattern        # Search
:set hlsearch   # Highlight all matches
:nohlsearch     # Clear highlights (or :noh)
```

**Search history:**
```
/               # Start search
Up arrow        # Previous search
Down arrow      # Next search
```

**Search for word boundaries:**
```
/\<word\>       # Exact word "word"
# Won't match "words" or "sword"
```

**Clear search highlighting:**
```
:noh            # Clear highlights
# or
:nohlsearch
```

---

## VIM Quick Reference

### Essential Commands

**Mode switching:**
```
Esc             # Normal mode
i/a/o/O         # Insert mode
v/V/Ctrl+v      # Visual mode
:               # Command mode
```

**Navigation:**
```
h/j/k/l         # Left/Down/Up/Right
w/b             # Word forward/backward
gg/G            # First/Last line
0/$             # Line start/end
```

**Editing:**
```
x               # Delete character
dd              # Delete line
yy              # Copy line
p/P             # Paste after/before
u               # Undo
Ctrl+r          # Redo
```

**Search/Replace:**
```
/pattern        # Search forward
n/N             # Next/Previous
:%s/old/new/g   # Replace all
```

**Save/Exit:**
```
:w              # Save
:q              # Quit
:wq             # Save and quit
:q!             # Quit without saving
```

---

**End of Model Answers for Lab 4**
