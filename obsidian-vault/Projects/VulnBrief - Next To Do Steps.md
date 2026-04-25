# VulnBrief — Detailed Next-To-Do Steps Review

**Status:** Live at https://vulnbrief.vercel.app (API: https://vulnbrief-api.fly.dev)  
**Last Review:** April 25, 2026  
**Current Phase:** Post-MVP — Production Hardening & User-Facing Features

---

## Executive Summary

VulnBrief v1.0 is **production-deployed** with 32 passing tests, 23,300 CVEs, and full dashboard functionality. However, **before real paying users**, you must address **4 critical security/correctness issues** and **5 high-value features**. This document prioritizes work into 3 phases.

---

## Critical Fixes (MUST DO BEFORE PAYING USERS)

These are blocking issues that could cause data loss, security breach, or user experience failure.

### 1. API Authentication (CRITICAL)

**Problem:** Zero authentication on any endpoint. Anyone can call `/admin/ingest/*`, read all CVE data, scrape summaries.

**Severity:** 🔴 CRITICAL  
**Effort:** 2–3 hours  
**Impact:** Without this, your API is open to abuse and data theft.

**Tasks:**

- [ ] Add API key header validation to all `/admin/*` endpoints
  - Supabase Auth or simple bearer token validation
  - Sample: `Authorization: Bearer sk_vulnbrief_..._prod`
- [ ] Add rate limiting to search/detail endpoints (slowapi already installed)
  - 100 requests/min for unauthenticated, 1000 for authenticated
- [ ] Document API key generation in README

**Code Location:** `backend/app/main.py` (CORS middleware section)

**Implementation:**

```python
# Add FastAPI dependency for API key validation
async def verify_api_key(authorization: str = Header(None)):
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Missing API key")
    # Validate token (Supabase or your key store)
    return await get_user_from_key(authorization.replace("Bearer ", ""))

# Apply to admin routes
@router.post("/admin/ingest/{source}")
async def trigger_ingest(source: str, auth: User = Depends(verify_api_key)):
    ...
```

---

### 2. CORS Hardening (CRITICAL)

**Problem:** Current CORS allows `*.vercel.app` wildcard. Attacker can host `evil.vercel.app` and make cross-origin requests to your API.

**Severity:** 🔴 CRITICAL  
**Effort:** 15 minutes  
**Impact:** Prevents session hijacking and credential leakage.

**Tasks:**

- [ ] Replace `allow_origins=["*"]` with explicit domain list
- [ ] Pin to `https://vulnbrief.vercel.app` only
- [ ] Add `allow_credentials=True` if using cookies

**Code Location:** `backend/app/main.py` (FastAPI instantiation)

**Implementation:**

```python
app = FastAPI()
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://vulnbrief.vercel.app", "http://localhost:3000"],  # prod + dev only
    allow_credentials=False,  # no cookies for API
    allow_methods=["GET", "POST"],
    allow_headers=["Content-Type", "Authorization"],
)
```

---

### 3. Alembic Baseline (CRITICAL)

**Problem:** No baseline migration recorded. Next schema change will fail with "can't find earlier migration".

**Severity:** 🟠 HIGH  
**Effort:** 30 minutes  
**Impact:** Prevents controlled schema evolution. Blocks v2.0 migration.

**Tasks:**

- [ ] Capture current PostgreSQL schema as `alembic/versions/0001_initial.py`

  ```bash
  alembic revision --autogenerate -m "Initial schema"
  ```
- [ ] Validate migration runs without error

  ```bash
  alembic downgrade base  # should fail (no downgrade path)
  alembic upgrade head    # should be idempotent
  ```
- [ ] Document in README: "Run `alembic upgrade head` on fresh DB"

**Code Location:** `backend/alembic/` (create `0001_initial.py`)

---

### 4. Claude API Error Handling (CRITICAL)

