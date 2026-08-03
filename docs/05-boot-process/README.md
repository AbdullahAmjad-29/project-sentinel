\# Phase 5: Boot Process



\## Objective

Understand the Linux boot sequence — from firmware through GRUB2

to the kernel and systemd — and practice safely modifying boot-level

configuration, including kernel parameters and recovery boot targets.



\## Steps Taken



\### 1. Firmware Identification

\- Checked for `/sys/firmware/efi` to confirm boot mode

\- Confirmed the VM boots in legacy BIOS mode (directory absent)



\### 2. GRUB2 Configuration

\- Reviewed `/etc/default/grub`, the human-editable GRUB settings file

\- Identified `GRUB\_CMDLINE\_LINUX`, containing kernel parameters

&#x20; `crashkernel=auto resume=UUID=... rhgb quiet`

\- Removed `quiet` using `sed -i "s/ quiet//" /etc/default/grub`,

&#x20; so future boots display kernel messages instead of a silent splash

\- Regenerated the actual boot config with

&#x20; `grub2-mkconfig -o /boot/grub2/grub.cfg`, since GRUB reads the

&#x20; generated `/boot/grub2/grub.cfg`, not the human-edited file directly



\### 3. Verifying the Change

\- Rebooted the VM and logged back in successfully

\- Confirmed the live kernel command line with `cat /proc/cmdline`

\- Verified `quiet` was absent while `rhgb` remained, confirming the

&#x20; change was applied and persisted through a real reboot



\### 4. Recovery Boot Targets

\- Reviewed the conceptual difference between Rescue mode (filesystems

&#x20; mounted read-write, minimal services) and Emergency mode (root

&#x20; filesystem read-only, bare minimum environment)

\- Attempted a one-time boot into rescue mode by editing the GRUB boot

&#x20; entry live (appending `systemd.unit=rescue.target` to the `linux`

&#x20; line, then booting with Ctrl+X) — this type of edit is temporary

&#x20; and does not modify the saved GRUB configuration



\### 5. Systemd Targets

\- Checked the system's default boot target using

&#x20; `systemctl get-default`

\- Confirmed `multi-user.target` — full multi-user mode without a

&#x20; graphical interface, standard for a minimal server install



\## Why This Matters

Boot-level knowledge is critical for diagnosing servers that won't

start normally. Kernel parameters control low-level system behavior

at startup; GRUB is the layer that applies them. Rescue and emergency

modes exist specifically for situations where the normal boot process

fails — for example, resetting a lost root password or fixing a

corrupted config file that's preventing services from starting.

Understanding systemd targets also clarifies what "normal" boot

actually means on a given system (server vs. desktop).



\## Notes / Issues Encountered



\*\*Issue — First rescue mode attempt booted normally instead:\*\*

The live GRUB edit adding `systemd.unit=rescue.target` did not appear

to take effect on the first attempt; the system booted to a standard

login prompt instead of rescue mode. Since this was a live, one-time

edit (not a saved config change), the most likely causes are: the

text not landing precisely at the end of the `linux` line, or Ctrl+X

not being pressed immediately after typing. This was not fully

re-diagnosed, but the underlying concept (rescue vs. emergency mode,

and how a one-time GRUB edit works) was confirmed understood.

Follow-up: revisit this hands-on if time allows, to get a fully

successful rescue mode boot documented.

