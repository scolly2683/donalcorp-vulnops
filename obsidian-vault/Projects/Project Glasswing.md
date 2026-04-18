---
tags: [project, glasswing, vulnerability-management, prioritisation, startup]
created: 2026-04-17
status: active
---

# Project Glasswing

Vulnerability prioritisation initiative — getting ahead of the rising volume of critical CVEs through better tooling, deeper pre-condition analysis, and exploit chain mapping.

---

## Goals

- Build a repeatable, automatable CVE intake and triage process
- Go beyond CVSS — use EPSS, CISA KEV, pre-conditions, and chain potential to prioritise accurately
- Test findings against real (authorised) environments to validate actual business risk
- Identify market gaps in vulnerability tooling large enough to build a startup around
- Help DonalCorp clients understand their real exposure, not just their patch backlog

---

## What "Prepared" Looks Like

- [ ] Repeatable CVE triage process running with `/vuln-triage`
- [ ] Pre-condition checklists for every P0/P1 CVE
- [ ] POC library in `donalcorp-vulnops/` for the highest-risk vulnerabilities
- [ ] A shortlist of market gaps with product/startup potential
- [ ] Automated CVE feed monitoring (routine candidate)

---

## Triage Framework

See [[Vulnerability Prioritisation]] for the full framework.

**Priority scale:**

| Rating | Trigger |
|--------|---------|
| P0 — Act now | CISA KEV, OR EPSS > 50%, OR weaponised public exploit + low pre-conditions |
| P1 — This week | EPSS > 10%, OR public PoC, OR CVSS ≥ 9 + low pre-conditions |
| P2 — This sprint | CVSS ≥ 7, moderate pre-conditions, patch available |
| P3 — Backlog | High pre-conditions, limited exposure |
| P4 — Monitor | Low EPSS, no public PoC, high pre-conditions |

---

## Pre-Condition Focus

The biggest insight in modern vuln management: **CVSS measures worst-case severity, not real-world exploitability.** Pre-conditions are the gap between the two.

Key pre-conditions to always check:
- Network position (internet-facing vs internal vs localhost)
- Authentication required (none / user / admin)
- Non-default configuration required
- User interaction required
- OS / runtime / version specificity
- Race condition reliability

---

## Exploit Chain Thinking

Individual vulnerabilities are often medium risk. Chained, they become critical.

Always ask:
- What does initial exploitation enable next?
- Does this weakness lower the bar for another attack?
- Is there a known chain in the wild that includes this CVE?

---

## Market Gap Log

Gaps identified during research that could support a product or startup.

| Date | Gap | Notes |
|------|-----|-------|
| — | — | Add gaps here as found |

---

## Active CVEs

| CVE | CVSS | EPSS | Priority | Status |
|-----|------|------|----------|--------|
| [[CVE-2026-32746]] | 9.8 | — | — | POC built |

---

## Useful Tools for Triage

| Tool | Purpose | Link |
|------|---------|------|
| EPSS (First.org) | 30-day exploitation probability | https://www.first.org/epss |
| CISA KEV | Known exploited vulnerabilities list | https://www.cisa.gov/known-exploited-vulnerabilities-catalog |
| Shodan | Internet exposure of affected software | https://shodan.io |
| GreyNoise | Active scanning/exploitation in the wild | https://greynoise.io |
| Nuclei | Template-based vulnerability detection | https://nuclei.projectdiscovery.io |
| ExploitDB | Public exploit and PoC archive | https://www.exploit-db.com |
| NVD | CVE details and references | https://nvd.nist.gov |

---

## Routine Ideas

Recurring tasks worth automating:
- **Daily CVE feed** — pull new Critical/High CVEs, auto-triage via EPSS + KEV, flag P0/P1s
- **Weekly exposure scan** — check whether P0/P1 CVEs affect known internet-exposed assets
- **Market gap digest** — weekly summary of triage notes flagged as potential opportunities

---

## Related

- [[CVE-2026-32746]] — first documented CVE in Glasswing scope
- [[Vulnerability Prioritisation]] — full triage framework
- [[Remediation]] — remediation guidance
