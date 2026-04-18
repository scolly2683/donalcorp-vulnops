---
tags: [project, startup, product, vulnerability-intelligence, glasswing]
created: 2026-04-17
updated: 2026-04-18
status: active — v1 deployed, chain index shipped
---

# VulnBrief — Vulnerability Intelligence for Everyone

> "Every tool explains what's vulnerable. None of them explain whether YOU are — in language you can actually act on."

A vulnerability intelligence platform that serves both technical practitioners and non-technical decision-makers from the same underlying data.

---

## The Problem

| Audience | Current reality |
|----------|----------------|
| Security engineers | Overwhelmed by raw feeds — CVSS noise, no prioritisation signal |
| CISOs / CTOs | Can't read existing tools — get briefed by their team, no independent view |
| SMB owners | Can't afford enterprise tools, can't interpret NVD/CISA raw data |
| Developers | Don't know if a CVE affects their specific stack without manual research |

No tool bridges these. That's the gap.

---

## Core Differentiators

1. **Two views, one data source** — every CVE has a Technical tab and a Plain English tab
2. **"Am I affected?" pre-condition questionnaire** — maps CVE pre-conditions to your environment in 3–5 questions
3. **Exploit chain context** — shows what initial access enables, what it chains to
4. **EPSS-first prioritisation** — surfaces what's actually being exploited, not just what scores high on CVSS
5. **Supply chain lens** — enter a package + version, get your exposure across npm/PyPI/Go/Maven

---

## What Is Live (v1 — deployed April 2026)

**Frontend:** https://vulnbrief.vercel.app
**Backend:** https://vulnbrief-api.fly.dev
**Database:** Fly.io Postgres (vulnbrief-db, London region)

### Shipped and working
- [x] CVE detail page: EPSS gauge, CVSS, KEV status, risk score, plain-English summary (Claude API), pre-conditions, exploit chain context, remediation brief, PoC references
- [x] Search by CVE ID, keyword
- [x] Dashboard: trending CVEs, KEV additions, stats
- [x] Pre-condition questionnaire (5-question wizard → LIKELY/POSSIBLY/UNLIKELY verdict)
- [x] Supply chain lookup: package + version → affected CVEs via OSV
- [x] Composite risk scoring: EPSS 35% / KEV 30% / CVSS 20% / PoC 15%
- [x] **CVE Chain Index** — `cve_chain_links` table, NVD reference parser, KEV campaign grouper, `GET /cves/{id}/chains` API, ChainLinks frontend component with confidence badges and campaign grouping
- [x] Auto-deploy: Vercel watches GitHub main branch

### Data seeded
- 1,569 CISA KEV entries
- 200k+ EPSS scores
- ~22,000 NVD CVEs (last 120 days)
- Chain links from NVD references (low confidence) + KEV campaigns (medium confidence)

---

## Next Features (Phase 2)

### Immediate build queue

**1. MSRC Patch Supersedence** ← high value for Windows shops
- Ingest MSRC CVRF API (monthly security updates)
- `cve_patches` table: CVE → KB mapping
- `patch_supersedence` table: KB → superseded_by KB
- Display on CVE detail: "Fix: KB5032189 — superseded by KB5033375"
- Microsoft ExploitabilityIndex as 5th composite score component for Windows CVEs
- See: [[VulnBrief - Patch Supersedence Feature]]

**2. CVE Lifecycle Timeline** ← from Feedly competitive analysis
- Show chronological event log per CVE: NVD published, first PoC date, KEV addition date, scanner detection
- Converts static data into narrative: "published 3 days ago, PoC appeared yesterday, KEV today"
- Data already in DB — mostly a frontend addition

**3. Exploit Maturity Expansion**
- Add Metasploit module availability check
- Add Nuclei template availability check
- ExploitDB entry presence
- "Time to weaponisation" estimate (Wiz: 28.3% weaponised within 24h of publication)

**4. Compensating Controls in Pre-Condition Wizard**
- Add Q6: "Do you have compensating controls for this vector?" (WAF / EDR / network segmentation / MFA)
- What Cymulate validates at enterprise scale; VulnBrief asks as a questionnaire question

**5. Shodan/Censys passive exposure lookup**
- On CVE detail page: "X internet-facing assets found running affected software version"
- No active scanning — passive lookup only
- Shodan API ($49/mo) or Censys free research tier

### Medium-term (Phase 2 continued)

