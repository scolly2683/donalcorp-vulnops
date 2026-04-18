---
tags: [tool, scanner, cidr, concurrent, telnet]
cve: "[[CVE-2026-32746]]"
destructive: false
created: 2026-04-17
---

# scan_org.py — Concurrent Org-wide CIDR Scanner

Wraps the non-destructive [[detect]] logic to scan multiple CIDR ranges in parallel using `ThreadPoolExecutor`. Safe for production — no overflow payload is sent.

---

## Usage

```bash
# Scan a /24
python3 scan_org.py 10.0.0.0/24

# Scan multiple ranges, 100 threads, save to CSV
python3 scan_org.py 10.0.0.0/8 172.16.0.0/12 --threads 100 --output results.csv

# Scan lab target on non-standard port
python3 scan_org.py 127.0.0.1/32 --port 2323

# Show all results including non-vulnerable hosts
python3 scan_org.py 10.0.0.0/24 --verbose
```

| Argument | Default | Description |
|----------|---------|-------------|
| `CIDR [CIDR...]` | — | One or more CIDR ranges or individual IPs |
| `--threads` | 50 | Concurrent threads |
| `--port` | 23 | Telnet port to probe |
| `--output` | — | CSV output file path |
| `--verbose` | off | Show all results, not just vulnerable |

---

## Output

By default only **vulnerable hosts** are printed. With `--verbose`, all results are shown.

CSV columns: `ip, port, status, detail`

| Status | Meaning |
|--------|---------|
| `VULNERABLE` | LINEMODE accepted — likely GNU InetUtils telnetd ≤ 2.7 |
| `NOT_VULNERABLE` | LINEMODE not accepted |
| `NO_TELNET` | Port closed or no telnet banner |
| `UNREACHABLE` | Timeout or connection error |

**Exit codes:**
- `0` — no vulnerable hosts found
- `1` — one or more vulnerable hosts detected

---

## Related

- [[CVE-2026-32746]] — vulnerability details
- [[detect]] — single-host detection logic this wraps
- [[exploit]] — use after identifying vulnerable hosts (with authorisation)
