# Linux Processes Explained: Process States, Signals, and Process Management

Understanding **processes** is one of the most important concepts for a Linux System Administrator. Every application, command, service, or program running on a Linux server is associated with one or more processes.

In this guide, we will understand:

* What is a process?
* How a process is created
* Parent and child processes
* `systemd` and the Linux process hierarchy
* Process IDs (PID)
* Process states
* Zombie processes
* `ps` and `top`
* Linux signals
* `kill`, `killall`, `pkill`, and `pgrep`
* `w`, `who`, and `uptime`
* Practical examples

---

## 1. What Is a Process?

A **process is a running instance of a program**.

For example, when you execute:

```bash
firefox
```

or:

```bash
vim test.txt
```

Linux loads the required program into memory and creates a process.

A simple way to understand it is:

```text
Program
   ↓
Execute the program
   ↓
Linux creates a process
   ↓
Process receives resources
   ↓
Process executes
```

### Program vs Process

A **program** is a file containing executable code.

A **process** is that program while it is running.

For example:

```text
/usr/bin/sleep       → Program
sleep 1000           → Process
```

If you run:

```bash
sleep 1000 &
```

Linux creates a running process for the `sleep` program.

---

# 2. What Does a Process Need?

When Linux creates a process, the process requires several resources to execute properly.

Some important resources include:

* CPU
* Memory
* File descriptors
* Security credentials
* Environment variables
* One or more execution threads
* Access to files and devices

For example, if a program requires memory and the system cannot provide the required resources, the process may fail to start or may be terminated later due to resource pressure.

---

# 3. PID – Process ID

Every running process in Linux has a unique **Process ID (PID)**.

You can see process IDs using:

```bash
ps -ef
```

Example:

```text
root       1023       1  0 10:10 ?        00:00:00 sshd
user       2456    1023  0 10:12 ?        00:00:00 sshd
```

Here:

```text
PID = 1023
PID = 2456
```

The PID is used to identify and manage a process.

For example:

```bash
kill 2456
```

sends a signal to process `2456`.

---

# 4. The First Process in RHEL

On modern RHEL systems, the first userspace process is normally:

```text
systemd
```

It has:

```text
PID = 1
```

You can verify it with:

```bash
ps -p 1 -f
```

Example:

```text
UID   PID  PPID  CMD
root    1     0  /usr/lib/systemd/systemd
```

`systemd` is responsible for managing many system services and processes during system startup and operation.

---

# 5. Parent and Child Processes

Linux processes can create other processes.

The process that creates another process is called the **parent process**.

The newly created process is called the **child process**.

Example:

```text
Parent Process
      |
      +---- Child Process
```

A process can also create multiple children:

```text
Parent
 ├── Child 1
 ├── Child 2
 └── Child 3
```

You can see the parent process ID using:

```bash
ps -ef
```

The output contains both:

```text
PID
PPID
```

Where:

* `PID` = Process ID
* `PPID` = Parent Process ID

---

# 6. How Is a New Process Created?

Linux commonly uses the `fork()` system call to create a new process.

Conceptually:

```text
Parent Process
      |
      | fork()
      ↓
Child Process
      |
      ↓
exec()
      |
      ↓
New Program
```

The child initially inherits various attributes/resources from the parent and then may use `exec()` to replace its process image with another program.

For example, when a shell launches a command:

```bash
ls
```

the shell creates a child process and the child executes the `ls` program.

A simplified view is:

```text
bash
 |
 +---- ls
```

---

# 7. Process Creation Example

Run:

```bash
sleep 1000 &
```

Then check:

```bash
ps -ef | grep sleep
```

You may see:

```text
user   3210  2500  0 10:20 pts/0  00:00:00 sleep 1000
```

Here:

```text
PID  = 3210
PPID = 2500
```

The shell is the parent and `sleep` is the child.

You can also use:

```bash
pstree
```

to visualize the process hierarchy.

Example:

```text
systemd
 ├── sshd
 │    └── bash
 │         └── sleep
 ├── NetworkManager
 └── cron
```

---

# 8. Process Lifecycle

A simplified process lifecycle looks like:

```text
fork()
   ↓
New Process
   ↓
Runnable
   ↓
Running
   ↓
Waiting/Sleeping
   ↓
Runnable
   ↓
Running
   ↓
Terminated
```

A process does not continuously execute on the CPU.

The Linux scheduler decides when a runnable process gets CPU time.

---

# 9. Linux Process States

You can see process states using:

```bash
ps -eo pid,ppid,state,cmd
```

Common process states include:

| State | Meaning               |
| ----- | --------------------- |
| `R`   | Running or runnable   |
| `S`   | Interruptible sleep   |
| `D`   | Uninterruptible sleep |
| `T`   | Stopped               |
| `Z`   | Zombie                |
| `I`   | Idle kernel thread    |