- **Asset Profile Module** — users define software inventory, network exposure tier, auth config; CVEs auto-matched; risk score personalised
- **Chain Completion Risk** — detect when a user's asset profile contains multiple CVEs from the same documented chain; alert "chain completable in your environment"
- **Reachability scoring** — fourth dimension alongside EPSS/KEV/CVSS: is this CVE reachable given your network topology?
- **DFIR Report integration** — manual curation of chain links from real IR cases; high-confidence chain enrichment

### Phase 3 — Scanning (scale feature, high value)
Users enter their domain, IP range, or paste a dependency manifest. VulnBrief returns:
- **Passive exposure check** (Shodan/Censys/GreyNoise — no active probing): "We found 3 internet-facing services on your IP range matching vulnerable software versions"
- **Dependency scan** (OSV.dev match): "Your package.json includes axios 1.14.0 — affected by CVE-2026-XXXXX"
- **Active scan** (Nuclei templates, authorised targets only): full vulnerability detection with explicit consent gate

> **Scale note:** Scanning is the feature that converts free users to paid. Passive lookups are cheap to run at scale. Active scanning requires auth workflow, rate limiting, and legal terms. Build passive first, active as a paid-tier feature with explicit written consent capture.

---

## Data Sources

### Free, high-quality, API-accessible (use all of these)

| Source | Data | Update frequency | Notes |
|--------|------|-----------------|-------|
| CISA KEV | Confirmed exploited CVEs | Daily JSON feed | Authoritative — P0 by definition |
| NVD (NIST) | Full CVE detail, CVSS, CPE, references | Continuous REST API | Primary CVE data store |
| EPSS (First.org) | 30-day exploitation probability | Daily CSV + REST | Most important prioritisation signal |
| OSV.dev (Google) | Open source ecosystem vulns (npm, PyPI, Go, Maven, RubyGems, Rust) | Real-time REST API | Best supply chain source |
| GitHub Advisory DB | Package-level advisories with ecosystem context | GraphQL API | Complements OSV, more human-readable |
| ExploitDB | Public POC and exploit archive | Periodic scrape | POC availability signal |

### Infrastructure & Exposure Intelligence (post-MVP)

| Source | Data | Why it matters | Cost |
|--------|------|---------------|------|
| **Shodan** | Internet-exposed assets, vuln filters for IoT/ICS | Shows real exposure count for affected software versions | $49/mo API |
| **Censys** | Certificate transparency, shadow cloud instances | Catches assets Shodan misses — better for cloud exposure | Free research tier |
| **GreyNoise** | Active scanning signals — background noise vs targeted | Distinguishes "everyone is probing this" from "you are being targeted" | Free community tier |

### Threat Context (post-MVP)

| Source | Data | Why it matters | Cost |
|--------|------|---------------|------|
| **Hudson Rock** | Infostealer / stolen credential exposure | 2026 breaches increasingly start with stolen session cookies not technical exploits | Free lookup tier |
| **OpenCVE** | Customisable CVE alert subscriptions by vendor/product | Solves the firehose problem — only surface what matters to a specific stack | Free self-hosted |
| **CVECrowd** | Community trending signal | Practitioner attention = real-world relevance signal | Free |

### Offensive/Research Tools (inform the product, don't integrate directly)

| Tool | What it is | Relevance to VulnBrief |
|------|-----------|----------------------|
| **VulHunt (Binarly)** | AI-powered vulnerability detection in compiled software/firmware | Shows what AI-discovered vuln classes are emerging — informs pre-condition content |
| **BlacksmithAI** | Multi-agent AI automated pen testing framework | Surfaces how vulns are being chained in practice — exploit chain research input |
| **Betterleaks** | Leaked secrets detection (from Gitleaks creator) | Supply chain angle — secret leakage is a pre-condition for many attack chains |

### Skip for now
- VirusTotal GTI — expensive, IOC-focused, not core to this product's angle

### Claude API (core to the product)

Used for:
- Generating plain-English CVE summaries
- Translating pre-condition questionnaire answers into risk verdicts
- Writing the non-technical "what this means for your business" copy
- Drafting weekly digest emails

Model: Claude Sonnet (cost-efficient at scale). Opus for high-complexity chains only.

---

## Technical Architecture

```
┌─────────────────────────────────────────────┐
│              Ingestion Layer                 │
│  NVD  │  CISA KEV  │  EPSS  │  OSV  │  GH  │
└─────────────┬───────────────────────────────┘
              │ background jobs (daily/hourly)
┌─────────────▼───────────────────────────────┐
│           PostgreSQL                         │
│  cves | epss_scores | kev_entries |          │
│  osv_advisories | pocs | summaries           │
└─────────────┬───────────────────────────────┘
              │
┌─────────────▼───────────────────────────────┐
│           FastAPI (Python)                   │
│  /cve/{id}  /search  /supply-chain          │
│  /dashboard  /digest  /pre-condition         │
└─────────────┬───────────────────────────────┘
              │ Claude API (summaries)
┌─────────────▼───────────────────────────────┐
│           Next.js frontend                   │
│  Technical view  │  Plain English view        │
│  Pre-condition questionnaire                 │
│  Supply chain lookup                         │
│  Dashboard / trending                        │
└─────────────────────────────────────────────┘
```

