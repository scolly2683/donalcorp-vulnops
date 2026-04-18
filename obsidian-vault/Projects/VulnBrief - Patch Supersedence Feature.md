---
tags: [project, vulnbrief, feature, phase-2, microsoft, supersedence, windows]
created: 2026-04-18
status: idea
---

# VulnBrief — Patch Supersedence Feature

## The Problem

A CVE fixed by KB5032189 (November) may already be covered by KB5033375 (December cumulative update). Without supersedence awareness, VulnBrief flags the CVE as unpatched even if the environment already has the later rollup installed. This creates noise and erodes trust.

Supersedence information answers: **"Do I actually need to apply this patch, or is it already covered by something newer I've deployed?"**

---

## Data Source: MSRC Security Update Guide API

Free, public, no auth required for most endpoints.

**Monthly CVRF data:**
```
GET https://api.msrc.microsoft.com/cvrf/v3.0/cvrf/{year}-{month}
e.g. https://api.msrc.microsoft.com/cvrf/v3.0/cvrf/2026-Apr
```

Returns JSON with:
- CVE → KB mapping (which patch fixes which CVE)
- `Supersedes` / `SupersededBy` fields per KB
- `RevisionHistory` showing patch evolution
- Severity, affected products, CVSS

**Update summary endpoint (lighter):**
```
GET https://api.msrc.microsoft.com/updates
```
Returns list of available updates with supersedence chains.

**Limitations:** No data prior to 2016.

---

## What to Ingest

### Table: `patch_supersedence`
```sql
CREATE TABLE patch_supersedence (
    id              SERIAL PRIMARY KEY,
    kb_id           VARCHAR(20) NOT NULL,         -- e.g. KB5032189
    superseded_by   VARCHAR(20) NOT NULL,         -- e.g. KB5033375
    product         VARCHAR(200),                 -- e.g. Windows 11 22H2
    source_month    VARCHAR(7) NOT NULL,           -- e.g. 2026-Apr
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(kb_id, superseded_by, product)
);
```

### Table: `cve_patches`
```sql
CREATE TABLE cve_patches (
    id          SERIAL PRIMARY KEY,
    cve_id      VARCHAR(20) REFERENCES cves(id),
    kb_id       VARCHAR(20) NOT NULL,
    product     VARCHAR(200),
    severity    VARCHAR(20),
    source_month VARCHAR(7),
    UNIQUE(cve_id, kb_id, product)
);
```

---

## How It Changes VulnBrief

### 1. Patch status context on CVE detail page
For any Windows CVE, show:
```
Fix available: KB5032189 (Nov 2026)
Superseded by: KB5033375 (Dec 2026) — install this instead
```

### 2. "Already covered" detection (Asset Profile phase)
When a user's asset profile includes their current patch level (e.g. "December 2026 cumulative installed"), VulnBrief can:
- Mark CVEs fixed by superseded KBs as "covered by your current patch"
- Remove them from the active remediation queue automatically
- This is the Windows equivalent of "version not affected"

### 3. Remediation brief improvement
Current remediation_brief is Claude-generated text. With supersedence data, it becomes:
> "Apply KB5033375 (December 2026 cumulative update). This supersedes KB5032189 and KB5031455, covering this CVE and 47 others in the December rollup."

### 4. Chain link enrichment
Multiple CVEs fixed by the same KB = natural chain candidates (same attack surface, same patch).
→ Auto-create chain links for CVEs sharing a KB, with `source_type = "msrc_advisory"` and `confidence = "medium"`.

---

## Supersedence as a Composite Score Modifier

If a CVE's fix KB has been superseded and the user has the superseding KB:
- Risk score drops to near 0 (already patched)
- Flag: "Covered by your current patch level"

If the CVE's fix KB has been superseded but the user hasn't applied the new KB either:
- Risk score stays high
- Remediation brief points to the newer cumulative, not the original KB

---

## MSRC API — Additional Signals Available

| Field | VulnBrief use |
|---|---|
| `AffectedProducts` | Auto-tag CVEs as Windows-specific |
| `CVSS` | Cross-check against NVD CVSS |
| `Severity` | Microsoft's own severity (sometimes differs from NVD) |
| `ExploitabilityIndex` | Microsoft's "Exploited / More Likely / Less Likely" — useful signal |
| `RevisionHistory` | Track when Microsoft changed a CVE's severity |

**ExploitabilityIndex** is particularly valuable — Microsoft independently rates exploitation likelihood as:
- Exploitation Detected
- Exploitation More Likely
- Exploitation Less Likely
- Not Applicable

This is a Microsoft-sourced equivalent of KEV / EPSS and could be a 5th composite score component for Windows CVEs.

---

## Build Complexity

| Component | Complexity | Notes |
|---|---|---|
| MSRC ingest (monthly CVRF) | Low-Medium | HTTP + JSON parse, no auth |
| `cve_patches` table | Low | Simple FK to cves |
| `patch_supersedence` table | Low | Self-referential KB links |
| CVE detail: patch + supersedence display | Low | New UI section |
| ExploitabilityIndex as composite signal | Medium | Conditional on product type |
| Asset profile patch level check | Medium | Needs asset profile first (v2.0) |

**MVP:** MSRC ingest + `cve_patches` + `patch_supersedence` + display on CVE detail. 2–3 days.
**Full feature:** Asset profile patch level matching. Depends on Phase 2 asset profile.

---

## Other Vendors with Supersedence Data

| Vendor | Source | Notes |
|---|---|---|
| Microsoft | MSRC API (CVRF) | Best structured data |
| Red Hat | RHSA advisories | Errata supersedence, well documented |
| Ubuntu | USN advisories | Package version replacement |
| Debian | DSA advisories | Similar pattern |
| Oracle | ELSA advisories | RHEL-compatible |

Non-Microsoft supersedence = "this package version supersedes this older version." Same concept, different framing. Can be inferred from OSV `fixed_versions` data (already ingested).

---

## Related

- [[VulnBrief - Product Plan]] — Phase 2 features
- [[VulnBrief - CVE Chain Index Blueprint]] — MSRC data enriches chain links
- [[VulnBrief - Competitive Intelligence]] — no free tool surfaces supersedence context per CVE
