\# Phase 4: Process Management



\## Objective

Learn to inspect, prioritize, and control both individual processes

and systemd-managed services — the core toolkit for diagnosing

performance issues and managing what's running on a Linux server.



\## Steps Taken



\### 1. Process Inspection

\- Viewed all running processes with `ps aux` (all users, user-oriented

&#x20; format, including processes without a terminal)

\- Used `top` for a live, auto-refreshing view of CPU/memory usage,

&#x20; load average, and process states

\- Searched for specific processes by name using `pgrep`, cleaner than

&#x20; piping `ps aux` through `grep`



\### 2. Process Priority Control

\- Started a disposable background process with `sleep 300 \&`

\- Changed its scheduling priority using `renice 10 -p <PID>`

\- Verified the change with `ps -o pid,ni,cmd -p <PID>`

\- Terminated it gracefully with `kill <PID>` (default SIGTERM signal)



\### 3. Service Management (systemctl)

\- Checked service status with `systemctl status sshd`

\- Practiced stop/start on `crond`, confirming state changes each time

&#x20; with `systemctl status`

\- Checked boot-persistence separately from runtime state using

&#x20; `systemctl is-enabled crond`



\### 4. Service Logs (journalctl)

\- Viewed logs for a specific unit with `journalctl -u sshd`

\- Limited output to the most recent entries with `journalctl -u sshd -n 20`

\- Retrieved the earliest entries instead by piping through `head`:

&#x20; `journalctl -u sshd | head -n 20`, since journalctl's `-n` flag only

&#x20; supports "most recent N," not "first N"



\## Why This Matters

Every production server needs the ability to diagnose what's consuming

resources, adjust priority when one task shouldn't compete with

critical services, and cleanly stop misbehaving processes without

corrupting data (graceful SIGTERM before forceful SIGKILL). Just as

important is understanding that `start` and `enable` are independent —

a service running today but not enabled will silently vanish after

the next reboot, a very common and avoidable production mistake.



\## Notes / Issues Encountered



\*\*Issue 1 — Misplaced background operator:\*\*

Ran `sleep \& 300` instead of `sleep 300 \&`. Bash interpreted this as

running `sleep` with no argument in the background, then tried (and

failed) to run `300` as a separate command. Lesson: `\&` must always

go at the very end of the full command, after all arguments.



\*\*Issue 2 — journalctl has no "first N lines" flag:\*\*

Assumed `journalctl` would support head-like behavior directly.

In reality, `-n` in journalctl always means "most recent N entries,"

mirroring `tail`, not `head`. To get the earliest entries instead,

the full output must be piped into the standalone `head` command.

