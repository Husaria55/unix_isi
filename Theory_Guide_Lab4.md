# Theory Guide - Lab 4: VIM Text Editor

## Table of Contents
1. [Introduction to VIM](#introduction-to-vim)
2. [VIM Philosophy](#vim-philosophy)
3. [Modal Editing](#modal-editing)
4. [Basic Navigation](#basic-navigation)
5. [Editing Commands](#editing-commands)
6. [Search and Replace](#search-and-replace)
7. [Working with Lines](#working-with-lines)
8. [Visual Mode](#visual-mode)
9. [Buffers and Windows](#buffers-and-windows)
10. [VIM Configuration](#vim-configuration)

---

## Introduction to VIM

### What is VIM?

**VIM** = "Vi IMproved" - an enhanced version of the classic Unix vi editor.

**Key characteristics:**
- **Modal editor** - Different modes for different tasks
- **Keyboard-centric** - Minimal mouse use
- **Highly efficient** - Once mastered
- **Ubiquitous** - Available on virtually all Unix systems
- **Extensible** - Plugins, scripts, customization

### VI vs VIM

| Aspect | vi | vim |
|--------|----|----|
| **Age** | 1976 | 1991 |
| **Features** | Basic | Extended |
| **Availability** | Every Unix | Most systems |
| **Syntax highlighting** | No | Yes |
| **Multiple undo** | Limited | Unlimited |
| **Visual mode** | No | Yes |
| **Plugins** | No | Yes |

**In practice:** On modern systems, `vi` is often symlinked to `vim`.

---

## VIM Philosophy

### Why VIM Exists

**Design goals:**
1. **Efficiency** - Minimize keystrokes for common tasks
2. **Composability** - Combine simple commands for complex operations
3. **No mode switching cost** - Once learned, faster than GUI editors
4. **Remote editing** - Works over SSH, terminal-only environments

### The VIM Way

**Principle:** Commands are **composable** - they combine in logical ways.

**Pattern:** `[count] operator [motion]`

**Examples:**
- `d3w` - Delete 3 words
- `c$` - Change to end of line
- `y2j` - Yank (copy) 2 lines down
- `>G` - Indent from here to end of file

---

## Modal Editing

### The Three Main Modes

| Mode | Purpose | How to Enter | Indicator |
|------|---------|--------------|-----------|
| **Normal** | Navigation, commands | `Esc` | Default (-- INSERT -- not shown) |
| **Insert** | Text entry | `i`, `a`, `o`, etc. | `-- INSERT --` |
| **Visual** | Text selection | `v`, `V`, `Ctrl+v` | `-- VISUAL --` |
| **Command** | Ex commands | `:` | `:` prompt at bottom |

### Normal Mode (Command Mode)

**Default mode** - for navigation and text manipulation.

**Purpose:**
- Move cursor
- Delete, copy, paste
- Undo, redo
- Search
- Execute commands

**Key principle:** Keys are commands, not text input.

### Insert Mode

**Purpose:** Type text like a normal editor.

**Entering insert mode:**

| Command | Effect |
|---------|--------|
| `i` | Insert before cursor |
| `a` | Append after cursor |
| `I` | Insert at line start |
| `A` | Append at line end |
| `o` | Open line below |
| `O` | Open line above |
| `s` | Substitute character |
| `S` | Substitute line |

**Exiting insert mode:** Press `Esc`

### Visual Mode

**Purpose:** Select text for operations.

**Types:**

| Command | Mode | Selects |
|---------|------|---------|
| `v` | Character-wise | Individual characters |
| `V` | Line-wise | Entire lines |
| `Ctrl+v` | Block | Rectangular block |

**Once selected:** Use operators like `d` (delete), `y` (yank), `c` (change)

### Command Mode

**Purpose:** Execute Ex commands.

**Entering:** Type `:` in Normal mode

**Common commands:**
- `:w` - Write (save)
- `:q` - Quit
- `:wq` - Write and quit
- `:q!` - Quit without saving
- `:set` - Change settings
- `:help` - Get help

---

## Basic Navigation

### Character Movement

| Key | Movement |
|-----|----------|
| `h` | Left |
| `j` | Down |
| `k` | Up |
| `l` | Right |

**Mnemonic:** j looks like down arrow

**With counts:**
- `5j` - Move down 5 lines
- `10l` - Move right 10 characters

### Word Movement

| Key | Movement |
|-----|----------|
| `w` | Next word start |
| `W` | Next WORD start (space-delimited) |
| `e` | Next word end |
| `E` | Next WORD end |
| `b` | Previous word start |
| `B` | Previous WORD start |

**Word vs WORD:**
- **word:** delimited by punctuation: `hello-world` (2 words)
- **WORD:** delimited by spaces: `hello-world` (1 WORD)

### Line Movement

| Key | Movement |
|-----|----------|
| `0` | Start of line |
| `^` | First non-blank character |
| `$` | End of line |
| `g_` | Last non-blank character |

### Document Movement

| Key | Movement |
|-----|----------|
| `gg` | First line |
| `G` | Last line |
| `5G` | Line 5 |
| `:42` | Line 42 |
| `Ctrl+f` | Page forward |
| `Ctrl+b` | Page backward |
| `Ctrl+d` | Half page down |
| `Ctrl+u` | Half page up |
| `H` | Top of screen (High) |
| `M` | Middle of screen |
| `L` | Bottom of screen (Low) |

### Search Movement

| Key | Movement |
|-----|----------|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Next match |
| `N` | Previous match |
| `*` | Search word under cursor (forward) |
| `#` | Search word under cursor (backward) |

**Example:**
```
/TODO     # Search for "TODO"
n         # Next occurrence
N         # Previous occurrence
```

---

## Editing Commands

### Operators

**Operators** act on text defined by motions.

| Operator | Action |
|----------|--------|
| `d` | Delete (cut) |
| `y` | Yank (copy) |
| `c` | Change (delete and insert) |
| `>` | Indent right |
| `<` | Indent left |
| `=` | Auto-indent |
| `g~` | Toggle case |
| `gu` | Lowercase |
| `gU` | Uppercase |

### Delete Commands

| Command | Action |
|---------|--------|
| `x` | Delete character |
| `X` | Delete before cursor |
| `dd` | Delete line |
| `D` | Delete to end of line |
| `dw` | Delete word |
| `d$` | Delete to line end |
| `d0` | Delete to line start |
| `dG` | Delete to file end |
| `d3w` | Delete 3 words |

### Change Commands

**Change** = Delete + Enter insert mode

| Command | Action |
|---------|--------|
| `cc` | Change line |
| `C` | Change to line end |
| `cw` | Change word |
| `c$` | Change to end |
| `ciw` | Change inner word |
| `ci"` | Change inside quotes |
| `cit` | Change inside tag |

### Copy (Yank) and Paste

| Command | Action |
|---------|--------|
| `yy` | Yank line |
| `Y` | Yank line (same as yy) |
| `yw` | Yank word |
| `y$` | Yank to line end |
| `p` | Paste after cursor |
| `P` | Paste before cursor |
| `"ayy` | Yank line to register 'a' |
| `"ap` | Paste from register 'a' |

### Undo and Redo

| Command | Action |
|---------|--------|
| `u` | Undo |
| `Ctrl+r` | Redo |
| `U` | Undo all changes on line |

### Repeat

| Command | Action |
|---------|--------|
| `.` | Repeat last change |

**Power of dot:**
```
dw    # Delete word
.     # Delete another word
.     # Delete another word
```

### Insert Mode Commands

| Command | Action |
|---------|--------|
| `i` | Insert before cursor |
| `a` | Append after cursor |
| `I` | Insert at line start |
| `A` | Append at line end |
| `o` | Open line below |
| `O` | Open line above |
| `s` | Substitute character |
| `S` | Substitute line |
| `R` | Replace mode |

---

## Search and Replace

### Searching

**Forward search:**
```
/pattern
```

**Backward search:**
```
?pattern
```

**Navigate matches:**
- `n` - Next match
- `N` - Previous match

**Case sensitivity:**
- `/pattern\c` - Case-insensitive
- `/pattern\C` - Case-sensitive

### Replace (Substitute)

**Syntax:**
```
:[range]s/old/new/[flags]
```

**Ranges:**
- `:%s` - Entire file
- `:s` - Current line
- `:5,10s` - Lines 5-10
- `:.,$s` - Current line to end
- `:'<,'>s` - Visual selection (auto-inserted)

**Flags:**
- `g` - Global (all on line)
- `c` - Confirm each
- `i` - Ignore case
- `I` - Case-sensitive

**Examples:**

```vim
:s/old/new/           " First on line
:s/old/new/g          " All on line
:%s/old/new/g         " All in file
:%s/old/new/gc        " All in file, confirm each
:5,10s/old/new/g      " Lines 5-10
:%s/old/new/gi        " Case-insensitive, all
```

### Special Characters in Search

| Character | Meaning |
|-----------|---------|
| `.` | Any character |
| `*` | Zero or more |
| `^` | Start of line |
| `$` | End of line |
| `\<` | Start of word |
| `\>` | End of word |
| `[]` | Character class |
| `\|` | OR |

**Examples:**
```vim
/^Error           " Lines starting with "Error"
/error$           " Lines ending with "error"
/\<the\>          " The word "the" (not "there")
/[0-9]\+          " One or more digits
/foo\|bar         " "foo" or "bar"
```

---

## Working with Lines

### Line Operations

| Command | Action |
|---------|--------|
| `dd` | Delete line |
| `yy` | Copy line |
| `cc` | Change line |
| `p` | Paste below |
| `P` | Paste above |
| `J` | Join with next line |
| `>>` | Indent right |
| `<<` | Indent left |
| `==` | Auto-indent |

### Multiple Lines

**With counts:**
```
3dd        " Delete 3 lines
5yy        " Copy 5 lines
4>>        " Indent 4 lines
```

**With ranges:**
```
:5,10d     " Delete lines 5-10
:5,10y     " Copy lines 5-10
:5,10>>    " Indent lines 5-10
```

### Sorting Lines

```vim
:sort           " Sort all lines
:5,10sort       " Sort lines 5-10
:'<,'>sort      " Sort visual selection
:sort u         " Sort and remove duplicates
:sort n         " Numeric sort
```

### Line Numbers

**Display:**
```vim
:set number       " Show line numbers
:set nu           " Short form
:set nonumber     " Hide
:set nu!          " Toggle
```

**Relative numbers:**
```vim
:set relativenumber    " Relative to cursor
:set rnu               " Short form
```

---

## Visual Mode

### Entering Visual Mode

| Command | Mode |
|---------|------|
| `v` | Character-wise |
| `V` | Line-wise |
| `Ctrl+v` | Block (column) |

### Visual Mode Operations

**Once in visual mode:**
1. Move to select text
2. Apply operator

| Command | Action |
|---------|--------|
| `d` | Delete selection |
| `y` | Yank selection |
| `c` | Change selection |
| `>` | Indent right |
| `<` | Indent left |
| `~` | Toggle case |
| `u` | Lowercase |
| `U` | Uppercase |

**Example workflow:**
```
V        " Start line-wise visual
3j       " Select 4 lines total
>        " Indent them
```

### Block Visual Mode

**Purpose:** Edit rectangular selections (columns).

**Use cases:**
- Add comment markers to multiple lines
- Edit tabular data
- Bulk insertions at column

**Example:**
```
Ctrl+v      " Block visual
3j          " Down 3 lines
I# <Esc>    " Insert "# " at start of each
```

---

## Buffers and Windows

### Buffers

**Buffer** = An in-memory representation of a file.

| Command | Action |
|---------|--------|
| `:e filename` | Edit (open) file |
| `:bnext` or `:bn` | Next buffer |
| `:bprevious` or `:bp` | Previous buffer |
| `:bd` | Delete (close) buffer |
| `:buffers` or `:ls` | List buffers |

### Windows (Splits)

**Split view** - View multiple files/locations simultaneously.

| Command | Action |
|---------|--------|
| `:split` or `:sp` | Horizontal split |
| `:vsplit` or `:vs` | Vertical split |
| `Ctrl+w s` | Horizontal split |
| `Ctrl+w v` | Vertical split |
| `Ctrl+w w` | Switch windows |
| `Ctrl+w q` | Close window |
| `Ctrl+w h/j/k/l` | Navigate windows |

---

## VIM Configuration

### The .vimrc File

**Purpose:** Customize VIM behavior.

**Location:** `~/.vimrc`

### Common Settings

```vim
" Basic settings
set number              " Show line numbers
set relativenumber      " Relative line numbers
set tabstop=4           " Tab width
set shiftwidth=4        " Indent width
set expandtab           " Spaces instead of tabs
set autoindent          " Auto-indent new lines
set smartindent         " Smart indenting
set hlsearch            " Highlight search results
set incsearch           " Incremental search
set ignorecase          " Case-insensitive search
set smartcase           " Case-sensitive if uppercase used
syntax on               " Syntax highlighting
set ruler               " Show cursor position
set showcmd             " Show command in status
set wildmenu            " Command completion
set mouse=a             " Enable mouse
```

### Useful Mappings

```vim
" Map jj to escape insert mode
inoremap jj <Esc>

" Easy save
nnoremap <C-s> :w<CR>

" Toggle line numbers
nnoremap <F2> :set nu!<CR>
```

---

## Essential Commands Summary

### Exiting VIM

| Command | Action |
|---------|--------|
| `:q` | Quit (fails if unsaved) |
| `:q!` | Quit without saving |
| `:wq` | Write and quit |
| `:x` | Write if changed and quit |
| `ZZ` | Write and quit (normal mode) |
| `ZQ` | Quit without saving (normal mode) |

**The Famous VIM Exit Problem:**
Many beginners struggle to exit VIM. Remember: `Esc` then `:q!`

### Getting Help

| Command | Action |
|---------|--------|
| `:help` | Open help |
| `:help command` | Help for command |
| `:help :command` | Help for Ex command |
| `:help i_ctrl-v` | Help for insert mode Ctrl+V |
| `:q` | Close help window |

---

## VIM Learning Resources

### Built-in Tutorial

```bash
vimtutor
```

**Covers:**
- Lessons 1-7 progressively teach VIM
- Takes 25-30 minutes
- Hands-on practice

### Cheat Sheets

- [vim.rtorr.com](https://vim.rtorr.com) - Comprehensive reference
- Quick lookup for commands
- Organized by category

### Practice Sites

- [VIM Adventures](https://vim-adventures.com/) - Game-based learning
- [VIM Genius](http://www.vimgenius.com/) - Flashcard practice

---

## Common VIM Idioms

### Efficient Editing Patterns

**Delete word and insert:**
```
ciw    " Change inner word
```

**Change inside quotes:**
```
ci"    " Change inside "
```

**Delete inside parentheses:**
```
di(    " Delete inside ()
```

**Visual select and indent:**
```
V5j>   " Select 6 lines and indent
```

**Copy line and paste below:**
```
yyp    " Duplicate line
```

**Delete to character:**
```
dfx    " Delete forward to 'x'
```

---

## Best Practices

### 1. Stay in Normal Mode

**Anti-pattern:** Using insert mode for navigation
**Good:** Use normal mode for everything except text input

### 2. Use Motions with Operators

**Anti-pattern:** `xxx` to delete 3 characters
**Good:** `3x` or `dw`

### 3. Use Dot Command

**Pattern:** Make a change, then repeat with `.`

### 4. Learn One Command at a Time

**Strategy:** Master basic commands, gradually add more

### 5. Customize Gradually

**Approach:** Start with vanilla VIM, add customizations as needs arise

---

## Summary

VIM is a powerful, modal text editor with a steep learning curve but exceptional efficiency once mastered:

**Key Concepts:**
- **Modal editing** - Different modes for different tasks
- **Composability** - Commands combine logically
- **Keyboard-centric** - Minimal mouse use

**Essential Modes:**
- **Normal** - Navigation and commands (`Esc`)
- **Insert** - Text entry (`i`, `a`, `o`)
- **Visual** - Selection (`v`, `V`, `Ctrl+v`)
- **Command** - Ex commands (`:`)

**Core Skills:**
- Navigation: `hjkl`, `w`, `b`, `gg`, `G`
- Editing: `d`, `y`, `c`, `p`
- Search: `/`, `?`, `n`, `N`
- Save/Exit: `:w`, `:q`, `:wq`

**Learning Path:**
1. Complete `vimtutor`
2. Practice basic navigation
3. Master operators and motions
4. Learn advanced features
5. Customize configuration

VIM's philosophy: **Minimize keystrokes, maximize efficiency.**
