# RHCSA Detailed Notes: Paths, Links, man, Vim, Variables

Source: instructor session transcript. Items marked **Correction** fix points where the session's explanation was inaccurate or garbled. Verify them in your own VM and with `man`.

---

## 1. Absolute vs relative paths

| | Absolute path | Relative path |
|---|---|---|
| Starts from | `/` (root directory) | Current working directory |
| Begins with | `/` | Anything except `/` |
| Works from anywhere | Yes | No, depends on where you are |
| Example | `/root/earth/asia/india` | `earth/asia/india` (from `/root`) |

Helper commands:

- `pwd` prints the present working directory.
- `ls` lists the current directory.
- `cd <path>` changes directory. `cd ..` goes up one level, `cd ../..` up two, `cd ../../..` up three.

**Exam tip:** if unsure where your shell is, use an absolute path.

---

## 2. File types

Check the first character of `ls -l` output.

| Char | Type |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |
| `b` | Block device |
| `c` | Character device |
| `s` | Socket |
| `p` | Named pipe |

The instructor grouped these as three kinds: regular, directory, and special (links, block, character, socket, pipe).

---

## 3. Links

A link gives a file more than one name.

```bash
ln source destination        # hard link
ln -s source destination     # soft (symbolic) link
ls -ilh                      # -i inode, -l long, -h human-readable
```

The **inode number** identifies the underlying file. Same inode means same file data.

### 3.1 Hard links

Demo: `ln /root/lnpanda HL1` (then `HL2`, `HL3`).

- All names share the **same inode**.
- Editing through one name changes the content seen through all names.
- Deleting one name leaves the others working.
- **Cannot link directories** (`ln moving HLfile` is refused).
- Cannot cross filesystems.

**Correction:** the instructor said hard links use more storage. They do not use extra data space, because the names share the same data and inode.

### 3.2 Soft links

Demo: `ln -s /root/lnpanda SL1` (then `SL2`, `SL3`).

- Has its **own inode**, so it is easy to tell from the target.
- A small file that stores the path to the target.
- **Can point to directories**.
- Independent of hard links: you can soft-link any original file.
- If you delete the original, the link **breaks** (dangling link).
- Can cross filesystems.

### 3.3 Comparison

| | Hard link | Soft link |
|---|---|---|
| Inode | Same as target | Own inode |
| Link to directory | No | Yes |
| After original deleted | Still works | Broken |
| Cross filesystems | No | Yes |

---

## 4. The `man` command

`man <command>` opens the manual page: name, synopsis, description, options, files, bugs, author.

Inside a man page:

| Key | Action |
|---|---|
| `/word` | Search forward |
| `n` / `N` | Next / previous match |
| `g` / `G` | Top / bottom of the page |
| `q` | Quit |
| `h` | Help for pager keys |

Keyword search: `man -k <keyword>` lists page names with one-line summaries.

**Corrections:**

- `man -k` shows matching page names and short descriptions, not the options. Open the page with `man <name>` for options.
- If `man -k` returns nothing on a fresh system, run `mandb` as root.

**Exam tip:** the manual pages are available in the exam. Practice finding options with `/` instead of memorizing them.

---

## 5. Vim

On RHEL, packages are RPMs managed by DNF (not APT). `vi` comes from `vim-minimal`, `vim` from `vim-enhanced`. They behave the same for daily use.

### 5.1 Modes

| Mode | Purpose | Enter with |
|---|---|---|
| Command (normal) | Cut, copy, paste, delete, move | Default on open; `Esc` returns here |
| Insert | Type text | `i`, `a`, `o` and similar |
| Visual | Select text | `v` chars, `V` lines, `Ctrl+V` block |
| Extended command (ex) | Save, quit, settings | `:` from command mode |

### 5.2 Movement (command mode)

- `h` left, `l` right, `j` down, `k` up
- `/word` search, `n` next match

### 5.3 Insert keys

| Key | Typing starts |
|---|---|
| `i` | At the cursor |
| `I` | Beginning of the line |
| `a` | After the cursor |
| `A` | End of the line |
| `o` | New line below |
| `O` | New line above |

**Correction:** the transcript mixed these up. This table is standard Vim behavior.

### 5.4 Editing (command mode)

- `x` delete a character
- `u` undo
- `yy` copy the current line
- `p` paste
- `dd` delete the current line
- In visual mode: select, then `y` to copy or `d` to delete

### 5.5 Settings, save, quit

| Command | Meaning |
|---|---|
| `:set nu` | Show line numbers |
| `:set nonu` | Hide line numbers |
| `:w` | Save |
| `:q` | Quit (fails if unsaved changes) |
| `:wq` | Save and quit |
| `:q!` | Quit without saving |
| `:wq!` | Force save and quit |
| `ZZ` | Save and quit (from command mode) |

### 5.6 Split windows

```bash
vim -o file1 file2    # stacked (horizontal split)
vim -O file1 file2    # side by side (vertical split)
```

**Correction:** the instructor said the reverse. `-o` stacks windows, `-O` places them side by side.

### 5.7 Encrypting a file

- In Vim, type `:X`, enter a password, and save.
- Reopen with `vim file` and enter the password to read it.
- To remove: `:X`, leave the password empty, save.
- This is light protection, not strong security.

### 5.8 Copy and paste in different environments

- Red Hat lab console: you can paste into it using its text panel, but you cannot copy out of it.
- PuTTY or PowerShell over SSH: selecting text copies, right-click pastes.

---

## 6. Variables

A variable stores a value under a name.

```bash
name="Laxmi"        # no spaces around =
echo $name
```

### 6.1 Lifetime

A variable set at the prompt is lost when the shell closes or the system reboots.

### 6.2 Persist for one user

1. Add `name="Deepika"` to `~/.bashrc`.
2. Reload: `source ~/.bashrc` (or `. ~/.bashrc`).
3. It survives new logins and reboots.

If you edit `~/.bashrc` but don't `source` it, your current shell keeps the old value.

### 6.3 Environment variables

```bash
export name=value
env | grep name
```

`export` passes the variable to child processes. It is not permanent by itself, so put the `export` line in `~/.bashrc` to keep it.

### 6.4 Global for all users

Demo: add `name=Mobin` to `/etc/environment` (no spaces around `=`), then check as other users.

**Correction and caution:**

- `/etc/environment` is read at login, so users see the value after logging in again.
- A common alternative is a script in `/etc/profile.d/`, for example `/etc/profile.d/myvars.sh` containing `export name=value`.

---

## 7. VM tip from the session start

Create the VM with a **20 GB disk**. An install on a roughly 8.8 GB disk failed at the installation destination step. Delete the VM and recreate it rather than trying to add space.

---

## 8. Next session

`alias` and pipelines (`|`).