---
tags: [research, threat-intel, openai, glasswing, competitive-context, defender-tools]
created: 2026-04-18
source: OpenAI Blog
author: OpenAI
---

# Reading Note: OpenAI Trusted Access for Cyber + GPT-5.4-Cyber

> OpenAI, April 2026 (response to Anthropic Mythos Preview)

---

## What This Is

OpenAI's strategic response to the Mythos Preview moment. While Anthropic launched Project Glasswing (controlled, partner-gated access to Mythos for defenders of critical software), OpenAI is going broader and faster with a tiered identity-verified access model.

---

## The TAC Program: What It Does

**Trusted Access for Cyber (TAC)** — scaling from hundreds to thousands of verified defenders:

| Tier | Access | Verification |
|---|---|---|
| Individual | Reduced safeguard friction on dual-use cyber tasks | KYC identity verification at chatgpt.com/cyber |
| Enterprise | Team access via OpenAI rep | Organisation-level request |
| Highest tier | GPT-5.4-Cyber (more permissive, purpose-fine-tuned) | Full vetting — limited, iterative deployment |

**Identity verification → access expansion**, not manual decisions. OpenAI explicitly frames this as the only scalable approach: "We don't think it's practical or appropriate to centrally decide who gets to defend themselves."

---

## GPT-5.4-Cyber: Key Capabilities

- Fine-tuned variant of GPT-5.4 specifically for cybersecurity
- **Lower refusal boundary** for legitimate cybersecurity work
- **Binary reverse engineering** — analyze compiled software (closed-source) for malware potential and vulnerabilities without source code access (this is the new capability vs. standard GPT-5.4)
- Classified as "high" cyber capability under OpenAI's Preparedness Framework
- Comes with **restrictions on Zero-Data Retention** — OpenAI needs visibility into use

This directly mirrors Mythos Preview's closed-source reverse engineering capability from the Anthropic article.

---

## Codex Security (Already in Production)

- Automatically monitors codebases, validates issues, proposes fixes
- 3,000+ critical/high vulnerabilities fixed since launch
- Free for open source projects via "Codex for Open Source" (1,000+ projects)
- Private beta six months ago, research preview earlier this year

This is OpenAI's equivalent of Anthropic's Glasswing — but deployed as a product, not a research programme.

---

## Three Strategic Principles

**1. Democratized access**
Make tools as widely available as possible. Avoid arbitrary gatekeeping. Use objective criteria (KYC, identity verification) to determine access. Automate over time.

**2. Iterative deployment**
Learn by putting systems into the world. GPT-5.2 → cyber safety training. GPT-5.3-Codex → additional safeguards. GPT-5.4 → "high" cyber rating. Each model improves on the last.

**3. Ecosystem resilience**
$10M Cybersecurity Grant Program, Codex for Open Source (free scanning), TAC programme. Investment in the defensive community at scale.

---

## The Key Strategic Contrast: Anthropic vs OpenAI

| Dimension | Anthropic (Glasswing) | OpenAI (TAC) |
|---|---|---|
| Access model | Partner-gated, critical infrastructure first | Identity-verified tiers, broader access |
| Primary tool | Mythos Preview (not generally available) | GPT-5.4-Cyber (tiered access, TAC) |
| Defender tooling | Project Glasswing (research programme) | Codex Security (product already deployed) |
| Open source | Coordinated disclosure of bugs found | Codex for Open Source (free scanning) |
| Philosophy | Controlled deployment, transitional period | Democratise access, scale with KYC |

Neither is wrong. Anthropic is more cautious about the weapons-grade capabilities. OpenAI is more focused on the access infrastructure. Both are responding to the same threat curve.

---

## What This Means for the Industry

**Both major labs are now in the defensive AI space at scale.** The question for every security team shifts from "should we use AI for security?" to "which access programme do we join and what workflow do we build around it?"

The operational gap these programmes create:
- Mythos/GPT-5.4-Cyber = **capability layer** (find and exploit vulnerabilities)
- What's missing = **triage and prioritisation layer** (which of the thousand bugs found should we fix first, in what order, given our specific environment)

This is VulnBrief's lane. Neither Glasswing nor TAC solves the prioritisation problem. They produce more findings faster. VulnBrief tells you what those findings mean for your environment.

---

## The Dual-Use Framing (Critical for VulnBrief Pitch)

> "Cyber capabilities are inherently dual-use, so risk isn't defined by the model alone. It also depends on the user, the trust signals around them, and the level of access they're given."

This is the regulatory and institutional consensus forming. AI-assisted security tools are going to be broadly available. The question is verified defenders having access before (or at least simultaneously with) unverified attackers.

VulnBrief's TAC angle: as a verified defensive tool, VulnBrief could be positioned as complementary to TAC — the prioritisation layer that makes the output of GPT-5.4-Cyber/Mythos actionable at the team level.

---

## Implications for VulnBrief

### Urgency confirmed
Both labs are scaling defender access now. The window where defenders are ahead of (or even at parity with) attackers is short. VulnBrief's value proposition — faster triage, better prioritisation, pre-condition analysis — becomes more important as the volume of AI-discovered findings increases.

### Potential partnership/integration angle
- **Codex Security output → VulnBrief input:** If Codex Security flags CVEs in your codebase, VulnBrief scores and prioritises them with EPSS/KEV/chain context.
- **TAC programme listing:** VulnBrief could apply to be a TAC partner — giving verified VulnBrief users reduced friction on cyber queries.

### CVE Chain Index timing
With GPT-5.4-Cyber now doing binary reverse engineering and N-day weaponisation, the chain index feature is not a Phase 2 nice-to-have — it's the immediate defensive response. CVE chains documented today prevent chain-assisted exploitation tomorrow.

### Product positioning update
Frame VulnBrief not just as "CVE prioritisation" but as the **triage and decision layer** between AI-discovered findings (Glasswing, Codex Security, GPT-5.4-Cyber) and the human security team that has to decide what gets patched in what order.

> "AI finds thousands of bugs. VulnBrief tells you which 10 to fix this week."

---

## Key Quotes

> "Defenses should be continually scaled with capability. As model capabilities increase, defenses need to scale alongside them."

> "We don't think it's practical or appropriate to centrally decide who gets to defend themselves."

> "The strongest ecosystem is one that continuously identifies, validates, and fixes security issues as software is written."

---

## Related

- [[Reading - Anthropic Mythos Preview Cybersecurity Assessment]] — the capability announcement this responds to
- [[Project Glasswing]] — Anthropic's defensive programme
- [[VulnBrief - Product Plan]] — triage/prioritisation layer positioning
- [[VulnBrief - CVE Chain Index Blueprint]] — now more urgent given both labs scaling capability
- [[VulnBrief - Competitive Intelligence]] — neither Glasswing nor TAC is a VulnBrief competitor; they're the upstream
