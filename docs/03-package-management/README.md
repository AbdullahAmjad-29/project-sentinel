\# Phase 3: Package Management



\## Objective

Understand and work with RHEL-family package management (DNF/YUM),

including repository configuration, building a local package

repository from scratch, and enforcing package authenticity through

GPG signing — reflecting how enterprise environments manage trusted,

often offline, software distribution.



\## Steps Taken



\### 1. Repository Investigation

\- Confirmed active repos via `dnf repolist`: appstream, baseos, epel,

&#x20; extras, and a pre-configured localrepo

\- Verified repo URLs pointed to `vault.centos.org` (CentOS 8's

&#x20; official EOL archive), explaining why updates worked despite

&#x20; CentOS 8 being end-of-life since December 2021



\### 2. Local Repository Creation

\- Populated `/localrepo` with real `.rpm` packages

\- Installed `createrepo\_c` and generated repo metadata with

&#x20; `createrepo\_c --update /localrepo`



\### 3. GPG Key Generation \& Signing

\- Generated an RSA 3072-bit GPG key pair (`gpg --full-generate-key`)

\- Exported the public key to `/localrepo/RPM-GPG-KEY-sentinel`

\- Imported the key into RPM's trust store (`rpm --import`)

\- Installed `rpm-sign` and signed all packages in the repo with

&#x20; `rpm --resign`



\### 4. Enforcing GPG Verification

\- Added `gpgcheck=1` and `gpgkey=` to `/etc/yum.repos.d/local.repo`

\- Removed a conflicting legacy `gpgcheck=0` line that was silently

&#x20; overriding enforcement (last-defined value wins in repo configs)

\- Verified enforcement with a scoped install using

&#x20; `--disablerepo="\*" --enablerepo="localrepo"`



\## Why This Matters

Without GPG enforcement, a package manager will install anything

placed in a repository, signed or not — meaning a compromised or

malicious package could be installed without warning. Signing and

verification ensure packages can be trusted to have come from a

known source and haven't been tampered with. This matters even more

for offline/local repos, which are common in secured enterprise

environments where servers can't reach the public internet.



\## Notes / Issues Encountered



\*\*Issue 1 — Checksum mismatch after signing:\*\*

Ran `createrepo\_c` before signing packages with `rpm --resign`.

Signing modifies a package's binary content (embeds the signature

in the RPM header), which changes its checksum. Since repodata had

already recorded the pre-signing checksums, DNF flagged every

package as having an "incorrect checksum" on install.

\*\*Fix:\*\* always run `createrepo\_c` AFTER signing, never before.



\*\*Issue 2 — Interrupted dnf process:\*\*

Pressed Ctrl+C during a `dnf update` cleanup phase near completion.

Verified system integrity afterward with `dnf check` (came back

clean). Lesson: avoid interrupting package manager operations

mid-write, as it risks corrupting the RPM database.

