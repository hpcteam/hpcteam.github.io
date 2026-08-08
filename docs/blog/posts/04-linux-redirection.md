---
title: Understanding Redirection in Linux
date: 2026-08-07
authors:
  - ayyappa
categories:
  - Linux
tags:
  - redirection
  - shell
  - stdin
  - stdout
  - stderr
---

# Understanding Redirection in Linux

Today I learned about **redirection** in Linux — how to control where a command's output (or errors) actually go, instead of just letting them print to the screen. Here's the simple explanation.

## The Theory: Every Command Has 3 Data Streams

Whenever you run a command in Linux, it doesn't just "do something" — it also communicates using three standard **data streams**. Think of these as three separate channels a command uses to talk to you.

| Stream | Name | Code | What it means |
|--------|------|------|----------------|
| stdin | Standard Input | `0` | Data going **into** the command (like keyboard input) |
| stdout | Standard Output | `1` | The **normal/successful result** of the command |
| stderr | Standard Error | `2` | Any **error messages** the command produces |

By default:
- `stdin` comes from your keyboard
- `stdout` and `stderr` both print to your terminal screen

**Redirection** means telling Linux: "Instead of sending this stream to the screen, send it somewhere else — usually into a file."

## Why Does This Matter?

Normally, when a command runs, both its successful output *and* its errors show up mixed together on your screen. That's fine for small commands, but it becomes messy when:

- You're running a script and want to keep a clean log of only the successful output
- You want to save only the errors separately, to review or debug later
- You're automating tasks and don't want anything printed to the screen at all

Redirection lets you separate and control exactly where each type of output goes.

## The Redirection Symbols

| Symbol | What it does |
|--------|----------------|
| `>` | Redirect **stdout** (output) to a file — overwrites the file |
| `2>` | Redirect **stderr** (errors) to a file |
| `>>` | Same as `>` but **appends** to the file instead of overwriting it |
| `&>` | Redirect **both stdout and stderr** to the same file |
| `2>&1` | Redirect stderr into wherever stdout is currently going |

Notice: for **output**, you don't need to mention the code (`1`) — it's the default. But for **errors**, you must mention `2`, otherwise Linux won't know you mean stderr.

## Examples

### 1. Redirect only the output (stdout) to a file

```bash
firewall-cmd --list-ports > output.txt
```

If the command succeeds, the result gets saved into `output.txt` instead of printing on screen.

### 2. Redirect only the errors (stderr) to a file

```bash
firewall-cmd --list-portss 2> error.txt
```

If the command fails or produces a warning, that error message goes into `error.txt` — but any normal output would still print on the screen.

### 3. Redirect output and errors to two separate files

```bash
firewall-cmd --list-ports > output.txt 2> error.txt
```

This keeps things clean and separated — successful results in one file, problems in another. This is very useful for troubleshooting.

### 4. Redirect both output and errors into a single file

```bash
lscpu &> output_error.txt
```

Here, everything the command produces — whether it's a success message or an error — goes into the same file.

## Quick Recap

> Every Linux command produces up to three types of data: input (`0`), output (`1`), and errors (`2`). Using `>`, `2>`, `>>`, and `&>`, you can control exactly where each of these goes — whether that's a single file, separate files, or somewhere you can review later. This is one of the most useful basics for scripting, automation, and troubleshooting on Linux.
