\# Phase 10: Logs \& Troubleshooting



\## Objective

Build practical log-investigation skills across the systemd journal and

traditional flat-file logs, including making the journal persistent

across reboots, filtering by priority and time, and understanding log

rotation.



\## Steps Taken



\### 1. The systemd Journal

\- Used `journalctl -b` to filter logs to the current boot session only

\- Discovered the journal was volatile (memory-only, wiped on reboot) —

&#x20; `/var/log/journal` didn't exist, meaning `/run/log/journal` was the

&#x20; only backing store

\- Made the journal persistent: created `/var/log/journal` and restarted

&#x20; `systemd-journald`, which automatically switched logging to disk

\- Confirmed persistence with `journalctl --disk-usage` (non-zero size)

&#x20; and, after a reboot, `journalctl --list-boots` showing two boot

&#x20; entries instead of one — proof logs survived the reboot



\### 2. Filtering the Journal

\- Filtered by severity with `journalctl -p <level>` (e.g. `err` and

&#x20; above) to cut noise and surface only real problems

\- Filtered by time window with `journalctl --since "..." --until "..."`

\- Combined unit + priority filters (`journalctl -u sshd -p err`) to

&#x20; narrow logs to a specific service's errors only



\### 3. Flat-File Logs and Rotation

\- Reviewed `/var/log` contents: `secure` (authentication/SSH),

&#x20; `messages` (general system log), `cron` (from Phase 9), among others

\- Attempted to locate an earlier SSH root-login-denial event (from

&#x20; Phase 8's SSH hardening test) in `/var/log/secure`, but the entry

&#x20; had already rotated out of the active log file

\- Discovered the rotated archive `/var/log/secure-<date>` and searched

&#x20; it directly, confirming that log rotation limits how far back a flat

&#x20; file can be investigated — relevant context for real incident

&#x20; response, where the right log may already be gone by the time

&#x20; someone looks



\## Why This Matters

Logs are only useful if you know where to look and how much history is

actually available. A volatile-only journal means a crash or reboot can

destroy the exact evidence needed to diagnose what caused it — making it

persistent is a small, permanent fix for that blind spot. Filtering by

priority and time turns an unusable flood of log lines into something

that can actually be triaged during an incident. And understanding

rotation means not being caught off guard when older evidence has

already aged out of the active log file — knowing the rotated filename

pattern is what turns a dead end into a working search.



\## Notes / Issues Encountered



\*\*Issue 1 — Journal was memory-only by default:\*\*

`/var/log/journal` didn't exist, meaning journald was only logging to

`/run/log/journal`, which is wiped on every reboot. Fix: created

`/var/log/journal` and restarted `systemd-journald` to switch it to

persistent, disk-backed logging; verified across a real reboot with

`journalctl --list-boots`.



\*\*Issue 2 — Expected SSH log entry had already rotated out:\*\*

A search for an earlier root-login-denial event in `/var/log/secure`

returned nothing, because the log had rotated since that test occurred.

Fix: located the rotated archive (`/var/log/secure-<date>`) and searched

that file instead — a reminder that active logs only cover a limited

recent window, and rotated files need to be checked separately for

older events.