**Problem:** Unhandled `anthropic.APIError` in enrichment pipeline. If Claude API fails, entire batch stalls with no retry logic or logging.

**Severity:** 🟠 HIGH  
**Effort:** 1 hour  
**Impact:** Data enrichment pipeline reliability. Prevents silent failures.

**Tasks:**

- [ ] Wrap Claude API calls with try/except

  ```python
  try:
      response = await client.messages.create(...)
  except anthropic.APIError as e:
      logger.exception(f"Claude API error for {cve_id}", extra={"cve_id": cve_id, "error": str(e)})
      # Retry or skip gracefully
  ```
- [ ] Add retry logic with exponential backoff (Tenacity library)
- [ ] Log rate limit headers to track quota consumption

**Code Location:** `backend/app/enrichment/claude.py`

---

## High-Value Features (SHOULD DO IN v1.1)

These unlock real user value and differentiate VulnBrief from free tools.

### 5. Pagination ORDER BY (HIGH)

**Problem:** `list_cves()` has no ORDER BY clause. Results returned in arbitrary order, making pagination unreliable.

**Severity:** 🟠 HIGH  
**Effort:** 30 minutes  
**Impact:** Users can't reliably browse CVE list. Dashboard looks unpredictable.

**Tasks:**

- [ ] Add `ORDER BY published_at DESC, id ASC` to `list_cves` query
- [ ] Add index on `published_at` for query performance

  ```sql
  CREATE INDEX idx_cves_published_at ON cves(published_at DESC, id ASC);
  ```
- [ ] Test pagination with `?page=1&page=2&page=3` — verify no duplicates

**Code Location:** `backend/app/api/cves.py` (line ~120)

**Implementation:**

```python
query = select(CVE).order_by(CVE.published_at.desc(), CVE.id.asc())
```

---

### 6. Persist Risk Score (HIGH)

**Problem:** Risk score computed at query time (`0.3 × in_kev + epss_score`). Can't sort/filter by risk. Slow for large datasets.

**Severity:** 🟠 HIGH  
**Effort:** 2–3 hours (includes backfill)  
**Impact:** Users can't sort dashboard by "highest risk first". Reports are slow.

**Tasks:**

- [ ] Add `risk_score FLOAT` column to `cves` table

  ```sql
  ALTER TABLE cves ADD COLUMN risk_score FLOAT DEFAULT 0.0;
  ```
- [ ] Create migration to backfill:

  ```sql
  UPDATE cves SET risk_score = CASE WHEN in_kev THEN 0.3 ELSE 0.0 END + COALESCE(epss_score, 0)::float;
  ```
- [ ] Update upsert logic to recompute on every ingest
- [ ] Add index: `CREATE INDEX idx_cves_risk_score ON cves(risk_score DESC);`
- [ ] Update dashboard query to use `ORDER BY risk_score DESC`

**Code Location:**

- Migration: `backend/alembic/versions/0002_add_risk_score.py`
- Upsert: `backend/app/ingest/nvd.py` (CVE insert logic)

---

### 7. Ingest Cursor State (MEDIUM)

**Problem:** Every ingest run re-fetches all data. NVD rate-limited at 120 requests/hour. Manual seed runs slow and error-prone.

**Severity:** 🟡 MEDIUM  
**Effort:** 2–3 hours  
**Impact:** Ingest jobs run faster. Less reliance on manual commands.

**Tasks:**

- [ ] Add `ingest_state` table:

  ```sql
  CREATE TABLE ingest_state (
    source VARCHAR(50) PRIMARY KEY,
    last_cursor VARCHAR(500),  -- NVD: last change_id
    last_run TIMESTAMP,
    next_run TIMESTAMP
  );
  ```
- [ ] Update each ingest (`kev.py`, `nvd.py`, `epss.py`, `osv.py`) to:
  - Read `last_cursor` from table
  - Query only `where change_id > last_cursor`
  - Update `last_cursor` after successful run
