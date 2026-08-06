\# Phase 9: Scheduling



\## Objective

Explore and verify the three main task-scheduling mechanisms on a CentOS

server: cron (recurring), `at` (one-time), and systemd timers (recurring,

systemd-native).



\## Steps Taken



\### 1. Cron

\- Created a user crontab entry with `crontab -e` to run a command every

&#x20; minute (`\* \* \* \* \*`)

\- Initially used a static string, then switched to `date` to get a

&#x20; timestamped record of each execution

\- Verified execution both by checking the output file and by tailing

&#x20; system cron logs (`sudo tail -f /var/log/cron`, and cross-checked with

&#x20; `journalctl -u crond`), confirming each run was logged with the exact

&#x20; command, PID, and time

\- Removed the test crontab entry afterward (`crontab -e`, deleted the

&#x20; line) and confirmed with `crontab -l`



\### 2. `at`

\- Confirmed the `at` binary was installed (`which at`) and that its

&#x20; daemon, `atd`, was active (`systemctl status atd`) — the daemon is

&#x20; named `atd`, not `at`

\- Scheduled a one-time job to run one minute in the future:

&#x20; `echo "date >> \~/at\_test.log" | at now + 1 minute`

\- Verified it appeared in the queue with `atq`, executed once at the

&#x20; scheduled time, and automatically removed itself from the queue

&#x20; afterward — unlike cron, which persists and repeats

\- Cleaned up the test output file



\### 3. Systemd Timers

\- Created a paired unit-file setup in `/etc/systemd/system/`:

&#x20; - `sentinel-test.service` — a `oneshot` service running `date` to a

&#x20;   log file

&#x20; - `sentinel-test.timer` — the timer unit, using `OnBootSec=1min` for

&#x20;   the first run after boot and `OnUnitActiveSec=1min` to repeat every

&#x20;   minute thereafter

\- Reloaded the systemd daemon (`systemctl daemon-reload`) and enabled +

&#x20; started the timer (`systemctl enable --now sentinel-test.timer`) —

&#x20; the timer unit is what gets enabled, not the service directly

\- Verified with `systemctl list-timers | grep sentinel` and

&#x20; `systemctl status sentinel-test.timer`, and confirmed the log file was

&#x20; being written to on schedule

\- Cleaned up by disabling the timer, removing both unit files, reloading

&#x20; the daemon, and deleting the test log



\## Why This Matters

Automation is core to server administration — recurring maintenance,

one-off deferred tasks, and time-based triggers all rely on these three

mechanisms. Cron remains the most widely used for simple recurring jobs

and is portable across almost any Linux system. `at` fills the gap for

genuinely one-time tasks where a persistent cron entry would be the wrong

tool. Systemd timers are increasingly the RHEL/CentOS-native preferred

approach, since they integrate with `systemctl`, journal logging, and

unit dependencies in a way cron and `at` can't — worth knowing all three

so the right tool gets picked for the job.



\## Notes / Issues Encountered



\*\*Issue 1 — `systemctl status at` failed with "Unit at.service could not

be found":\*\*

The `at` command-line tool and its backing daemon don't share a name —

the daemon is `atd`, not `at`. Fix: queried `systemctl status atd`

instead, which correctly showed the daemon active and running.



\*\*Issue 2 — `/var/log/cron` not readable as a normal user:\*\*

`tail -f /var/log/cron` returned "Permission denied" when run without

elevated privileges, since cron's system log is root-only by default.

Fix: re-ran with `sudo`.

