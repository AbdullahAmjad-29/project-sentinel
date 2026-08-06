\# Phase 8: Security



\## Objective

Harden a CentOS server through SELinux enforcement and troubleshooting,

firewalld rule refinement, SSH hardening, and TLS certificate generation.



\## Steps Taken



\### 1. SELinux

\- Confirmed SELinux was enforcing under the `targeted` policy (`getenforce`,

&#x20; `sestatus`)

\- Reproduced a real SELinux denial: moving a file into `/var/www/html` with

&#x20; `mv` preserved its original `user\_tmp\_t` context instead of the expected

&#x20; `httpd\_sys\_content\_t`, causing httpd to be denied access; fixed by

&#x20; relabeling the file with `restorecon -v`

\- Reviewed httpd-related SELinux booleans (`getsebool -a | grep httpd`) and

&#x20; persistently enabled `httpd\_can\_network\_connect\_db` with `setsebool -P`,

&#x20; then verified the change held



\### 2. Firewalld — Rich Rules

\- Added a rich rule restricting SSH access to source `192.168.249.0/24`

&#x20; (`firewall-cmd --add-rich-rule=...`), reloaded, and verified it was live

&#x20; with `--list-rich-rules`

\- Discovered the broad `ssh` entry in the zone's general allowed-services

&#x20; list was overriding/duplicating the rich rule, making the restriction

&#x20; ineffective as written

\- Removed the broad service-level allow (`firewall-cmd --zone=public

&#x20; --remove-service=ssh --permanent` + `--reload`), then verified SSH access

&#x20; was still reachable only via the rich rule by confirming a fresh

&#x20; connection from within the allowed subnet still succeeded



\### 3. SSH Hardening

\- Edited `/etc/ssh/sshd\_config`:

&#x20; - `PermitRootLogin no` — disables direct root login over SSH

&#x20; - `PasswordAuthentication no` — disables password-based login, since

&#x20;   key-based auth for `sentinel\_admin` was already confirmed working in

&#x20;   Phase 7

&#x20; - Verified `PubkeyAuthentication` was not disabled (left at its default

&#x20;   `yes`, commented out)

\- Restarted `sshd` (`systemctl restart sshd`) and verified from a fresh

&#x20; terminal, without closing the existing session, that:

&#x20; - `ssh sentinel\_admin@192.168.249.150` still logs in via key, no password

&#x20;   prompt

&#x20; - `ssh root@192.168.249.150` is refused outright ("Permission denied")



\### 4. OpenSSL Self-Signed Certificate

\- Generated a 2048-bit RSA private key and a self-signed X.509 certificate

&#x20; in one step:

&#x20; `openssl req -x509 -newkey rsa:2048 -keyout /etc/pki/tls/private/sentinel.key

&#x20; -out /etc/pki/tls/certs/sentinel.crt -days 365 -nodes`

\- Set Common Name to `sentinel-server`, validity to 1 year

\- Verified the private key was saved with `600` permissions (root-only) and

&#x20; the certificate with `644` (world-readable, as expected for a public cert)

\- Confirmed subject and validity dates with

&#x20; `openssl x509 -in sentinel.crt -noout -subject -dates`



\## Why This Matters

Security on a server isn't one control — it's layered. SELinux contains

what a compromised process can touch even if the network perimeter is

breached. Firewalld defines exactly what's reachable, and a rule that

looks restrictive but is silently overridden by a broader allow is worse

than no rule at all, since it gives false confidence. SSH hardening

removes the two most common brute-force/compromise vectors on any

internet-facing service: root as a login target, and passwords as a

guessable credential. A self-signed certificate demonstrates the same TLS

mechanics used in production, even without a public CA to sign it.



\## Notes / Issues Encountered



\*\*Issue 1 — SELinux context not inherited correctly by `mv`:\*\*

Moving a file into `/var/www/html` with `mv` kept its source context

(`user\_tmp\_t`) instead of picking up the target directory's expected

context (`httpd\_sys\_content\_t`), causing httpd to be denied access even

though standard Linux permissions were correct. Fix: `restorecon -v` on

the file to relabel it to the correct context for its new location.



\*\*Issue 2 — Rich rule silently overridden by the broader service allow:\*\*

A firewalld rich rule restricting SSH to `192.168.249.0/24` had no real

effect, because the `public` zone's general `services` list still

included `ssh`, which took precedence. Fix: removed `ssh` from the

zone's permanent services list (`--remove-service=ssh --permanent`) so

only the rich rule governs SSH access going forward.



\*\*Issue 3 — `sshd\_config` edit landed on the wrong line initially:\*\*

The first attempt to set `PasswordAuthentication no` appeared to save

successfully but a follow-up `grep` showed the active (uncommented) line

was still `yes` — the edit had actually modified a commented example

line elsewhere in the file instead. Fix: used `grep -n` to find the exact

line number of the active directive, then reopened the file with

`vi +<line\_number>` to edit that specific line directly, avoiding

ambiguity from multiple similarly-named lines in the file.