---

## Files to Create

```
vulnbrief/
├── backend/
│   ├── main.py                    # FastAPI app
│   ├── models.py                  # SQLAlchemy models
│   ├── database.py                # DB connection
│   ├── ingest/
│   │   ├── nvd.py                 # NVD API ingestion
│   │   ├── kev.py                 # CISA KEV feed
│   │   ├── epss.py                # EPSS daily CSV
│   │   ├── osv.py                 # OSV.dev API
│   │   ├── github_advisory.py     # GitHub GraphQL
│   │   └── exploitdb.py           # ExploitDB scraper
│   ├── enrichment/
│   │   ├── summarise.py           # Claude API integration
│   │   ├── preconditions.py       # Pre-condition logic
│   │   └── chains.py              # Exploit chain mapping
│   ├── api/
│   │   ├── cves.py                # /cve endpoints
│   │   ├── search.py              # /search
│   │   ├── supply_chain.py        # /supply-chain
│   │   └── digest.py              # /digest
│   └── jobs/
│       └── scheduler.py           # Daily/hourly ingest jobs
├── frontend/
│   ├── app/
│   │   ├── page.tsx               # Dashboard
│   │   ├── cve/[id]/page.tsx      # CVE detail (technical + plain-english)
│   │   ├── search/page.tsx        # Search results
│   │   └── supply-chain/page.tsx  # Package lookup
│   └── components/
│       ├── TechnicalView.tsx
│       ├── PlainEnglishView.tsx
│       ├── PreConditionWizard.tsx
│       ├── EpssGauge.tsx
│       └── ChainMap.tsx
├── docker-compose.yml
└── README.md
```

---

## Database Schema (live tables — April 2026)

```sql
-- Core CVE record
cves (
  id TEXT PRIMARY KEY,          -- CVE-2026-32746
  published_at TIMESTAMP,
  last_modified TIMESTAMP,
  cvss_v3_score NUMERIC(4,1),
  cvss_v3_vector TEXT,
  cvss_v3_severity TEXT,
  description TEXT,
  cwe_ids TEXT[],
  affected_products JSONB,      -- CPE list from NVD
  references JSONB,             -- NVD reference array
  created_at TIMESTAMP,
  updated_at TIMESTAMP
)

-- EPSS exploitation probability (daily, one row per CVE per date)
epss_scores (
  id SERIAL PRIMARY KEY,
  cve_id TEXT FK→cves,
  score NUMERIC(7,6),           -- 0.0–1.0
  percentile NUMERIC(7,6),
  scored_date TEXT              -- YYYY-MM-DD
  UNIQUE(cve_id, scored_date)
)

kev_entries (
  cve_id TEXT PRIMARY KEY FK→cves,
  vendor TEXT,
  product TEXT,
  vulnerability_name TEXT,
  date_added TEXT,
  due_date TEXT,
  required_action TEXT,
  ransomware_campaign TEXT,     -- used to group chain links
  notes TEXT
)

-- Plain-English summaries (Claude-generated, cached)
summaries (
  cve_id TEXT PRIMARY KEY FK→cves,
  technical_summary TEXT,
  plain_english TEXT,
  business_impact TEXT,
  pre_conditions JSONB,         -- structured checklist
  chain_context TEXT,           -- Claude-generated text (Phase 2: structured graph)
  remediation_brief TEXT,
  priority_rationale TEXT,
  model_used TEXT,
  prompt_version TEXT,
  generated_at TIMESTAMP,
  needs_refresh BOOLEAN
)

-- Supply chain (OSV.dev advisories)
osv_advisories (
  id TEXT PRIMARY KEY,          -- OSV advisory ID
  cve_id TEXT FK→cves,
  ecosystem TEXT,               -- npm, PyPI, Go, Maven, etc.
  package_name TEXT,
  affected_versions TEXT[],
  fixed_versions TEXT[],
  published_at TIMESTAMP
)

-- PoC / exploit references
poc_refs (
  id SERIAL PRIMARY KEY,
  cve_id TEXT FK→cves,
  source TEXT,                  -- github, exploitdb, metasploit
  url TEXT,
  maturity TEXT,
  published_at TEXT
)

-- CVE chain links (shipped April 2026)
-- Links CVEs that are documented together in attack chains
cve_chain_links (
  id SERIAL PRIMARY KEY,
  source_cve_id TEXT FK→cves,
  linked_cve_id TEXT FK→cves,
  chain_role TEXT,              -- initial_access, info_leak, privilege_escalation, etc.
  direction TEXT,               -- forward / backward
  confidence TEXT,              -- low / medium / high
  source_type TEXT,             -- nvd_reference, cisa_advisory, dfir_report, manual
  source_ref TEXT,
  campaign_name TEXT,           -- e.g. "ALPHV Apr 2026"
  created_at TIMESTAMP
  UNIQUE(source_cve_id, linked_cve_id, chain_role, source_type)
)

-- Ingest audit log
ingest_log (
  id SERIAL PRIMARY KEY,
  source TEXT,                  -- kev, epss, nvd_backfill, chain_nvd, chain_kev...
  started_at TIMESTAMP,
  finished_at TIMESTAMP,
  records_fetched INT,
  records_upserted INT,
  status TEXT,                  -- running / success / error
  error_message TEXT
)

-- PLANNED (Phase 2)
-- cve_patches: CVE → KB mapping from MSRC
-- patch_supersedence: KB → superseded_by KB
-- chain_campaigns: named attack campaigns with ordered CVE steps
```

