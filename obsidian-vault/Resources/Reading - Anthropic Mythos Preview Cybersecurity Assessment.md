---
tags: [research, threat-intel, mythos, glasswing, exploit-chaining, zero-day, n-day, priority-read]
created: 2026-04-18
source: Anthropic Red Team Blog
author: Nicholas Carlini, Newton Cheng, Keane Lucas et al.
url: https://red.anthropic.com/2026/mythos-preview/
---

# Reading Note: Assessing Claude Mythos Preview's Cybersecurity Capabilities

> Anthropic Red Team, April 7, 2026

**Priority: CRITICAL — This is the foundational document for Project Glasswing.**

---

## What This Document Is

Anthropic's own technical disclosure of Claude Mythos Preview's offensive cybersecurity capabilities. Written by Anthropic's red team for researchers and practitioners. Unprecedented transparency from a frontier lab about AI-assisted vulnerability discovery and exploitation.

---

## Headline Capabilities

### Zero-Day Discovery
- Identified exploitable bugs in **every major OS and web browser**
- 27-year-old OpenBSD SACK TCP bug (crash any OpenBSD host remotely)
- 16-year-old FFmpeg H.264 vulnerability (out-of-bounds write, missed by all fuzzers)
- 17-year-old FreeBSD NFS RCE — **fully autonomous**, unauthenticated, remote, root
- Thousands of additional high/critical vulnerabilities under responsible disclosure (>99% unpatched at time of writing)

### N-Day to Working Exploit
> Starting from just a CVE identifier and a git commit hash — the entire process from turning these public identifiers into functional exploits now happens **much faster, cheaper, and without intervention.**

- Formerly took skilled researchers days to weeks per bug
- Mythos does it **autonomously, in hours, for under $2,000**
- This is the single most important sentence for VulnBrief's urgency argument

### Vulnerability Chaining (Key for VulnBrief)
Mythos demonstrated the ability to **independently identify, then chain together, a set of vulnerabilities** to achieve root:

Linux LPE chain example (4 vulnerabilities chained):
1. Vulnerability A → bypass KASLR (kernel address randomisation)
2. Vulnerability B → read contents of an important kernel struct
3. Vulnerability C → write to a previously-freed heap object
4. Heap spray → place a struct exactly where the write lands → root

Web browser chain:
1. Read primitive → KASLR bypass
2. Write primitive → JIT heap spray
3. Renderer sandbox escape
4. OS-level LPE
→ Single webpage visit → attacker writes to OS kernel

---

## The Economics (Critical Signal for VulnBrief)

| Task | Previous cost | Mythos cost |
|---|---|---|
| 1000-run OpenBSD scan | — | <$20,000 |
| Single FreeBSD NFS exploit | weeks of expert time | <$50 + hours |
| Complex read-chain LPE exploit | weeks | <$2,000, <1 day |
| Firefox JS engine exploits | 2/several hundred (Opus 4.6) | 181/same (Mythos) |

At <$2,000 per working exploit, N-day weaponisation is now economically trivial for any motivated attacker. This is the number to put in front of management.

---

## The N-Day Problem (Most Actionable for Defenders)

The article explicitly states that N-day exploitation is now automated from just a CVE ID + git commit:

> "The patch itself is a roadmap to the bug, and the only thing standing between disclosure and mass exploitation is the time it takes an attacker to turn that patch into a working exploit."

**Previous assumption:** patch window = weeks (time for attacker to reverse-engineer)
**Post-Mythos assumption:** patch window = hours to days

This directly validates VulnBrief's PoC recency signal and "time to weaponisation" feature ideas. It also means the KEV catalogue's "confirmed exploited" threshold is now the wrong signal to wait for.

---

## How Mythos Works (Scaffold = How VulnBrief Should Think About Chains)

The Mythos scaffold:
1. Launch isolated container with source code
2. Prompt: "Find a security vulnerability in this program"
3. Claude reads code → hypothesises vulnerabilities → runs program → tests → confirms
4. Claude ranks files 1–5 by bug likelihood, starts with 5s
5. Separate validator agent: "Is this real and interesting?"
6. If tasked: "Write an exploit for triage purposes"

**The chain types Mythos exploits consistently:**

| Chain Role | Example | Technique |
|---|---|---|
| Info leak / KASLR bypass | Read from cpu_entry_area (fixed VA) | Out-of-bounds read / hardcopy bypass |
| Write primitive | Overwrite struct via UAF or OOB write | slab cross-cache reclaim |
| Code execution | Overwrite ops->peek with commit_creds | Heap spray into freed slot |
| Persistence | Append SSH public key via ROP | kern_openat + kern_writev |

These chain roles are the taxonomy VulnBrief needs for CVE chain linking.

---

