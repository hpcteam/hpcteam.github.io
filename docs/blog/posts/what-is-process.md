---
title: "Linux Process Management: States, fork(), and Killing Processes Explained"
date: 2026-09-22
authors:
  - ayyappa
categories:
  - Linux
tags:
  - processes
  - systemd
  - fork
  - zombie-process
  - kill
  - ps
  - top
---

# Linux Process Management 

This post covers how Linux processes are created and managed — what a process actually is, how `fork()` creates child processes, the full lifecycle of process states, how to view running processes, and how to safely terminate them with `kill`, `killall`, and `pkill`.

## 1. What Is a Process?

A **process** is a running instance of a launched, executable program — essentially, a piece of code that is actively executing on your server.

When you start an application, the system first checks whether the required **resources** are available before it will even begin. A process needs:

- **Memory**
- **Security properties** (permissions, user/group context)
- **One or more execution threads** to run the program's code

If the required resources aren't available, the process simply won't start.

### `systemd` — The Ancestor of Every Process

The very first process started on a RHEL system is **`systemd`** (PID 1). Every other process on the system is, directly or indirectly, a **child process of `systemd`** — any process can go on to create its own child processes, but tracing that chain back far enough always leads to `systemd`.

```bash
ps -ef | head -1
# UID   PID  PPID  ...  CMD
# root    1     0  ...  /usr/lib/systemd/systemd
```

## 2. How a Process Is Created:

Before a parent process starts a child process, it performs an operation called **`fork()`**.

**`fork()`** means the parent process **duplicates its own resources** and hands that duplicate to a newly created child process.

Here's the sequence:

1. The parent process calls `fork()`, duplicating its resources for the child.
2. The new child process is assigned a unique **Process ID (PID)**.
3. The child process gathers whatever additional resources it needs, then begins executing its own code.
4. While the child runs, the **parent process goes into a sleep state**, waiting.
5. Once the child finishes executing, it must **release the resources** it acquired from the parent.
6. The child then sends a signal — commonly referred to here as the **`wait`** signal — back to the parent, effectively saying: *"My work is done, you can reuse these resources to create another child process."*
7. The parent wakes up and is free to fork another child process.

### Zombie Processes

Sometimes a child process **finishes executing but doesn't release the resources** it acquired from the parent, and its exit status is never collected. This leftover, "not-quite-cleaned-up" process is called a **zombie process** — it shows up in the process table (marked `Z`) but is no longer doing any actual work.

```bash
ps aux | awk '$8=="Z" {print}'
```
This lists any processes currently stuck in the zombie state.

## 3. The Process Lifecycle (Process States)

A process moves through a defined lifecycle:

```
fork() → new → scheduled → runnable (ready) → running → sleeping/waiting → (back to runnable, or terminated)
```

- **New / Scheduled:** after `fork()`, the process is created and handed to the scheduler.
- **Runnable (Ready):** the process is ready to run. In this state, it can receive signals like **stop**, **resume**, or be sent to **sleep**.
- **Running:** the process is actually executing on the CPU. From here, it can receive a **wait** (sleep) signal.
- **Important rule:** once a process goes into a wait/sleep state, it **cannot** go directly back to running — it must first re-enter the runnable (ready) queue and be scheduled again.

### Process State Codes

| Code | State | Meaning |
|---|---|---|
| `R` | Running / Runnable | The process is either actively executing on the CPU, or ready and waiting for CPU time. |
| `S` | Sleeping — `TASK_INTERRUPTIBLE` | The process is waiting (for I/O, a resource, a condition, etc.) but **can** respond to signals — it can be brought back to a normal running state on request. |
| `D` | Sleeping — `TASK_UNINTERRUPTIBLE` | The process is also sleeping, but unlike `S`, it does **not** respond to signals. This usually happens during low-level device I/O, where interrupting it mid-operation could leave the device in an unpredictable state. |
| `K` | `TASK_KILLABLE` | Similar to `D` (uninterruptible), but it **will** respond specifically to a `KILL` signal. |
| `I` | `TASK_REPORT_IDLE` | The process only responds to **fatal** signals sent forcefully — a general signal or a normal kill request won't wake it. |
| `T` | Stopped | The process has been stopped (e.g., via a stop signal), and execution is paused. |
| `Z` | Zombie | The process has finished executing but its resources/exit status haven't been cleaned up yet. |

**Example — watch a process's state live:**

```bash
ps -o pid,stat,cmd -C firefox
```
The `STAT` column shows the current code (`R`, `S`, `D`, `T`, `Z`, etc.) for that process.

## 4. Viewing Processes: `ps` and `top`

| Command | What it shows |
|---|---|
| `ps -ef` | A **static** (point-in-time) snapshot of the process table, in full-format listing |
| `ps -aux` | A **static** snapshot of the process table, BSD-style, showing all users' processes |
| `top` | A **dynamic**, continuously refreshing, real-time view of the process table |

**Example:**

```bash
ps -ef | grep firefox
top
```
`ps` gives you a one-time list; `top` keeps updating live so you can watch CPU/memory usage change in real time.

## 5. Killing Processes

### `kill` — Send a Signal to a Specific PID

`kill` sends a **signal** to a process. Linux defines **64 signals** in total, each meaning something different — not all of them terminate a process (some pause it, some tell it to reload configuration, etc.), but the two most commonly used to stop one are `-9` and `-15`.

**Step 1 — find the process ID:**

```bash
pidof firefox
```

**Step 2 — send a signal using that PID (or, in many shells, the process name directly):**

```bash
kill -15 firefox     # SIGTERM: politely ask the process to terminate, allowing cleanup
kill -9 firefox       # SIGKILL: forcefully terminate immediately, no cleanup
```

**Best practice:** always try `kill -15` (graceful termination) **first**, and only fall back to `kill -9` (forceful termination) if the process doesn't respond — this gives the process a chance to close files, release resources, and shut down cleanly before being force-killed.

### `killall` — Kill by Process Name

`killall` kills **all processes that share the same name** — useful when multiple instances of the same program are running.

```bash
killall firefox
```
This terminates every running `firefox` process at once, instead of having to `kill` each PID individually.

### `pkill` — Kill by Pattern or Owner

`pkill` matches processes by name pattern **or** by other attributes, such as the user who owns them.

```bash
pkill -U apple
```
This kills **all processes owned by the user `apple`**.

### `pgrep` — Find Matching Processes (Without Killing Them)

`pgrep` is the "read-only" counterpart to `pkill` — instead of killing matching processes, it just **lists their PIDs**, so you can inspect what would be affected before actually killing anything.

```bash
pgrep firefox
```
Returns the PID(s) of any running `firefox` processes.

```bash
pgrep -U apple
```
Returns the PIDs of every process owned by user `apple` — the same matching logic as `pkill -U apple`, but without terminating them. This makes `pgrep` the safer way to double-check exactly what a `pkill` command would target, before you run it for real.

## 6. Checking System Load and Logged-In Users

| Command | What it shows |
|---|---|
| `w` | System load average, plus which users are currently connected and what they're running |
| `uptime` | How long the system has been running (system uptime), along with a quick load average |
| `who` | Which users are logged in, and when/where they connected from |

**Example:**

```bash
w
uptime
who
```
