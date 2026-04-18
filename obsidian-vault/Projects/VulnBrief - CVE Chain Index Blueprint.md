---
tags: [project, vulnbrief, blueprint, feature, phase-2, exploit-chaining]
created: 2026-04-18
status: blueprint
---

# VulnBrief — CVE Chain Index: Feature Blueprint

> **Context:** Anthropic's Mythos Preview can take a CVE ID + git commit hash and produce a working exploit in hours for <$2,000, autonomously chaining 2–4 vulnerabilities. VulnBrief's defensive response is to surface known CVE chains so defenders prioritise patch sets, not individual CVEs. This blueprint uses Mythos's own documented chain taxonomy as the data model.

---

## The Problem

Current vulnerability management — including VulnBrief today — scores CVEs individually. But real attacks chain vulnerabilities:

- KASLR bypass (EPSS 0.1, "Medium") + write primitive (EPSS 0.08, "Low") = root on Linux
- SSRF (EPSS 0.15) + metadata endpoint exposure = cloud account takeover
- Auth bypass (EPSS 0.2) + exposed admin endpoint = RCE

A defender who patches either CVE in isolation remains fully compromised. The risk only disappears when the chain is broken.

---

## Chain Role Taxonomy

Derived from Mythos Preview's documented exploitation patterns:

| Role | Description | Example CVEs |
|---|---|---|
| `initial_access` | Entry point — internet-facing, pre-auth | Apache ActiveMQ RCE, Citrix Bleed |
| `info_leak` | Discloses kernel/heap/ASLR addresses needed for next step | KASLR bypasses, pointer leaks |
| `write_primitive` | Out-of-bounds write, UAF write — controllable memory corruption | Most kernel OOB write CVEs |
| `privilege_escalation` | Moves from low-priv user to root/SYSTEM | PwnKit, DirtyPipe, PrintNightmare |
| `lateral_movement` | Pivots from one host/segment to another | SMB vulns, credential relay |
| `persistence` | Maintains access after reboot/logout | Bootkit CVEs, scheduled task abuse |
| `defence_evasion` | Disables or bypasses detection | AV/EDR bypass CVEs |
| `exfiltration` | Extracts data | Data exposure APIs, S3 misconfig chains |
| `sandbox_escape` | Breaks out of VM/container/browser renderer | VMM CVEs, browser sandbox escapes |

Every CVE gets one primary role. Some get two (e.g., an info_leak that also enables write via type confusion).

---

## Database Schema

```sql
-- New table
CREATE TABLE cve_chain_links (
    id              SERIAL PRIMARY KEY,
    source_cve_id   VARCHAR(20) NOT NULL REFERENCES cves(id),
    linked_cve_id   VARCHAR(20) NOT NULL REFERENCES cves(id),
    
    -- Role of linked_cve_id in the chain (what does it enable?)
    chain_role      VARCHAR(30) NOT NULL,
    -- e.g. "initial_access", "info_leak", "write_primitive", 
    --      "privilege_escalation", "lateral_movement", 
    --      "persistence", "defence_evasion", "sandbox_escape"
    
    -- Direction: does source_cve enable linked_cve, or vice versa?
    direction       VARCHAR(10) NOT NULL DEFAULT 'forward',
    -- 'forward'  = source → linked (source creates foothold, linked escalates)
    -- 'backward' = linked → source (linked creates foothold, source escalates)
    
    confidence      VARCHAR(10) NOT NULL DEFAULT 'medium',
    -- 'low'    = theoretical / inferred from technique similarity
    -- 'medium' = referenced in threat intel or advisory
    -- 'high'   = documented in IR report / Anthropic / vendor advisory
    
    source_type     VARCHAR(30) NOT NULL,
    -- 'nvd_reference'     = linked_cve_id appears in source_cve's NVD references field
    -- 'cisa_advisory'     = CISA joint advisory groups these CVEs
    -- 'dfir_report'       = documented in DFIR Report incident
    -- 'vendor_advisory'   = vendor security advisory chains them
    -- 'mitre_attack'      = same ATT&CK technique, overlapping CVE lists
    -- 'manual'            = manually curated from threat intel
    
    source_ref      TEXT,          -- URL or document title/ID
    campaign_name   VARCHAR(200),  -- e.g. "ALPHV Mar 2026", "LockBit 3.0"
    
    created_at      TIMESTAMPTZ DEFAULT NOW(),
    
    UNIQUE (source_cve_id, linked_cve_id, chain_role, source_type)
);

-- Index for API queries
CREATE INDEX idx_chain_source ON cve_chain_links(source_cve_id);
CREATE INDEX idx_chain_linked ON cve_chain_links(linked_cve_id);

-- Optional: named chains (a campaign may chain 3+ CVEs)
CREATE TABLE cve_chain_campaigns (
    id              SERIAL PRIMARY KEY,
    name            VARCHAR(200) NOT NULL UNIQUE,
    description     TEXT,
    source_ref      TEXT,
    first_seen      DATE,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Junction: campaign → ordered chain steps
CREATE TABLE cve_chain_campaign_steps (
    campaign_id     INTEGER REFERENCES cve_chain_campaigns(id),
    step_order      INTEGER NOT NULL,
    cve_id          VARCHAR(20) REFERENCES cves(id),
    chain_role      VARCHAR(30),
    PRIMARY KEY (campaign_id, step_order)
);
```

