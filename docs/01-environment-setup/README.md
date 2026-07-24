\# Phase 1: Environment Setup



\## Objective

Set up a CentOS 8 virtual machine to serve as the base system for

Project Sentinel — a hardened, self-monitoring Linux server built

to practice and demonstrate core Linux system administration skills.



\## VM Specifications

\- Hypervisor: VMware Workstation

\- OS: CentOS Linux 8 (kernel 4.18.0-240.el8.x86\_64)

\- RAM: \[fill in]

\- CPU: \[fill in]

\- Disk: \[fill in]

\- Network: NAT (to be reconfigured in the Networking phase)



\## Steps Taken

1\. Installed CentOS 8 Minimal on a fresh VM.

2\. Verified OS and kernel version using `cat /etc/os-release \&\& uname -r`.

3\. Changed the default hostname from `osboxes` to `sentinel-server`

&#x20;  using `hostnamectl set-hostname sentinel-server`.



\## Why This Matters

A minimal install was chosen deliberately — production servers rarely

run a GUI or unnecessary packages, since every extra package increases

attack surface and resource usage. Documenting the base state here

establishes a clean starting point that later phases (storage,

networking, security) will build on and modify.



\## Notes / Issues Encountered

\- CentOS 8 reached end-of-life in December 2021, meaning default repos

&#x20; are no longer maintained. This will need to be addressed in the

&#x20; Package Management phase.

