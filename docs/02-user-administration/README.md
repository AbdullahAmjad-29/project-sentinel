\# Phase 2: User \& Access Management



\## Objective

Create a properly privileged, non-root operational user and enforce

security best practices around authentication, password policy, and

fine-grained file access — rather than relying on the root account

directly.



\## Steps Taken



\### 1. User \& Group Creation

\- Created user `sentinel\_admin` with `useradd`

\- Created group `sentinel\_ops` with `groupadd`

\- Added `sentinel\_admin` to `sentinel\_ops` using `usermod -aG`

&#x20; (append mode, to avoid wiping existing group memberships)



\### 2. Sudo Configuration

\- Verified `sentinel\_admin` had zero privileges by default

\- Granted sudo access via a dedicated rule in `/etc/sudoers`

&#x20; (edited safely using `visudo`, never edited directly):

&#x20; `sentinel\_admin ALL=(ALL) ALL`

\- Verified access with `sudo whoami` returning `root`



\### 3. Password Aging Policy

\- Set via `chage -m 1 -M 90 -W 7 sentinel\_admin`

\- Minimum 1 day between changes, maximum 90-day expiry,

&#x20; 7-day advance warning before expiry



\### 4. PAM Password Complexity

\- Edited `/etc/security/pwquality.conf`

\- Enforced: `minlen = 10`, `minclass = 3`, `maxrepeat = 3`

\- Ensures strong, varied passwords system-wide



\### 5. ACLs (Access Control Lists)

\- Created `/opt/sentinel\_logs` owned by root

\- Granted `sentinel\_admin` explicit `rwx` access via

&#x20; `setfacl -m u:sentinel\_admin:rwx /opt/sentinel\_logs`

\- Demonstrates fine-grained access without changing ownership

&#x20; or group membership



\## Why This Matters

Logging in and operating as root directly is a common but risky habit —

a single mistake has unlimited blast radius, and there's no per-user

accountability. This phase builds the standard real-world pattern:

a named, auditable user account with only the access it actually needs,

enforced through layered controls (sudo scoping, password policy,

authentication rules, and ACLs) rather than a single all-or-nothing

switch.



\## Notes / Issues Encountered

\- Initial `sed` substitution for `minlen` appeared not to apply on

&#x20; first check; re-running the command and verifying with `grep -n`

&#x20; confirmed it had in fact succeeded — a good reminder to always

&#x20; verify configuration changes rather than assume a command worked.

