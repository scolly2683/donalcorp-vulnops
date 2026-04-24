# donalcorp-vulnops

Internal vulnerability operations sandbox. Authorised internal testing only — all tooling is for known CVEs against lab environments or explicitly scoped targets.

## Repository Layout

```
donalcorp-vulnops/
├── CLAUDE.md
├── README.md
├── cve-2026-32746/          ← CVE assessment module (GNU InetUtils telnetd)
│   ├── README.md            ← Full vulnerability documentation
│   ├── detect.py            ← Non-destructive probe (safe to run)
│   ├── exploit.py           ← PoC — triggers crash, reads leaked BSS memory
│   ├── scan_org.py          ← Concurrent CIDR scanner, outputs CSV
│   ├── Dockerfile           ← Vulnerable lab target (Debian + inetutils-telnetd 2.7)
│   ├── docker-compose.yml   ← Exposes lab on localhost:2323
│   └── xinetd-telnet.conf
└── obsidian/                ← Obsidian knowledge base notes
    └── OpenMythos.md        ← Ingest of kyegomez/OpenMythos (RDT architecture)
```

## CVE-2026-32746

Pre-auth buffer overflow in GNU InetUtils telnetd. Key scripts:

- `detect.py` — sends LINEMODE SLC negotiation, identifies vulnerable hosts without crashing
- `exploit.py` — sends 60 SLC triplets to trigger overflow; analyzes leaked BSS memory
- `scan_org.py` — scans CIDR ranges concurrently (default 50 threads); CSV output with columns: `ip`, `port`, `status` (VULNERABLE / NOT_VULNERABLE / NO_TELNET / UNREACHABLE), `detail`

Lab: `docker compose up` in `cve-2026-32746/` → telnetd on `localhost:2323`.

## Obsidian Ingest

Notes in `obsidian/` are Markdown files ingested from external sources. Each note includes source URL, ingest date, architecture/concept summary, usage examples, and Obsidian wikilinks.

To ingest a new repo: fetch README + key source files, create `obsidian/<Name>.md` following the pattern in `OpenMythos.md`.

## Branch Conventions

Feature branches follow `claude/<task>-<ID>` naming (created by Claude Code sessions). Always develop on the designated branch and push before ending a session.

## GitHub MCP Scope

Restricted to `scolly2683/donalcorp-vulnops`. To add another repo (e.g. vulnbrief), update the allowed repository list in the session harness config and restart the session.