SQLAlchemy model (`app/models/chain.py`):
```python
class CVEChainLink(Base):
    __tablename__ = "cve_chain_links"
    id = mapped_column(Integer, primary_key=True)
    source_cve_id = mapped_column(String(20), ForeignKey("cves.id"), nullable=False)
    linked_cve_id = mapped_column(String(20), ForeignKey("cves.id"), nullable=False)
    chain_role = mapped_column(String(30), nullable=False)
    direction = mapped_column(String(10), default="forward")
    confidence = mapped_column(String(10), default="medium")
    source_type = mapped_column(String(30), nullable=False)
    source_ref = mapped_column(Text)
    campaign_name = mapped_column(String(200))
    created_at = mapped_column(DateTime(timezone=True), server_default=func.now())
```

---

## Automated Ingest

### Source 1: NVD References Field (Immediate, High Coverage)

NVD CVE records include a `references` array. Many descriptions also mention other CVE IDs inline. Parse both:

```python
import re

CVE_PATTERN = re.compile(r'CVE-\d{4}-\d{4,7}')

def extract_cve_references(cve_data: dict) -> list[str]:
    refs = []
    
    # From references URLs and tags
    for ref in cve_data.get("references", []):
        found = CVE_PATTERN.findall(ref.get("url", "") + " " + ref.get("source", ""))
        refs.extend(found)
    
    # From description text
    for desc in cve_data.get("descriptions", []):
        if desc.get("lang") == "en":
            refs.extend(CVE_PATTERN.findall(desc.get("value", "")))
    
    return list(set(refs))
```

When CVE-A's NVD record references CVE-B, create a `cve_chain_links` row:
- `source_cve_id` = CVE-A
- `linked_cve_id` = CVE-B
- `confidence` = "low" (reference ≠ confirmed chain)
- `source_type` = "nvd_reference"

**Estimated yield:** Thousands of links. Confidence low but a useful baseline graph.

### Source 2: CISA KEV Campaign Grouping (Medium Confidence)

CISA KEV JSON includes `knownRansomwareCampaignUse` field. Group KEV entries by:
- Same campaign name → all CVEs in that campaign are chain candidates
- Same date cluster (±7 days) within same ransomware family → same incident

```python
# From CISA KEV JSON
for vuln in kev_data["vulnerabilities"]:
    campaign = vuln.get("knownRansomwareCampaignUse")
    if campaign and campaign != "Unknown":
        # Link all CVEs in this campaign to each other
        # chain_role = infer from CVE description (auth bypass = initial_access, etc.)
        # confidence = "medium"
        # source_type = "cisa_advisory"
```

**Estimated yield:** Hundreds of high-value links (KEV = confirmed exploited).

### Source 3: MITRE ATT&CK CVE→Technique Mapping (Low Confidence, High Coverage)

MITRE publishes STIX bundles mapping CVEs to ATT&CK techniques. Download from:
`https://github.com/mitre-attack/attack-stix-data`

CVEs mapped to the same technique sequence (e.g., T1190 Exploit Public-Facing App → T1068 Privilege Escalation) are chain candidates:

```python
# MITRE technique → CVE list
# T1190 (initial access) CVEs + T1068 (privilege escalation) CVEs
# = candidates for initial_access → privilege_escalation chains
# confidence = "low", source_type = "mitre_attack"
```

### Source 4: Manual Curated (High Confidence, Low Volume)

Admin POST endpoint for security team to add confirmed chains from DFIR Report, vendor advisories, Anthropic Glasswing disclosures:

```
POST /admin/chain-links
{
  "source_cve_id": "CVE-2026-34197",
  "linked_cve_id": "CVE-2024-1709",
  "chain_role": "initial_access",
  "direction": "forward",
  "confidence": "high",
  "source_type": "dfir_report",
  "source_ref": "https://thedfirreport.com/2026/04/01/...",
  "campaign_name": "ALPHV Apr 2026"
}
```

**Estimated yield:** 50–200 high-confidence links per quarter (manual effort).

---

## API Endpoints

### GET /cve/{id}/chains

