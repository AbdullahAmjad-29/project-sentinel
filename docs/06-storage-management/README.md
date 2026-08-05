\# Phase 6: Storage Management



\## Objective

Learn Linux storage management from raw disk to usable, persistent

filesystems — covering partitioning, filesystem creation, UUID-based

mounting, the full LVM stack (Physical Volume → Volume Group →

Logical Volume), and swap — all verified to survive a real reboot.



\## Steps Taken



\### 1. Adding a Practice Disk

\- Added a second 5GB virtual disk in VMware to avoid risking the

&#x20; existing OS disk

\- Diagnosed and fixed a real BIOS boot-order issue caused by the

&#x20; new disk being detected before the OS disk



\### 2. Partitioning and Filesystem (Raw Partition)

\- Created a partition on the new disk with `fdisk`

\- Formatted it with XFS using `mkfs.xfs`

\- Mounted it at `/mnt/sentinel\_data`, retrieved its UUID with `blkid`,

&#x20; and made the mount permanent via `/etc/fstab`

\- Verified persistence safely with `mount -a` before rebooting



\### 3. LVM (Logical Volume Manager)

\- Repurposed the same disk for LVM by unmounting it and removing its

&#x20; fstab entry

\- Created a Physical Volume with `pvcreate`, a Volume Group

&#x20; (`sentinel\_vg`) with `vgcreate`, and a 3GB Logical Volume

&#x20; (`sentinel\_lv`) with `lvcreate`

\- Formatted and mounted the Logical Volume at `/mnt/sentinel\_lvm`,

&#x20; added a UUID-based fstab entry, and confirmed it survived a reboot



\### 4. Swap

\- Created a 512MB swap file with `fallocate`, secured it with

&#x20; `chmod 600`, formatted it with `mkswap`, and activated it with

&#x20; `swapon`

\- Made it persistent via `/etc/fstab` and confirmed it auto-activated

&#x20; after a reboot, alongside the existing swap partition



\## Why This Matters

Real servers rarely use plain static partitions for anything beyond

the base OS install — LVM is the standard because it allows storage

to be resized, extended across disks, and snapshotted without

downtime, which plain partitioning can't do. Correctly using UUIDs

instead of device names (`/dev/sda1`) in `/etc/fstab` avoids a common

production failure where storage silently fails to mount after a

reboot because a device name shifted. Swap management matters for

memory-constrained systems, and knowing both partition-based and

file-based swap gives flexibility depending on the environment.



\## Notes / Issues Encountered



\*\*Issue 1 — New disk not detected by the running VM:\*\*

After adding a second virtual disk in VMware, `lsblk` still showed

only the original OS disk. Cause: the disk was added while the VM

was running, and VMware didn't hot-detect it into the live system.

\*\*Fix:\*\* rebooted the VM so the kernel re-scanned hardware at boot

and picked up the new disk correctly.



\*\*Issue 2 — BIOS boot order broken after adding the disk:\*\*

After that reboot, the VM failed to boot at all (attempted a PXE

network boot and failed with "Operating System not found"). Cause:

adding the new, empty disk shifted the BIOS boot device order so

the empty disk was tried before the disk containing CentOS.

\*\*Fix:\*\* entered the VM's BIOS setup (VM → Power → Power On to

Firmware), opened the Boot menu's Hard Drive submenu, and moved the

existing OS disk above the new empty disk in priority order, then

saved and exited.



\*\*Issue 3 — `pvcreate` failed with "Can't open /dev/sda1

exclusively. Mounted filesystem?":\*\*

Attempted to run `pvcreate /dev/sda1` while that partition was still

mounted at `/mnt/sentinel\_data` from the earlier raw-partition

exercise. LVM refuses to claim a disk that's actively mounted, since

doing so could corrupt live data. \*\*Fix:\*\* unmounted the partition

with `umount /mnt/sentinel\_data` and removed its stale `/etc/fstab`

entry before retrying `pvcreate`, which then succeeded (after

confirming a prompt to wipe the old XFS signature still present on

the partition).