---

## Implementation Steps

1. **Set up project skeleton** — FastAPI backend, Next.js frontend, PostgreSQL via Docker Compose
2. **Build NVD ingestion** — pull and store CVE records, backfill last 2 years
3. **Build CISA KEV ingestion** — daily feed, mark CVEs as confirmed-exploited
4. **Build EPSS ingestion** — daily CSV download, store per-CVE scores
5. **Build OSV ingestion** — REST API, focus on npm/PyPI/Go initially
6. **Build GitHub Advisory DB ingestion** — GraphQL, fills gaps OSV misses
7. **Wire Claude API summaries** — generate technical + plain-English views, cache in DB
8. **Build CVE detail page** — two-tab UI (Technical / Plain English) with EPSS gauge, KEV badge, pre-condition checklist
9. **Build pre-condition questionnaire** — 5-question wizard per CVE, outputs "likely affected / unlikely affected / need more info"
10. **Build supply chain lookup** — enter package + version, returns affected CVEs from OSV
11. **Build search** — by CVE ID, product, vendor, CWE, severity
12. **Build dashboard** — trending by EPSS, recent KEV additions, community signal
13. **Build digest system** — weekly plain-English email of P0/P1 CVEs
14. **Add ExploitDB scraper** — POC availability badge on CVE pages
15. **Deploy** — Fly.io or Railway for cheap initial hosting

---

## Definition of Done (MVP)

- [ ] All 6 primary data sources ingesting and updating automatically
- [ ] CVE detail page renders both technical and plain-English views
- [ ] Pre-condition questionnaire live for top 100 EPSS CVEs
- [ ] Supply chain lookup working for npm and PyPI
- [ ] Search returning relevant results in < 500ms
- [ ] Dashboard showing trending CVEs with real EPSS data
- [ ] Weekly digest email generating and sending
- [ ] Deployed and publicly accessible

---

## Moats (what makes this hard to copy quickly)

1. **Pre-condition quality** — building accurate pre-condition checklists requires security expertise. Automating this well is hard. Your Glasswing research is the training input. No other platform in the market addresses this at the CVE level.
2. **Summarisation quality** — Claude API summaries need tuning per CVE class. Good prompt engineering here creates a real quality gap vs competitors.
3. **Community trust** — if practitioners find it accurate, they share it. Feedly/VirusTotal took years to build that trust.
4. **Supply chain depth** — OSV + GitHub Advisory + ExploitDB combined gives better coverage than any single source.
5. **Transparent scoring** — Nucleus and Tenable's VPR are black boxes. VulnBrief's composite score (EPSS 35% / KEV 30% / CVSS 20% / PoC 15%) is inspectable and explainable. Practitioners trust what they can audit.
6. **SMB price point with enterprise features** — Nucleus starts at $10/device/year (SMB-hostile at minimum deployment). Tenable minimum is $50K/year. VulnBrief's $49/mo Practitioner tier has no viable competition in the sub-$10K/year range for EPSS-first, pre-condition-aware CVE intelligence.

---

## Risks and Open Questions

