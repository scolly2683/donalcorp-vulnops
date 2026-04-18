---
tags: [project, management, pitch]
created: 2026-04-18
---

# VulnBrief — Vulnerability Intelligence That Tells You What To Fix First

---

## The Problem

Every week, hundreds of new vulnerabilities are published. Security teams are expected to assess, prioritise, and remediate them — but the raw data is technical, overwhelming, and designed for researchers, not operators.

The standard industry score (CVSS) ranks vulnerabilities by theoretical severity. It does not tell you whether a vulnerability is being actively exploited. It does not tell you whether your specific environment is affected. It does not tell you what to do about it in plain language.

The result: teams either treat everything as urgent (and burn out), or they rely on gut instinct (and miss things that matter).

---

## What We Are Building

A vulnerability intelligence platform that answers the three questions every security team actually needs answered:

**1. What is being exploited right now — not just what is theoretically severe?**
We prioritise using real-world exploitation probability, not just severity scores. A vulnerability rated 9.8 out of 10 that no attacker has ever used is less urgent than a 6.5 that is being actively weaponised today. We surface the difference.

**2. Does this vulnerability actually affect us?**
Most tools tell you a vulnerability exists. None of them tell you whether your specific configuration, software version, network position, or authentication setup makes you a viable target. We do. A short questionnaire maps the pre-conditions of each vulnerability to your environment and returns a clear verdict: affected, unlikely affected, or investigate further.

**3. What do we do about it — and who can understand the answer?**
Every vulnerability in our platform has two views. One for the technical team: the full detail, the attack vector, the exploit context. One for leadership and non-technical stakeholders: plain English, what it means for the business, and what action is required. Same data. Two audiences. No translation work.

---

## The Core Advantages

**We prioritise by what attackers are doing, not what vendors say is severe.**
We incorporate exploitation probability data, confirmed-exploited catalogues, and public proof-of-concept availability into a single composite score. Teams act on the right things first.

**We close the gap between a CVE and your environment.**
The market has no affordable tool that answers "are we actually affected?" at the individual vulnerability level. We do this through a structured pre-condition analysis — specific questions about your software versions, network exposure, authentication, and configuration — and return a verdict, not a list.

**We serve both the security team and the boardroom from one platform.**
The technical team gets depth. Leadership gets clarity. There is no separate reporting layer, no manual translation, no slide deck required to explain what is happening.

**We surface what attackers chain together, not just individual weaknesses.**
Many serious attacks are not single vulnerabilities — they are sequences. A vulnerability on its own may be low risk. Combined with a misconfiguration or a second weakness in the same environment, it becomes a critical path to breach. We map these chains and present the real blast radius.

**We are designed for teams that cannot afford to operate at enterprise scale.**
The tools that do parts of what we do cost tens of thousands of pounds per year and require dedicated analysts to operate. We are built to be accessible, actionable, and affordable — without sacrificing the quality of the intelligence.

---

## What This Means In Practice

A new critical vulnerability is published on a Monday morning. Within hours:

- It is ingested, scored, and prioritised against everything else in the queue
- The technical team sees the full detail, the exploit context, and whether a working exploit exists in the wild
- Leadership sees a plain-English summary: what it affects, what the business risk is, and what action is recommended
- Any team member can run a five-question check to confirm whether their environment is actually in scope
- If it chains with other known weaknesses in the environment, that is surfaced automatically

No manual triage. No translation meetings. No guesswork about whether to act.

---

## The Gap We Are Filling

The enterprise tools that address parts of this problem cost $50,000 to $500,000 per year and require significant operational overhead to run. The free tools — NVD, CISA, EPSS feeds — are raw data with no intelligence layer on top.

Nothing in between serves the teams that need it most.

That is what we are building.