Let's understand them one by one.

---

# 10. R – Running / Runnable

`R` means the process is either:

* Currently executing on a CPU, or
* Ready to run and waiting for CPU time.

Example:

```bash
ps -eo pid,state,cmd
```

Output:

```text
PID   S   CMD
2456  R   ./application
```

The process is actively running or ready to run.

---

# 11. S – Interruptible Sleep

`S` means:

```text
TASK_INTERRUPTIBLE
```

The process is sleeping and waiting for an event.

For example, a process may wait for:

* Input
* Network data
* A timer
* A file operation
* Another event

Processes in this state can normally be awakened by the required event or by appropriate signals.

Example:

```text
Process
   ↓
Waiting for input
   ↓
S state
   ↓
Event occurs
   ↓
Runnable
```

---

# 12. D – Uninterruptible Sleep

`D` means:

```text
TASK_UNINTERRUPTIBLE
```

The process is usually waiting for a kernel-level operation, commonly involving I/O.

For example:

```text
Process
   ↓
Waiting for I/O
   ↓
D state
```

A process stuck in `D` state may not respond to normal signals immediately.

This state is particularly important for System Administrators because a large number of processes stuck in `D` state can indicate problems with storage, NFS, disks, drivers, or other I/O operations.

You can check for `D` state processes using:

```bash
ps -eo pid,state,cmd | awk '$2=="D"'
```

---

# 13. K – Killable Sleep

`K` represents a killable sleep state in Linux kernel terminology.

It is similar to uninterruptible sleep but allows certain fatal signals to wake/terminate the task.

This state is more commonly encountered when examining detailed kernel task states than during normal day-to-day process administration.

---

# 14. I – Idle

`I` generally represents an idle kernel thread.

It is mainly associated with kernel threads rather than normal user applications.

For example:

```bash
ps -e -o pid,state,comm
```

may show processes with state `I`.

---

# 15. T – Stopped

`T` means the process has been stopped.

For example:

```bash
sleep 500
```

Press:

```text
Ctrl + Z
```

The shell stops the foreground process.

You can check it using:

```bash
jobs
```

Example:

```text
[1]+  Stopped  sleep 500
```

You can resume it in the foreground using:

```bash
fg
```

or in the background using:

```bash
bg
```

---

# 16. Z – Zombie Process

A **zombie process** is a process that has finished execution but still has an entry in the process table because its parent has not yet collected its exit status.

A simplified example:

```text
Parent
  |
  +---- Child
          |
          ↓
      Finished
          |
          ↓
       Zombie
```

You can find zombie processes using:

```bash
ps -eo pid,ppid,state,cmd | awk '$3=="Z"'
```

Example:

```text
PID   PPID  S  CMD
4210  3000  Z  [test] <defunct>
```

`<defunct>` is commonly displayed for zombie processes.

### Important

A zombie is **not a process actively consuming CPU**.

However, a large number of zombies can indicate that a parent process is not properly handling child process termination.

---

# 17. Viewing Processes with ps

The `ps` command provides a snapshot of running processes.

### Basic command

```bash
ps
```

### Show all processes

```bash
ps -ef
```

Example:

```text
UID    PID   PPID  CMD
root     1      0  /usr/lib/systemd/systemd
root  1023      1  /usr/sbin/sshd
user  2500   1023  -bash
```

Important columns:

```text
PID   → Process ID
PPID  → Parent Process ID
CMD   → Command
```

You can also use:

```bash
ps aux
```

This provides information such as:

* CPU usage
* Memory usage
* PID
* Process owner
* Start time
* Command

---

# 18. top – Real-Time Process Monitoring

Unlike `ps`, which provides a snapshot, `top` provides a continuously updating view.

Run:

```bash
top
```

You can monitor:

* CPU usage
* Memory usage
* Load average
* Running processes
* Process states
* Individual process resource consumption

For a Linux System Administrator, `top` is one of the most useful commands for quickly identifying resource-intensive processes.

---

# 19. Understanding Linux Signals

Linux uses **signals** to communicate with processes.

A signal is essentially a notification sent to a process.

You can view available signals using:

```bash
kill -l
```

Common signals include:

| Signal    | Number | Meaning                      |
| --------- | -----: | ---------------------------- |
| `SIGHUP`  |      1 | Hangup                       |
| `SIGINT`  |      2 | Interrupt                    |
| `SIGTERM` |     15 | Request graceful termination |
| `SIGKILL` |      9 | Force termination            |
| `SIGSTOP` |     19 | Stop process                 |
| `SIGCONT` |     18 | Continue process             |