```json
{
  "cve_id": "CVE-2026-34197",
  "as_entry_point": [
    {
      "linked_cve_id": "CVE-2024-1709",
      "chain_role": "initial_access",
      "confidence": "high",
      "campaign_name": "ALPHV Apr 2026",
      "source_ref": "https://thedfirreport.com/..."
    }
  ],
  "as_escalation_step": [
    {
      "linked_cve_id": "CVE-2021-4034",
      "chain_role": "privilege_escalation",
      "confidence": "medium",
      "source_ref": "https://www.cisa.gov/..."
    }
  ],
  "campaigns": [
    {
      "name": "ALPHV Apr 2026",
      "steps": [
        {"step": 1, "cve_id": "CVE-2024-1709", "role": "initial_access"},
        {"step": 2, "cve_id": "CVE-2026-34197", "role": "privilege_escalation"},
        {"step": 3, "cve_id": "CVE-2023-4966", "role": "lateral_movement"}
      ]
    }
  ]
}
```

### GET /chains/campaigns

List all named campaigns with their CVE steps. Used for the "Trending Attack Chains" dashboard widget.

---

## Scoring Impact: Chain Completion Risk

When a user has an Asset Profile, VulnBrief can detect chain overlap:

```python
def chain_completion_multiplier(cve_id: str, user_asset_cves: list[str]) -> float:
    """
    If this CVE is part of a chain, and the user also has the 
    other CVEs in that chain, multiply composite score.
    """
    chain_links = get_chain_links(cve_id)
    
    for link in chain_links:
        if link.linked_cve_id in user_asset_cves:
            # Chain is completable in this environment
            if link.confidence == "high":
                return 1.5   # 50% score boost
            elif link.confidence == "medium":
                return 1.25  # 25% score boost
    
    return 1.0  # no chain overlap detected
```

Dashboard alert: *"CVE-2026-34197 + CVE-2021-4034 are a documented privilege escalation chain. Both are present in your environment. Combined risk: Critical."*

---

## Frontend: CVE Detail Page Addition

New section below the pre-condition wizard:

```
┌─────────────────────────────────────────────────────┐
│  Known Attack Chains                                 │
│                                                      │
│  This CVE has been chained with:                     │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │ 🔴 ALPHV Apr 2026 Campaign          [HIGH]   │   │
│  │ CVE-2024-1709 → [THIS CVE] → CVE-2023-4966   │   │
│  │ Initial Access → Escalation → Lateral Move   │   │
│  │ Source: DFIR Report Apr 2026          [Link]  │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │ 🟡 Also commonly preceded by:      [MEDIUM]  │   │
│  │ CVE-2024-1709 (Auth bypass)                  │   │
│  │ Source: CISA Advisory AA26-089A     [Link]   │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## What NOT to Build in Phase 2

- **Automated chain discovery from code analysis** — that's Mythos. Out of scope.
- **Full attack graph visualisation** — React Flow graph is Phase 3.
- **DFIR Report automated parsing** — DFIR Report doesn't have a structured API. Manual curation only for now.
- **Chain path length > 4 hops** — beyond 4 CVEs the data quality degrades. Cap at 4 in Phase 2.

---

## Complexity Estimates

| Component | Complexity | Notes |
|---|---|---|
| DB schema + migration | Low | 3 tables, standard FK pattern |
| NVD reference parser | Low | Regex + run on existing NVD data |
| CISA KEV chain grouping | Low | KEV JSON already ingested |
| MITRE ATT&CK ingest | Medium | STIX parsing, data normalisation |
| Manual admin endpoint | Low | Simple POST with auth |
| GET /cve/{id}/chains | Low | Standard SQLAlchemy query |
| Score multiplier | Low | Add to existing composite logic |
| Frontend chain section | Medium | New component, campaign rendering |
| Dashboard "chain alerts" widget | Medium | Requires asset profile to be built first |

**Total:** 2–3 sprint weeks (solo developer). NVD reference parser + CISA KEV grouping can ship in week 1 as an MVP.

---

## Phase 2 MVP Sequence

1. **Week 1:** DB migration + NVD reference parser + backfill existing CVEs → chain graph with low-confidence links from NVD
2. **Week 1–2:** CISA KEV campaign grouping → medium-confidence links; manual admin POST endpoint
3. **Week 2:** GET /cve/{id}/chains API + frontend "Known Attack Chains" section
4. **Week 3:** Composite score chain multiplier; dashboard chain completion alert

---

## Related

- [[VulnBrief - Product Plan]] — Phase 2 features, chain context
- [[Reading - Anthropic Mythos Preview Cybersecurity Assessment]] — primary motivation for this feature
- [[Reading - Breaking the Web Vulnerability Chaining]] — chain taxonomy background
- [[VulnBrief - Competitive Intelligence]] — no competitor currently does chain-aware scoring
