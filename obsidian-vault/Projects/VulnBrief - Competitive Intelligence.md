---
tags: [project, competitive-analysis, startup, vulnerability-intelligence]
created: 2026-04-18
status: active
---

# VulnBrief — Competitive Intelligence

> Researched April 2026. Update when entering a new sales cycle or funding round.

---

## Competitive Landscape Map

```
CVE-driven / CMDB-anchored           BAS / Control Validation
        │                                       │
   [VulnBrief]                       [Cymulate] [AttackIQ]
   [Nucleus Security]
        │                            Autonomous Pentesting
   Scanning-anchored                         │
        │                             [NodeZero]
   [Tenable One]
   [Qualys VMDR]                    Cloud-native / ASM
        │                                  │
   Legacy scanners                   [Wiz UVM+ASM]
                                      [Orca Security]

CVE Awareness / Alerting (free tier)
        │
   [Intruder CVEMon]
   [Feedly CVE]
```

**VulnBrief sits in the CVE-driven / CMDB-anchored quadrant — the only SMB-priced player there.**
**CVEMon and Feedly sit in a separate free-tier awareness category — not direct competitors, but occupy attention in the same audience.**

---

## Platform Analysis

### Nucleus Security

**What it does:**
- Multi-source vulnerability aggregation and normalization (200+ connectors)
- AI-powered threat intelligence (Nucleus Insights, 2025): scans exploit repos, dark web, malware reports daily
- Customer Risk Score: custom org-specific scoring algorithm, operationalized in automation engine
- Asset deduplication across scanner outputs
- Automated ticketing and ownership assignment
- FedRAMP Moderate + SOC2

**CVE prioritization:** EPSS + KEV + Nucleus Insights AI threat intel + asset context. Custom scoring per org.

**Pricing:** ~$10–12/device/year (volume-tiered). SMB-hostile at minimum viable deployment.

**Gap for VulnBrief to exploit:** No SMB tier. No plain-English layer. Pre-condition analysis is missing.

**Our advantage:** Nucleus aggregates scanner output. VulnBrief is the CVE intelligence layer — they're complementary not competing.

---

### Tenable One

**What it does:**
- Vulnerability scanning (agent + agentless), CSPM, asset discovery
- Vulnerability Priority Rating (VPR): proprietary composite of CVSS + EPSS + threat intel + asset criticality
- Exposure Management platform (Tenable One)
- Sponsored EPSS research (Cyentia/FIRST 2024)

**CVE prioritization:** VPR score. EPSS is one input, not the lead signal. Asset criticality tagging required.

**Pricing:** Minimum 300 assets. Enterprise reality: $50K–$500K+/year.

**Gap for VulnBrief to exploit:** Entirely unaffordable for SMBs. No plain-English layer. VPR is a black box — practitioners don't trust what they can't inspect.

**Our advantage:** Transparent composite scoring (ours is open). SMB price point. Pre-condition questionnaire.

---

### AttackIQ

**What it does:**
- BAS platform — emulates ATT&CK techniques against your security controls
- 2,600+ enterprise deployments
- Adversarial Exposure Validation (AEV) platform (Feb 2025): AI-driven attack path analysis
- Excellent for measuring control coverage against MITRE ATT&CK

**CVE prioritization:** Not primary focus. CVEs are emulated but ATT&CK technique coverage is the measure.

**Pricing:** Enterprise custom. Three tiers (Flex / Ready! / Enterprise).

**Notable gaps:** No cloud testing. Only post-exploitation (misses delivery/initial access stage). Slower deployment vs Cymulate.

**Relevance to VulnBrief:** AttackIQ's AEV concept — synthesizing vulnerabilities + attack paths + threat intel — is the v3.0 direction for VulnBrief. Not a direct competitor now.

---

### Cymulate

**What it does:**
- Exposure Validation: full kill chain (pre-exploitation → post-exploitation)
- Same-day deployment advantage
- Cloud + Kubernetes + on-prem
- Remediation rules deployed directly into security stack
- 500+ enterprise customers. Gartner Customers' Choice 2024.
- Daily threat research integration — new CVE assessments within 24 hours

**CVE prioritization:** Exploitability-validated risk score. Does this CVE actually work against YOUR controls and environment? Considers compensating control effectiveness.

