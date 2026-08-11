---
title: Bash Basics - touch, Variables, export, and File Viewers
date: 2026-08-07
authors:
  - ayyappa
categories:
  - Linux
tags:
  - bash
  - variables
---

# Bash Basics: touch, Variables, export, and File Viewers

Today I learned a mix of small but very useful Linux/Bash basics — creating multiple files at once, working with variables, and the difference between two common file-viewing commands.

## Creating Files with `touch`

The `touch` command creates empty files (or updates the timestamp if the file already exists).

### Creating files one by one

```bash
touch preven1 preven2 preven3
```

This creates three separate files: `preven1`, `preven2`, and `preven3`.

### Creating a series of files using brace expansion

Instead of typing each file name manually, you can use `{ }` (curly braces) to generate a range automatically:

```bash
touch preven{1..5}
```

This creates 5 files at once: `preven1`, `preven2`, `preven3`, `preven4`, `preven5`.

You can even combine two ranges together:

```bash
touch preven{a..d}{1..5}
```

This creates every combination of letters `a` to `d` with numbers `1` to `5` — so `prevena1`, `prevena2` ... all the way to `prevend5` (20 files in total). This is called **brace expansion**, and it saves a lot of typing.

## Variables in Bash

A variable is just a name that holds a value, so you can reuse it instead of typing the value again and again.

### Creating a variable

```bash
course="this is redhat course"
```

Important: there should be **no spaces** around the `=` sign — `course = "..."` will cause an error.

### Using (printing) a variable

To use or print a variable's value, put a `$` symbol in front of its name:

```bash
echo $course
```

This prints: `this is redhat course`

### Making a variable permanent

Normally, a variable only exists for your current terminal session — once you close the terminal, it's gone. To make a variable available every time you open a new terminal, add it to your `.bashrc` file (found in your home directory).

### Exporting a variable

By default, a variable you create is only available in your current shell. If you want other programs or sub-processes (like a script you run) to also be able to see and use that variable, you need to **export** it:

```bash
export name=ayyappa
echo $name
```

This will print: `ayyappa`

The `export` command turns a normal shell variable into an **environment variable**, meaning any process started from that shell can access it too.

## Viewing Files: `less` vs `more`

Both commands let you view the contents of a file one screen at a time, instead of dumping the whole file on your terminal at once. But there are some differences:

| Command | Behavior |
|---------|----------|
| `more` | Shows the file a page (screen) at a time, but you can only scroll **forward** |
| `less` | Shows the file a page at a time too, but you can scroll **both forward and backward**, and search through the file more easily |

In both, you can press **Space** to move a full page forward, or **Enter** to move down one line at a time. The key advantage of `less` is the flexibility to go backward and jump around — which is why most people prefer `less` today, even though `more` came first.

## What is `.vimrc`?

`.vimrc` is a **configuration file for the Vim text editor**. It lives in your home directory (`~/.vimrc`) and lets you customize how Vim behaves every time you open it — for example, automatically turning on line numbers, changing colors, setting tab spacing, or adding shortcuts — without needing to type those settings manually every single time.

## Quick Recap

> `touch` with brace expansion (`{1..5}`) lets you create many files in one command. Variables store values you can reuse with `$variablename`, and `export` makes a variable visible to other programs, not just your current shell. `less` and `more` both page through a file's contents, but `less` gives you more control by allowing backward scrolling and search. And `.vimrc` is where you save your personal Vim settings so they apply automatically every time you open the editor.