- [ ] Test: Run KEV ingest twice, verify second run is <1 second (no data fetched)

**Code Location:**

- `backend/app/ingest/*.py` (all source adapters)
- New: `backend/app/models/ingest_state.py`

---

### 8. Patch Supersedence (MEDIUM—HIGH VALUE, COMPLEX)

**Problem:** Users patch 400 "unpatched CVEs", but half are covered by a newer cumulative patch. No tool surfaces this.

**Severity:** 🟡 MEDIUM  
**Effort:** 4–5 hours (includes MSRC API integration)  
**Impact:** Massive time savings for Windows admins. Huge differentiator.

**Data Source:** Microsoft MSRC API

```
https://api.msrc.microsoft.com/cvrf/v3.0/cvrf/2026-04
```

**Tasks:**

- [ ] Add `patch_supersedence` table:

  ```sql
  CREATE TABLE patch_supersedence (
    id SERIAL PRIMARY KEY,
    cve_id VARCHAR(50),
    superseded_by_kb VARCHAR(50),  -- e.g., "KB5012345"
    source VARCHAR(50),            -- "msrc"
    published_at TIMESTAMP,
    FOREIGN KEY (cve_id) REFERENCES cves(id)
  );
  ```
- [ ] Create `backend/app/ingest/msrc.py`:
  - Fetch MSRC monthly CVRF XML
  - Parse `<Vulnerability>` nodes for `<Supersedes>` elements
  - Extract KB numbers and CVE IDs
  - Upsert into `patch_supersedence`
- [ ] Update CVE detail page:
  - Show badge: "✅ Covered by KB5012345 — no separate patch needed"
- [ ] Update dashboard:
  - Filter: "Hide superseded CVEs" (checkbox)
  - Real patch queue: 400 → 40 actionable items

**Code Location:**

- New: `backend/app/ingest/msrc.py`
- Update: `backend/app/api/cves.py` (detail response)
- Update: `frontend/app/page.tsx` (filter checkbox)

**Note:** MSRC API requires monthly subscription to all historical data. Fallback: parse MSRC Security Updates page (free, manual refresh).

---

### 9. Email Digest (MEDIUM)

**Problem:** Email digest API endpoint exists but not wired to send. Users can't get daily threat alert by email.

**Severity:** 🟡 MEDIUM  
**Effort:** 2–3 hours  
**Impact:** Users get actionable alerts. Increases engagement.

**Tasks:**

- [ ] Set up Resend account (free tier: 100 emails/day)
- [ ] Create `backend/app/email/digest.py`:

  ```python
  async def send_daily_digest(user_email: str):
      digest = await generate_digest()
      await resend.emails.send(
          from_="alerts@vulnbrief.com",
          to=user_email,
          subject=f"VulnBrief Daily Brief — {digest.top_cve.id}",
          html=render_digest_html(digest),
      )
  ```
- [ ] Add scheduled job (APScheduler):

  ```python
  scheduler.add_job(send_daily_digest, "cron", hour=8, minute=0)
  ```
- [ ] Frontend: Add email subscribe form on dashboard
- [ ] Test: Send sample digest to your email

**Code Location:**

- New: `backend/app/email/digest.py`
- Update: `backend/app/jobs/scheduler.py`
- Update: `frontend/components/SubscribeForm.tsx`

---

## Medium-Term Improvements (v1.2 — NICE TO HAVE)

### 10. Test Suite

**Problem:** Manual testing only. No regression protection.

**Severity:** 🟡 MEDIUM  
**Effort:** 4–5 hours  
**Impact:** Confidence for changes. Catch bugs early.

**Scope (MVP):**

- [ ] Ingest happy-paths (KEV, NVD, EPSS)
- [ ] Risk scorer logic (expected outputs)
- [ ] API smoke tests (all endpoints return 2xx)
- [ ] Chain regex (correctly parses exploit chains)