**Pricing:** ~$7K+ (floor). Direct quote required for enterprise.

**Relevance to VulnBrief:**
- The "compensating controls" angle is the key differentiator to borrow
- Cymulate asks: "Does your EDR stop this?" VulnBrief's pre-condition wizard should ask this too
- Add as Q6 to pre-condition wizard: "Do you have compensating controls for this attack vector?"

---

### Horizon3.ai NodeZero

**What it does:**
- Autonomous penetration testing — lateral movement, credential attacks, exploit chaining
- Deliberately NOT CVE-first: prioritizes misconfigurations, identity/access, credential reuse
- Dynamic exploit chains (no predefined scripts — actor-like behavior)
- Cloud, internal network, web app pentesting
- Proactive zero-day and N-day research

**CVE prioritization:** De-emphasized by design. Focus is on demonstrated feasibility, not CVE scores.

**Pricing:** Per active IP address model. Custom pricing.

**Why this matters:**
- NodeZero shows what attack paths look like in practice — misconfigs and identity often matter more than CVEs
- This reinforces VulnBrief's exploit chain context feature: chains should include non-CVE steps
- NodeZero's "show me the actual attack path" is v3.0 VulnBrief Agent territory (see management proposal)

**Our differentiation:** VulnBrief is CVE-driven and CMDB-anchored — operationally more useful for a Vulnerability Management programme than NodeZero's generic red team approach.

---

### Intruder CVEMon

