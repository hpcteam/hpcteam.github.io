---
title: Vim Editor Basics for Beginners
date: 2026-08-07
authors:
  - ayyappa
categories:
  - Linux
tags:
  - vim
  - vi
  - text-editor
  - command-line
---

# Vim Editor Basics for Beginners

Today I learned the basics of the **Vim (or vi) editor** — how it works, its different modes, and the shortcuts that make editing files from the terminal much faster. Here's the clean, simple version.

## Opening a File

```bash
vim filename
```

This opens the file inside Vim. If the file doesn't exist yet, Vim will create it once you save.

## The Status Bar

At the bottom of the screen, Vim shows useful details:

- The **file name**
- How many **lines** are in the file
- Where your **cursor** is currently located
- How much of the file you're currently viewing (as a percentage)

## The Modes in Vim

Vim works using different **modes**. This is what makes it different from simple editors — you switch modes depending on whether you want to move around, type text, or run commands.

### 1. Normal Mode (also called Command Mode)

This is the mode you start in when you open a file. In this mode, your keystrokes are treated as **commands**, not text. For example, pressing `dd` deletes a line instead of typing the letters "dd."

### 2. Insert Mode

This is where you actually type and edit text, like a normal text editor.

- Press `i` → start typing right where your cursor is.
- Press `ESC` → go back to Normal mode (needed before you can save or run other commands).

### 3. Command-Line Mode (Extended Commands)

Press `:` while in Normal mode to enter this mode. Here you can run commands like saving, quitting, searching and replacing, or turning on line numbers.

Examples: `:wq`, `:q`, `:set nu`

### 4. Visual Mode

Used to **select/highlight text**, similar to how you'd click-and-drag to select text in a regular editor, but using the keyboard.

- Press `Shift + v` → highlights whole lines. Use the up/down arrow keys to extend the selection.

## Moving Around the File

| Shortcut | What it does |
|----------|----------------|
| `Shift + g` | Go to the **bottom** of the file |
| `gg` | Go to the **top** of the file |
| `/keyword` | **Search** for a word in the file |
| `:23` | Jump to a specific **line number** (e.g. line 23) |

## Deleting and Copying Text

| Shortcut | What it does |
|----------|----------------|
| `dd` | Delete the current line |
| `5dd` | Delete 5 lines starting from the current line (replace 5 with any number) |
| `yy` | Copy (yank) the current line |
| `5yy` | Copy 5 lines |
| `p` | Paste the copied/deleted text after the cursor |
| `3p` | Paste the copied text 3 times |
| `dw` | Delete a single word where the cursor is |
| `dG` (press `d` then `Shift+g`) | Delete everything from the cursor **to the end** of the file |
| `dgg` (press `d` then `gg`) | Delete everything from the cursor **to the beginning** of the file |
| `yG` (press `y` then `Shift+g`) | Copy everything from the cursor **to the end** of the file |
| `ygg` (press `y` then `gg`) | Copy everything from the cursor **to the beginning** of the file |

## Undo and Redo

- `u` → Undo the last change (make sure you're in Normal mode — press `ESC` first)
- `Ctrl + r` → Redo the change you just undid

## Insert Mode Shortcuts (Ways to Start Typing)

| Shortcut | What it does |
|----------|----------------|
| `i` | Start typing right where the cursor is |
| `Shift + i` | Move to the **beginning** of the line and start typing |
| `Shift + a` | Move to the **end** of the line and start typing |
| `a` | Move one character to the right of the cursor and start typing(Cursor move one character and insert next line ) |
| `o` | Create a **new empty line below** the cursor and start typing |
| `Shift + o` | Create a **new empty line above** the cursor and start typing |

## Find and Replace

```
:%s/oldword/newword/g
```

This finds every occurrence of `oldword` in the file and replaces it with `newword`.

- `%` means "apply to the whole file" (not just one line)
- `g` means "replace all matches in each line," not just the first one

## Saving and Quitting

| Command | What it does |
|---------|----------------|
| `:wq` | Save the file and quit |
| `:q` | Quit (only works if there are no unsaved changes) |
| `ZZ` (Shift + z, twice) | Save and quit — a shortcut for `:wq` |
| `:x` | Also saves and quits, similar to `:wq` |

## Line Numbers

- `:set nu` → Turn **on** line numbers
- `:set nonu` → Turn **off** line numbers

## Working with Multiple Files

- `vim -O file1 file2 file3` → Open multiple files at once, side by side (vertical split)
- `:vsplit filename` → Open another file in a vertical split while already inside Vim
- `Ctrl + w`, then `w` → Switch between the open file windows/splits
- `:wqa` → Save and quit **all** open files at once

## Password Protecting a File in Vim

- `:X` → Set a password for the file (you'll be asked to type and confirm it)
- To **remove** the password: open the file, type `:X`, then just press **Enter twice** without typing anything — this clears the password.

## Quick Recap

> Vim has different modes for different jobs: **Normal mode** to move around and run commands, **Insert mode** to type text, **Visual mode** to select text, and **Command-line mode** to save, quit, search, and replace. Once you get comfortable switching between these with `i`, `ESC`, and `:`, editing files becomes very fast — no mouse required.