**Code Location:** Create `backend/tests/`

---

### 11. Advanced Search Filters

**Problem:** Search is full-text only. Users can't filter by severity, ecosystem, date range.

**Severity:** 🟡 MEDIUM  
**Effort:** 2–3 hours  
**Impact:** Power users can triage faster.

**Tasks:**

- [ ] Add query params to `GET /cves`:

  ```
  ?severity=critical&ecosystem=npm&published_after=2026-04-01
  ```
- [ ] Frontend: Add filter UI on `/search` page
- [ ] Test: Verify filters compose (all 3 together)

**Code Location:**

- `backend/app/api/cves.py` (query parsing)
- `frontend/app/search/page.tsx` (filter UI)

---

## Long-Term Roadmap (v2.0+)

See `DECISIONS.md` and `docs/v2-migration-plan.md` for:

- **Auth foundation** — Supabase Auth, viewer/analyst/admin roles
- **Status workflow** — CVE triage pipeline (New → In Review → Risk Accepted → Resolved)
- **Export bundle** — JSON/PDF for ServiceNow/Jira integration
- **Patch timeline** — Disclosed → PoC → KEV → Patch (like Feedly)
- **Asset profile matching** — "Does this affect MY stack?"
- **Real-time scanning** — Shodan/Censys integration (v3.0)

---

## Recommended Priority Order

### Week 1 (BLOCKING)

1. **API Auth** (2–3 h) → enables billing, prevents abuse
2. **CORS Fix** (15 min) → security hardening
3. **Alembic Baseline** (30 min) → unblocks future migrations

**Target:** Ship these before announcing to investors/users.

### Week 2 (HIGH VALUE)

1. **Pagination ORDER BY** (30 min) → fixes dashboard UX
2. **Risk Score Persistence** (2–3 h) → enables sorting
3. **Claude Error Handling** (1 h) → reliability

**Target:** Improve core product quality.

### Week 3 (DIFFERENTIATION)

1. **Ingest Cursor State** (2–3 h) → faster syncs
2. **Patch Supersedence** (4–5 h) → killer feature for Windows shops

**Target:** Launch pilot with early customers.

### Week 4+ (ENGAGEMENT)

1. **Email Digest** (2–3 h) → daily alerts
2. **Test Suite** (4–5 h) → confidence
3. **Advanced Filters** (2–3 h) → power users

---

## Success Metrics

After completing this roadmap:

| Metric | Before | After |
|---|---|---|
| **Security** | 0 auth, open CORS | API key + pinned CORS |
| **Data Quality** | No ordering, arbitrary results | `published_at DESC` ordering |
| **Performance** | Risk score on every query | Persisted, indexed, <50ms queries |
| **Reliability** | Silent enrichment failures | Logged, retried, monitored |
| **User Experience** | Basic dashboard | Filtered, sorted, email alerts |
| **Differentiation** | Standard CVE data | Patch supersedence insights |

---

## Questions & Notes

**Q: Should I migrate to Supabase now?**  
A: No. Complete this roadmap on current stack (Fly.io + Postgres). v2.0 migration (Supabase Edge Functions) is a strategic lift for later.

**Q: How much will this cost?**  
A: Current: ~$27/month. After hardening: ~$40/month (with email). No major cost increase.

**Q: Can I do these in parallel?**  
A: Yes, after Week 1 blocking fixes. Auth + CORS + Baseline must complete first. Then fan out: Risk Score + Claude Errors in parallel with Cursor State + Patch Supersedence.

---

## Files to Read Before Starting

1. `CLAUDE.md` — Full architecture & design system
2. `DECISIONS.md` — Known bugs and architecture notes
3. `backend/app/api/cves.py` — Current list/detail logic
4. `backend/app/enrichment/claude.py` — Current enrichment pipeline

---

**Ready to build?** Start with Week 1 blocking fixes. They're quick, high-impact, and unblock everything else.