**What it does:**
- Free CVE monitoring and awareness tool (separate from Intruder's paid scanner)
- Three views: Activity feed, Trending, CVE browser
- **Activity feed:** Event timeline — KEV additions, exploit discoveries, score changes — for CVEs matching your watchlist
- **Trending / Hypemeter:** Top 10 CVEs trending on social media in the last 24 hours. Scored 0–100 ("Hypemeter") measuring social attention, not exploitability. Refreshes every ~24 mins.
- **CVE browser:** Browse by technology, product, vulnerability type. Latest feed of published CVEs with description snippets.
- **CVE detail:** CVSS score + severity badge, "Exploit known" flag (fire icon), product/vendor tags, tabs for Overview / Scores / Known Exploits / Weaknesses.

**CVE prioritization:** CVSS-led. No EPSS shown on main view. No composite scoring. No pre-conditions. Alerting based on KEV status changes and new exploit disclosures.

**Pricing:** Free. Funnel into Intruder's paid scanner.

**Target audience:** Security practitioners who want a free CVE news feed — not a prioritisation or decision-support tool.

**Gap for VulnBrief:** CVEMon tells you *what happened* (KEV added, exploit found). VulnBrief tells you *what it means for your environment and what to do*. Completely different jobs.

**Interesting feature — Hypemeter:** Social media trending score is a legitimate supplementary signal. High Hypemeter = researcher/attacker attention is spiking, independent of official EPSS. Worth watching as a future data source for VulnBrief's threat freshness signal.

---

### Feedly CVE

**What it does:**
- CVE intelligence layer built on top of Feedly's content aggregation platform
- **CVE detail page:** CVSS gauge + EPSS (shown as % alongside CVSS) + severity badge + status labels (Trending / Proof of exploit / Proof of concept / Exploited in the wild)
- **EUVD cross-reference:** Shows European Vulnerability Database ID alongside CVE ID — rare in the market
- **AI-generated CVSS estimate:** Before NVD publishes an official score, Feedly estimates CVSS severity from article content, attack complexity, and exploit information. Timestamps the estimate.
- **AI-generated Summary + Impact:** Plain-language summary of what the CVE does, who it affects, and attack impact. Names active campaigns (e.g., "RedSun/BlueHammer campaign") when detected in the wild.
- **Exploitation section:** Aggregates PoC sources — GitHub repos, security blogs, researcher posts. Named sources shown.
- **CVE Timeline:** Chronological event log per CVE:
  - First article mentioning the CVE
  - CVSS estimate (Feedly's own AI estimate)
  - CVSS publication (official NVD)
  - Scanner detection added (e.g., "Detection added to Qualys QID 92373")
  - KEV addition
  - First PoC/exploit disclosure
- **Trending Vulnerabilities homepage:** CVE cards with sparkline activity charts (media mention volume over time)
- **Track Updates:** Email watchlist alerting per CVE

**CVE prioritization:** CVSS-led, EPSS shown (but secondary — small text beneath CVSS gauge, not the lead signal). No composite scoring. No pre-conditions.

**Pricing:** Freemium. CVE detail pages appear accessible without login. Full watchlist and feed features behind Feedly Pro/Team subscription.

**Target audience:** Threat intel teams, CTI analysts, security researchers. Content-aggregation heritage — very strong on *news and narrative* around a CVE.

**What VulnBrief should borrow from Feedly:**

| Feedly Feature | VulnBrief Version | Priority |
|---|---|---|
| CVE Timeline (events as ordered list) | Show KEV addition date, first PoC date, NVD publish date on CVE detail | v2.0 |
| Scanner detection events | "Detection available in Qualys/Tenable" signal | v3.0 (Enterprise) |
| AI CVSS estimate pre-NVD | Not needed — we weight EPSS and KEV, not raw CVSS | — |
| Trending sparkline charts | Activity trend on dashboard / trending CVEs widget | v2.0 |
| Campaign name attribution | Where active exploitation is named (e.g., campaign names) | v3.0 |

**Where VulnBrief beats Feedly:**
- EPSS-first (Feedly buries it under CVSS)
- Pre-condition analysis (Feedly has none)
- Composite scoring (Feedly has none — it's still CVSS-led)
- Remediation brief (Feedly describes the problem, not the fix)
- SMB price point and operational focus (Feedly is CTI-analyst tool, not patch-decision tool)

---

### Wiz (UVM + ASM)

**What it does:**
- Unified Vulnerability Management (UVM): centralizes findings from external scanners + Wiz native scanners
- Attack Surface Management (ASM): agentless, continuous external asset discovery across cloud/AI/SaaS/on-prem/APIs
- Security Graph: links code → cloud infrastructure → runtime context
- Context-aware prioritization: EPSS + KEV + asset exposure + identity/permissions + data sensitivity + code reachability
- Attack path analysis: maps vulnerabilities to sensitive data access / privilege escalation paths
- Validates external exposure and what's actually loaded into memory (runtime signal)
- Shadow asset detection

**CVE prioritization:** EPSS threshold 0.2 (20%) warrants immediate attention. EPSS + KEV + asset context combined. Goes beyond global EPSS to environment-specific exploitability.

**Key insight from Wiz research:**
> 28.3% of exploited CVEs were weaponized within 1 day of publication (Q1 2025). Speed of prioritization matters as much as accuracy.

**Pricing:** Enterprise. Cloud-focused, not SMB.

**What VulnBrief should borrow from Wiz:**

| Wiz Capability | VulnBrief Version | Priority |
|---|---|---|
| Security Graph (code→cloud→runtime) | Asset Profile linking software → network exposure → auth | v2.0 |
| UVM aggregation layer | Ingest from Tenable/Qualys exports (Enterprise tier) | v3.0 |
| ASM (continuous external discovery) | Shodan/Censys passive lookup on CVE pages | v2.0 (planned) |
| Attack path visualization | React Flow chain map (already planned) | v2.0 |
| Compensating control context | Pre-condition wizard Q6 | v2.0 |
| Runtime signal | Out of scope — requires agent deployment | Post-v3.0 |

---

## Competitive Positioning Summary

| Dimension | VulnBrief | Nucleus | Tenable | NodeZero | Wiz | CVEMon | Feedly CVE |
|---|---|---|---|---|---|---|---|
| **EPSS-first?** | Yes | Yes (+custom) | VPR composite | No | Yes (+context) | No | No (CVSS-first) |
| **EPSS shown at all?** | Yes (lead) | Yes | Yes (input) | No | Yes | No | Yes (secondary) |
| **Pre-conditions?** | Yes (unique) | No | No | Implicit | Partially | No | No |
| **Plain-English layer?** | Yes (unique) | No | No | No | No | No | Yes (AI summary) |
| **Remediation guidance?** | Yes | Partial | Partial | No | No | No | No |
| **CVE Timeline/events?** | No (v2.0) | No | No | No | No | Partial | Yes (strong) |
| **Asset Profile matching?** | Planned v2.0 | Yes | Yes | Yes | Yes | No | No |
| **Attack chain?** | Planned v2.0 | No | No | Yes | Yes | No | No |
| **SMB price point?** | Yes ($49/mo) | No | No | No | No | Free | Freemium |
| **Open scoring?** | Yes | No | No | No | No | — | No |
| **Trending/social signal?** | No | No | No | No | No | Yes (Hypemeter) | Yes (sparklines) |

---

## Key Market Observations

**CVE volume crisis is real:** 40,000+ CVEs in 2024, 47,000+ projected 2025. Organizations remediate ~10% monthly. Prioritization is existential.

**EPSS is mainstream but misused:** All major platforms have integrated EPSS v3/v4. But most treat it as one signal among many, not as the primary triage signal. VulnBrief's EPSS-first approach is still differentiated.

**The asset context gap is the next battleground:** Global EPSS scores are useful but what matters is whether YOUR assets are reachable and what they can access. Wiz's Security Graph is the best current answer — but it's cloud-only and enterprise-priced.

**Pre-conditions are completely unaddressed in the market.** Every platform can tell you a CVE exists. None of them answer "does this CVE apply to YOUR environment given your specific configuration?" — except VulnBrief's pre-condition wizard. This is the genuine moat.

**Funding signal:** 2025 was the strongest cybersecurity funding year since 2021 ($13.97B). AI+security intersection is the dominant investment thesis. Astelia raised $35M Series B (2025) for AI vulnerability management. VulnBrief's angle (CVE-driven, CMDB-anchored, AI pre-conditions) has a strong funding narrative.

---

## Roadmap Additions Driven by This Research

### Immediate (v2.0 additions)

**1. Asset Profile Module**
Users define their environment once:
- Software inventory (name, version, vendor) — SBOM-importable
- Network exposure tiers (internet-facing / internal / air-gapped)
- Auth configuration (SSO, password, MFA status, default creds in use)
- OS family and version

CVEs are automatically cross-referenced against the profile. Risk score becomes personalized.

**2. Compensating Controls question in Pre-Condition Wizard**
Add Q6: "Do you have compensating controls for this attack vector?" (WAF / EDR / network segmentation / MFA)
This is what Cymulate validates at enterprise scale. VulnBrief can ask it as a questionnaire question.

**3. Exploit Maturity Expansion**
Beyond GitHub PoC refs, add:
- Metasploit module availability check
- Nuclei template availability check
- ExploitDB entry presence
- "Time to weaponization" estimate based on Wiz's 1-day data

**4. Threat Intel Freshness Signal**
Show when PoC activity was last detected, not just whether it exists. Wiz Q1 2025 data: 28.3% of exploited CVEs weaponized within 24 hours. Recency matters.

**5. CVE Lifecycle Timeline (from Feedly)**
Display a chronological event log on each CVE detail page:
- NVD published date
- First PoC/exploit disclosure date
- KEV addition date
- Scanner detection added (Qualys/Tenable plugin ID)
This converts static CVE data into a narrative: "this CVE was published 3 days ago, a PoC appeared yesterday, it hit KEV today." Context that changes the urgency signal immediately.

### Medium-term (v3.0)

**5. VulnBrief Environment Graph**
Simplified Security Graph:
- Nodes: Assets → Software → CVEs → Attack Paths
- Edges: "runs" / "affected by" / "chains to" / "exposes"
- React Flow visualization (already planned)
- Not as deep as Wiz but usable for SMBs without a full CMDB

**6. UVM Aggregation Layer (Enterprise tier)**
Ingest from Tenable, Qualys, Rapid7 CSV/API exports.
Normalize findings and apply VulnBrief composite scoring as the ranking layer.
This positions VulnBrief as the intelligence layer on top of existing scanners.

**7. Chain Context as Graph (not just text)**
`chain_context` currently returns a text paragraph (Claude-generated).
v3.0: structured `chain_nodes[]` and `chain_edges[]` — proper graph for React Flow.
Include non-CVE steps (misconfigurations, identity issues) based on NodeZero insight.

---

## Related

- [[Project Glasswing]] — the research program that feeds product decisions
- [[VulnBrief - Product Plan]] — the master roadmap this analysis updates
- [[VulnBrief - Intel Sources]] — data sources feeding the platform
- [[Vulnerability Prioritisation]] — the triage framework this platform automates