---

# 20. kill Command

Despite its name, the `kill` command does not necessarily terminate a process immediately.

It sends a signal to a process.

Syntax:

```bash
kill <PID>
```

For example:

```bash
kill 2456
```

By default, this sends:

```text
SIGTERM (15)
```

---

# 21. SIGTERM – Graceful Termination

You can explicitly send SIGTERM:

```bash
kill -15 2456
```

or:

```bash
kill -TERM 2456
```

SIGTERM asks the process to terminate gracefully.

This gives the application an opportunity to:

* Close files
* Save data
* Clean up resources
* Exit properly

For production systems, graceful termination should generally be attempted before forceful termination.

---

# 22. SIGKILL – Forceful Termination

You can send:

```bash
kill -9 2456
```

This sends:

```text
SIGKILL
```

SIGKILL cannot be caught or ignored by the process.

It immediately terminates the process from the kernel's perspective.

### Important

Do not make `kill -9` your first option.

A better general approach is:

```bash
kill -15 <PID>
```

Wait and check whether the process exits.

If the process does not terminate and there is a valid reason to force it:

```bash
kill -9 <PID>
```

Also remember that a process stuck in `D` state may not disappear immediately even after SIGKILL because it is waiting in an uninterruptible kernel operation.

---

# 23. Finding the PID of a Process

You can use:

```bash
pidof firefox
```

Example:

```text
1234 1256
```

This means multiple Firefox-related processes may be running.

You can also use:

```bash
pgrep firefox
```

---

# 24. pgrep Command

`pgrep` searches for processes based on their name or other attributes.

Example:

```bash
pgrep sshd
```

Output:

```text
1023
2045
```

You can also display the process name:

```bash
pgrep -a sshd
```

Example:

```text
1023 /usr/sbin/sshd -D
2045 sshd: user@pts/0
```

### Why use pgrep?

Instead of:

```bash
ps -ef | grep sshd
```

you can often simply use:

```bash
pgrep -a sshd
```

---

# 25. killall Command

`killall` can send a signal to processes based on their process name.

For example:

```bash
killall firefox
```

This targets processes named `firefox`.

You can specify a signal:

```bash
killall -15 firefox
```

or:

```bash
killall -9 firefox
```

### Be Careful

Because `killall` operates by process name, make sure you understand which processes will be affected before executing it on a production server.

---

# 26. pkill Command

`pkill` is another powerful command for sending signals to processes based on matching criteria.

For example:

```bash
pkill firefox
```

You can also target processes belonging to a specific user.

For example:

```bash
pkill -U alice
```

This sends the default termination signal to processes owned by user `alice`.

This can be useful for administrative tasks, but it should be used carefully because it can terminate multiple processes at once.

---

# 27. kill vs killall vs pkill

| Command   | Works With                                    |
| --------- | --------------------------------------------- |
| `kill`    | PID                                           |
| `killall` | Process name                                  |
| `pkill`   | Process name / user / other matching criteria |
| `pgrep`   | Finds matching PIDs                           |

Example:

```bash
kill 1234
```

Targets:

```text
PID 1234
```

Whereas:

```bash
killall nginx
```

targets processes by name.

And:

```bash
pkill -U alice
```

targets processes owned by a user.

---

# 28. Checking Logged-In Users

The `w` command shows currently logged-in users and system activity.

Run:

```bash
w
```

Example:

```text
10:30:15 up 12 days,  4:20,  3 users,  load average: 0.20, 0.15, 0.10

USER     TTY      FROM       LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    10.10.1.20 09:10   1:20   0.10s  0.05s top
admin    pts/1    10.10.1.30 10:00   5.00s  0.02s  0.02s bash
```

It provides information about:

* Current time
* System uptime
* Number of logged-in users
* Load average
* Logged-in users
* Their current activity

---

# 29. who Command

The `who` command shows who is currently logged into the system.

Run:

```bash
who
```

Example:

```text
root     pts/0   2026-09-22 09:10
admin    pts/1   2026-09-22 10:00
```

It is useful when you want to quickly identify active user sessions.

---

# 30. uptime Command

The `uptime` command displays:

* Current time
* System uptime
* Number of logged-in users
* Load average

Run:

```bash
uptime
```

Example:

```text
10:30:15 up 12 days, 4:20, 3 users, load average: 0.20, 0.15, 0.10
```

The three load-average values represent approximately:

```text
1 minute    5 minutes    15 minutes
```

Load average should be interpreted relative to the number of CPU cores and the type of workload. A load average of `8` means something very different on a 2-core system than on a 32-core system.

---

# 31. Practical Troubleshooting Example

Imagine a user reports:

> "The application is not responding."

As a System Administr
