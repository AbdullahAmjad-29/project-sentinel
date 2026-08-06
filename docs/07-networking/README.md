\# Phase 7: Networking



\## Objective

Configure and verify core networking on a CentOS server: a static IP

and DNS via nmcli, hostname management, static routing concepts,

NIC redundancy concepts, key-based SSH access, file transfer, and

firewall management with firewalld.



\## Steps Taken



\### 1. Static IP and DNS (nmcli)

\- Inspected interfaces with `ip addr show`; found `ens160` on a

&#x20; DHCP-assigned address (192.168.249.128/24)

\- Set a static IP (192.168.249.150/24) and gateway (192.168.249.2)

&#x20; via `nmcli con mod` + `nmcli con up`, replacing DHCP

\- Set DNS servers (8.8.8.8, 1.1.1.1) via `nmcli`, since manual IPv4

&#x20; addressing stops DHCP from supplying DNS automatically

\- Verified resolution and connectivity with `cat /etc/resolv.conf`

&#x20; and `ping google.com`



\### 2. Hostname

\- Verified hostname via `hostnamectl` (set to `sentinel-server`,

&#x20; matching the value configured back in Phase 1)



\### 3. Static Routes

\- Added a temporary route for a placeholder subnet

&#x20; (`10.10.10.0/24 via 192.168.249.1`) using `ip route add`

\- Verified with `ip route show`, then removed it cleanly with

&#x20; `ip route del` (temporary, non-persistent — for demonstrating

&#x20; the concept only, not a real destination)



\### 4. Bonding / Teaming (conceptual)

\- Reviewed bonding (kernel-level, `bond0`) vs. teaming (userspace

&#x20; daemon, `team0`, RHEL's preferred modern approach) and common

&#x20; modes (active-backup, round-robin, LACP)

\- Not built hands-on in this environment — a single VM with one NIC

&#x20; can't meaningfully demonstrate real failover, so this was covered

&#x20; conceptually rather than spending setup time on a second virtual

&#x20; NIC for low practical payoff



\### 5. SSH Key-Based Authentication

\- Generated/reused an Ed25519 SSH key pair on the Windows host

\- Added the public key to `sentinel\_admin`'s `\~/.ssh/authorized\_keys`

&#x20; (not root — direct root SSH login is disabled by convention on

&#x20; real servers; a named, sudo-enabled user is used instead)

\- Verified a full password-less SSH login from the Windows host:

&#x20; `ssh sentinel\_admin@192.168.249.150`



\### 6. File Transfer (SCP)

\- Copied a test file from the Windows host to the VM with `scp`,

&#x20; authenticated via the same SSH key — confirmed content matched

&#x20; on arrival

\- `rsync` was considered as the more efficient tool for incremental

&#x20; transfers, but was not set up hands-on due to Windows package

&#x20; manager friction; SCP already proved the core file-transfer skill



\### 7. Firewalld

\- Found firewalld installed but inactive and disabled by default

\- Started and enabled it (`systemctl start` / `enable firewalld`)

\- Reviewed the default zone (`public`) and its allowed services

&#x20; (`cockpit`, `dhcpv6-client`, `http`, `https`, `ssh`)

\- Opened a custom port (8080/tcp) permanently, then applied it with

&#x20; `firewall-cmd --reload` and verified it was live



\## Why This Matters

A server's network identity needs to be stable and predictable —

static IPs and DNS avoid the address-drift problems DHCP can cause

for anything depending on this machine (SSH sessions, DNS records,

firewall rules, monitoring). Key-based SSH is the real-world standard

over password auth, and logging in as a named user rather than root

preserves accountability. Firewalld defines exactly what's reachable

from the network — leaving it off, or leaving unused services like

`http`/`cockpit` enabled on a server that doesn't need them, is

unnecessary exposed attack surface.



\## Notes / Issues Encountered



\*\*Issue 1 — nmcli rejected `ipv4.method manual` before an address

was set:\*\*

Running `nmcli con mod ens160 ipv4.method manual` before setting an

address failed with "ipv4.addresses: this property cannot be empty

for 'method=manual'". Fix: set `ipv4.addresses` first, then

`ipv4.method manual`, so the connection is never left in an invalid

manual-but-empty state.



\*\*Issue 2 — sudo password lost for sentinel\_admin:\*\*

After confirming SSH key login worked, `sudo su` failed repeatedly

because the password set for `sentinel\_admin` back in Phase 2 wasn't

remembered. Fix: switched back to root on the console and reset it

directly with `passwd sentinel\_admin` (root doesn't need the old

password), then confirmed `sudo su` worked from the SSH session with

the new password.



\*\*Issue 3 — `systemctl start firewalld` triggered a PolicyKit

identity prompt instead of running directly:\*\*

Running the command over SSH (without `sudo`) prompted for a polkit

identity selection rather than completing normally. Selecting the

correct identity and authenticating resolved it; prefixing the

command with `sudo` going forward avoids the interactive prompt.



\*\*Issue 4 — rsync unavailable on Windows / no clean install path:\*\*

Git Bash on this machine didn't include `rsync`, and no obvious

`winget` package matched a genuine rsync build. Rather than spend

significant setup time on a Windows-side workaround for a secondary

tool, this was scoped out — SCP already demonstrated the core

file-transfer competency, and rsync's incremental-sync advantage was

covered conceptually instead.