| Risk | Mitigation |
|------|-----------|
| NVD rate limits during backfill | Implement respectful pagination, cache aggressively |
| Claude API cost at scale | Cache summaries in DB — only generate once per CVE update |
| Data accuracy for non-technical summaries | Human review queue for high-traffic CVEs |
| Competing with free tools (CISA, NVD) | Differentiation is synthesis + pre-conditions + dual-audience — not raw data |
| Legal: scraping ExploitDB | Use their official download/API, not aggressive scraping |

---

## Hosting & Scaling Architecture

### Start here (MVP — cheap, simple, fast to ship)

| Component | Service | Cost/mo | Notes |
|-----------|---------|---------|-------|
| Backend (FastAPI) | **Fly.io** | ~$10–30 | Auto-scales, simple deploy, good free tier |
| Frontend (Next.js) | **Vercel** | Free → $20 | Best Next.js DX, global CDN built in |
| Database | **Supabase** | Free → $25 | Managed PostgreSQL, has built-in REST API + auth |
| Background jobs | **Fly.io** cron workers | Included | Same platform as backend |
| File storage (CVE data dumps) | **Cloudflare R2** | ~$0 at MVP scale | S3-compatible, free egress |
| Email (digests) | **Resend** | Free → $20 | Modern email API, great deliverability |

**Total MVP cost: ~$0–75/month** until you have real traffic.

### Scale path (1k–100k users)

```
                        ┌─────────────────┐
                        │   Cloudflare    │  ← CDN, DDoS protection, caching
                        └────────┬────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
     ┌────────▼──────┐  ┌───────▼───────┐  ┌──────▼──────┐
     │  Vercel Edge  │  │  Fly.io API   │  │  Fly.io Jobs │
     │  (Next.js)    │  │  (FastAPI)    │  │  (ingest)    │
     └───────────────┘  └───────┬───────┘  └──────┬──────┘
                                │                  │
                        ┌───────▼──────────────────▼──────┐
                        │          Supabase                │
                        │  PostgreSQL + pgvector (future)  │
                        │  + Row Level Security for teams  │
                        └──────────────────┬───────────────┘
                                           │
                                  ┌────────▼────────┐
                                  │  Cloudflare R2  │
                                  │  (CVE data, exports, scan results)
                                  └─────────────────┘
```

**Key scaling decisions:**

| Decision | Recommendation | Why |
|----------|---------------|-----|
| CVE data caching | Cache Claude API summaries in DB permanently | Avoid re-generating — most CVEs don't change after initial enrichment |
| Search | Add **pgvector** to Supabase for semantic search later | Start with full-text search (PostgreSQL built-in), add vector when needed |
| Scanning jobs | Queue with **Inngest** or **Trigger.dev** | Serverless job queue — scales to zero, cheap at low volume |
| CDN / caching | Cloudflare in front of everything | Free tier handles enormous traffic; cache CVE pages at edge |
| Rate limiting | Cloudflare Workers rules | Block scan abuse before it hits your backend |
| Auth (when you add accounts) | **Supabase Auth** | Already in the stack, handles OAuth, magic links |

### What to avoid early

- **AWS / GCP / Azure** — too complex and expensive for MVP; migrate later if you need to
- **Kubernetes** — massively over-engineered until you have 10+ services
- **Self-managed PostgreSQL** — Supabase handles backups, replication, and connection pooling for you

### When to migrate

| Trigger | Action |
|---------|--------|
| > 10k daily active users | Add Redis (Upstash) for rate limiting and session caching |
| > 100k CVE lookups/day | Move to Neon (serverless Postgres) for connection pooling at scale |
| Scanning feature launches | Add dedicated Fly.io worker pool for scan jobs, separate from API |
| Enterprise tier | Consider dedicated DB instances per large customer, move to AWS RDS |

---

## Revenue Model (post-MVP)

| Tier | Price | Features |
|------|-------|---------|
| Free | $0 | Public CVE search, basic EPSS/KEV, plain-English summaries |
| Practitioner | $49/mo | Pre-condition wizard, supply chain scanner, digest email |
| Team | $199/mo | 5 seats, custom watchlists, API access, Slack alerts |
| Enterprise | Custom | SSO, custom feeds, SLA, white-label option |

---

## Connection to Project Glasswing

This IS the Glasswing product. Every pre-condition checklist you build for a CVE in the vulnops repo is content for this platform. Every exploit chain you map is a feature. Every market gap you find during `/vuln-triage` sessions is a product decision.

Build the research operation first (Glasswing). The product comes from what you learn.

---

## Related

- [[Project Glasswing]] — the research initiative feeding this product
- [[Vulnerability Prioritisation]] — the triage framework this platform automates
- [[CVE-2026-32746]] — example of the depth this platform should show
