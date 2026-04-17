---
tags: [project, vulnbrief, intel, sources, tools]
created: 2026-04-17
parent: "[[VulnBrief - Product Plan]]"
---

# VulnBrief — Intelligence Sources Reference

Full catalogue of sources to integrate or monitor. Grouped by purpose.

---

## Tier 1 — Free, High Quality, API-First (MVP)

These are the foundation. All free, reliable, and well-documented.

| Source | What it provides | API | Update cadence |
|--------|-----------------|-----|---------------|
| **NVD (NIST)** | Full CVE detail, CVSS vectors, CPE, references | REST | Continuous |
| **CISA KEV** | Confirmed exploited vulnerabilities — P0 by definition | JSON feed | Daily |
| **EPSS (First.org)** | 30-day exploitation probability per CVE | REST + daily CSV | Daily |
| **OSV.dev (Google)** | Open source ecosystem vulns: npm, PyPI, Go, Maven, Rust, RubyGems | REST | Real-time |
| **GitHub Advisory DB** | Package-level advisories, more readable than OSV | GraphQL | Continuous |
| **ExploitDB** | Public POC and exploit archive — POC availability signal | Download + scrape | Periodic |

---

## Tier 2 — Exposure & Scanning Intelligence (Phase 2)

Add these once the CVE data layer is solid.

### Shodan
- **What:** Internet-exposed assets with banner/service data and vulnerability filters
- **Why:** Shows real-world exposure counts for affected software versions — answers "how many internet-facing instances of Apache X.Y are out there right now?"
- **2026 note:** Vulnerability filters now essential for spotting IoT/ICS targets that AI-driven botnets hit first
- **Cost:** $49/mo API key; free for basic searches
- **Integration:** Query by CPE string from NVD data → show exposure count on CVE detail pages

### Censys
- **What:** Certificate transparency + internet scanning; better cloud asset coverage than Shodan
- **Why:** Catches shadow cloud instances Shodan misses. Critical for "are you actually exposed?" questions
- **Cost:** Free research tier; commercial for production
- **Integration:** Cross-reference with Shodan for fuller exposure picture

### GreyNoise
- **What:** Mass scanning and exploitation signals — separates internet background noise from targeted attacks
- **Why:** Tells you if a vulnerability is being actively probed by everyone (opportunistic) vs being weaponised against specific targets (targeted)
- **2026 note:** "Noise cancellation" is increasingly valuable as AI-driven scanning volume rises
- **Cost:** Free community API; $500/mo for full access
- **Integration:** Badge on CVE page: "🌐 Actively scanned by mass scanners" vs "🎯 Targeted exploitation signals"

---

## Tier 3 — Threat Context (Phase 2–3)

### Hudson Rock
- **What:** Infostealer credential and session cookie exposure database
- **Why:** In 2026, many breaches start with a stolen session cookie rather than a technical exploit. Pre-condition for many attack chains is "valid session token available on dark web"
- **Cost:** Free lookup; commercial for bulk
- **Integration:** For high-profile CVEs involving auth bypass, surface "stolen credentials for this service are actively traded" as a pre-condition multiplier

### OpenCVE
- **What:** Self-hosted, customisable CVE alert subscriptions by vendor/product
- **Why:** Solves the firehose problem for VulnBrief users — they subscribe to their stack, get only relevant alerts
- **Cost:** Free, open source (self-hosted)
- **Integration:** Could power VulnBrief's own alert/watchlist feature rather than building from scratch

### CVECrowd
- **What:** Community signal — practitioners sharing and discussing CVEs
- **Why:** Social signal of what security people actually care about vs what CVSS says is important
- **Cost:** Free
- **Integration:** "Trending in community" badge on CVE pages

---

## Tier 4 — Offensive/Research Signal (Research input, not direct integration)

These inform what we build but aren't integrated as data sources.

### VulHunt (Binarly) — Community Edition
- **What:** AI-powered vulnerability detection in compiled software and firmware — finds deep logic flaws
- **Why:** Shows what classes of vulnerabilities AI is finding at scale in 2026 — shapes what pre-condition categories to cover
- **Use:** Research tool for understanding emerging vuln classes. Informs content, not data pipeline.

### BlacksmithAI
- **What:** Open-source multi-agent AI framework for automated penetration testing
- **Why:** Shows how AI chains vulnerabilities in practice — primary research input for exploit chain mapping
- **Use:** Run it against lab environments to understand real-world chain patterns. Feeds into chain map content.

### Betterleaks (from Gitleaks creator)
- **What:** Leaked secrets detection — rapidly surfaces exposed credentials in code/repos
- **Why:** Secret leakage is a pre-condition for many attack chains. In 2026, AI can parse leaked codebases rapidly.
- **Use:** Could power a "leaked credentials" pre-condition check for supply chain CVEs

---

## Scanning Feature Sources

For the planned scanning capability (user enters their domain/IP, gets exposure analysis):

| Tool / API | What it enables |
|-----------|----------------|
| Shodan API | Retrieve exposed services and known vulns for an IP range |
| Censys API | Certificate and service data for a domain |
| GreyNoise API | Check if a specific IP has been seen scanning/exploiting |
| Nuclei (internal) | Active vuln detection templates — run against authorised targets |
| OSV.dev | Match declared dependencies against affected package versions |

**Important:** Scanning requires explicit user consent and authorisation confirmation before running any active probes. Passive lookups (Shodan/Censys/GreyNoise data already collected) can run without active scanning the target.

---

## Source Priority Matrix

```
High value, free → integrate immediately (NVD, KEV, EPSS, OSV, GitHub)
High value, cheap → integrate Phase 2 (Shodan, GreyNoise community)
High value, freemium → integrate Phase 2-3 (Censys, Hudson Rock)
Research input → use to inform content, not pipeline (VulHunt, BlacksmithAI)
Skip → VirusTotal GTI (expensive, wrong focus)
```

---

## Related

- [[VulnBrief - Product Plan]] — full architecture and implementation plan
- [[Vulnerability Prioritisation]] — the triage framework these sources feed into
- [[Project Glasswing]] — the research initiative these sources support