## The Linux Kernel LPE Chain in Detail

Two fully documented N-day chains (worth studying for the chain taxonomy):

**Chain 1:** CVE-2024-47711 (ipset bit-write) + DRR qdisc UAF
- ipset bitmap: 1-bit OOB write → flip PTE R/W flag → make kernel page writable from userspace
- Target: passwd binary (setuid root) → overwrite with shellcode → root

**Chain 2:** CVE-2024-47711 (unix_stream OOB read) + CVE (tc scheduler UAF)  
- One-byte read → controlled fake skb → arbitrary kernel read (one byte at a time)
- HARDENED_USERCOPY bypass using cpu_entry_area (fixed VA, not slab-managed)
- Read own kernel stack → leak oob_skb pointer → locate ring page in kernel VA
- Second UAF (drr_class) → msgsnd() heap spray → commit_creds pivot → root

---

## Defender Recommendations (from Article)

1. **Use current frontier models now** — Opus 4.6 already finds hundreds of high/critical bugs. Don't wait for Mythos access.
2. **Shorten patch cycles** — enable auto-update, treat dependency CVE bumps as urgent
3. **Automate incident response pipeline** — model-driven triage, alert prioritisation, parallel hunts
4. **Think beyond vuln finding** — models for patch proposal, PR security review, cloud misconfiguration analysis
5. **Review disclosure policies** — prepare for scale (thousands of bugs, not dozens)

> "Mitigations whose security value comes primarily from **friction** rather than hard barriers may become considerably weaker against model-assisted adversaries."

This is the key architectural insight: KASLR and W^X (hard barriers) still work. Extra manual steps (friction) no longer slow attackers down.

---

## Implications for VulnBrief

### Validates everything
- EPSS-first prioritisation: N-day window is now hours, EPSS must lead
- Pre-condition analysis: Mythos starts from "is this reachable?" — same question our wizard asks
- Remediation urgency: patch delay is now critical because automation makes weaponisation cheap
- PoC recency signal: once a PoC exists, Mythos-class tools can build a working exploit within the day

### The CVE Chain Index feature is now URGENT
Mythos chains vulnerabilities autonomously. VulnBrief's defensive answer must be:
- "These CVEs are known to chain — patch them together, not separately"
- Chain role taxonomy: **info_leak → write_primitive → code_exec → persistence**
- If an attacker has CVE-A (info leak) + CVE-B (write) in your environment, the chain is complete
- VulnBrief composite score must multiply when a CVE is part of a documented chain

### New metric to surface: "Chain Completion Risk"
If a user's asset profile shows:
- CVE-A (KASLR bypass / info leak) — EPSS 0.3, in their environment
- CVE-B (write primitive) — EPSS 0.1, in their environment

Neither CVE alone is critical. Together: full LPE. VulnBrief should detect chain overlap and flag it.

### Reachability is now the bottleneck
> "The set of vulnerabilities that actually matter in the wild is the set that is reachable inside a real attack path at a cost attackers are willing to pay."

Post-Mythos, the cost to weaponise is near-zero. Reachability is now the only meaningful filter.

---

## Key Quotes

> "Non-experts can also leverage Mythos Preview to find and exploit sophisticated vulnerabilities. Engineers at Anthropic with no formal security training have asked Mythos Preview to find remote code execution vulnerabilities overnight, and woken up the following morning to a complete, working exploit."

> "The vulnerabilities that Mythos Preview finds and then exploits are the kind of findings that were previously only achievable by expert professionals."

> "We see no reason to think that Mythos Preview is where language models' cybersecurity capabilities will plateau."

> "Imagining a future where language models become much stronger still is difficult; it is tempting to hope that future models won't continue to improve at the current rate. But we should prepare with the belief that the current trend is likely to continue."

---

## Project Glasswing Connection

This document *is* the reason Project Glasswing exists. Anthropic launched Glasswing as a coordinated defensive response to Mythos Preview — giving defenders (critical infrastructure, open source projects) access first before broader release.

VulnBrief is the practitioner-facing tool in this same defensive posture: it operationalises the prioritisation logic that lets real security teams survive the transitional period.

---

## Related

- [[Project Glasswing]] — directly referenced; Glasswing is Anthropic's defensive programme built on Mythos
- [[VulnBrief - Product Plan]] — CVE Chain Index, Chain Completion Risk — new Phase 2 features
- [[VulnBrief - Competitive Intelligence]] — no competitor currently does chain-aware composite scoring
- [[Reading - Mythos Memory Loss TrustedSec]] — TrustedSec predicted this; this article confirms it
- [[Reading - Breaking the Web Vulnerability Chaining]] — the attack patterns Mythos uses are documented here
- [[Vulnerability Prioritisation]] — "friction is no longer a defence" changes the whole framework
