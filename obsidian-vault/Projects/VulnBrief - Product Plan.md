---
tags: [project, startup, product, vulnerability-intelligence, glasswing]
created: 2026-04-17
status: planning
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

## MVP Scope (v1 — build this first)

### In scope
- [ ] CVE detail page: technical view + plain-English view (Claude API)
- [ ] Search by CVE ID, vendor, product, CWE
- [ ] Dashboard: trending by EPSS, CISA KEV additions, community signal
- [ ] Pre-condition questionnaire for each CVE (5 questions max)
- [ ] Supply chain lookup: package name + version → affected CVEs
- [ ] Weekly digest email: plain-English P0/P1 summary

### Out of scope for v1
- User accounts / saved watchlists
- API access for third parties
- Team/org features
- Custom alerting rules
- Threat actor attribution
- Active scanning features
- Paid tiers (ship free first, validate, then monetise)

### Phase 2 additions (post-MVP)
- Shodan + Censys + GreyNoise integration for exposure context on CVE pages
- Watchlists — subscribe to vendors/products, get alerts on new P0/P1s (OpenCVE-powered)
- Hudson Rock infostealer context for auth-related CVEs
- **Asset Profile Module** — users define their software stack, network exposure tier, and auth config once; CVEs auto-matched to profile; risk score becomes personalized (see Competitive Intelligence note — this is the gap all enterprise tools address only at $50K+)
- **Compensating Controls question in Pre-Condition Wizard** — add Q6: "Does a WAF / EDR / network segment mitigate this vector?" (what Cymulate validates at enterprise scale, VulnBrief asks as a wizard question)
- **Exploit maturity expansion** — beyond GitHub PoCs: add Metasploit module availability, Nuclei template availability, ExploitDB entry presence, time-to-weaponization estimate (Wiz Q1 2025: 28.3% of exploited CVEs weaponized within 24h — recency of PoC activity matters as much as presence)

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

## Database Schema (key tables)

```sql
-- Core CVE record
cves (
  id TEXT PRIMARY KEY,          -- CVE-2026-32746
  published_at TIMESTAMP,
  last_modified TIMESTAMP,
  cvss_score FLOAT,
  cvss_vector TEXT,
  description TEXT,
  cwe_ids TEXT[],
  affected_products JSONB,      -- CPE list
  references JSONB
)

-- Enrichment
epss_scores (
  cve_id TEXT,
  score FLOAT,                  -- 0.0–1.0
  percentile FLOAT,
  scored_at DATE
)

kev_entries (
  cve_id TEXT PRIMARY KEY,
  vendor TEXT,
  product TEXT,
  added_date DATE,
  due_date DATE,
  ransomware_use TEXT,
  notes TEXT
)

-- Plain-English summaries (Claude-generated, cached)
summaries (
  cve_id TEXT PRIMARY KEY,
  technical_summary TEXT,
  plain_english TEXT,
  business_impact TEXT,
  pre_conditions JSONB,         -- structured checklist
  chain_context TEXT,
  generated_at TIMESTAMP,
  model_used TEXT
)

-- Supply chain
osv_advisories (
  osv_id TEXT PRIMARY KEY,
  cve_id TEXT,
  ecosystem TEXT,               -- npm, PyPI, Go, etc.
  package_name TEXT,
  affected_versions TEXT[],
  fixed_versions TEXT[]
)
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
