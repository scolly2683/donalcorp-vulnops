---
tags: [lab, docker, setup]
cve: "[[CVE-2026-32746]]"
created: 2026-04-17
---

# Lab Setup — Docker Vulnerable Environment

Spins up a local Debian Bookworm container running a vulnerable version of GNU InetUtils telnetd for safe, isolated testing.

---

## Requirements

- Docker + Docker Compose

---

## Start the lab

```bash
cd cve-2026-32746/
docker compose up -d
# Vulnerable telnetd now listening on 127.0.0.1:2323
```

The container exposes port **2323** on the host (maps to port 23 inside the container).

---

## Test with tools

```bash
# Non-destructive detection
python3 detect.py 127.0.0.1 2323

# Exploit confirmation (will crash the telnetd process)
python3 exploit.py 127.0.0.1 2323
```

---

## Tear down

```bash
docker compose down
```

---

## Container details

| Property | Value |
|----------|-------|
| Base image | Debian Bookworm |
| Service | inetutils-telnetd via xinetd |
| Host port | 2323 |
| Container port | 23 |
| Test user | `testuser` (for login verification) |

---

## Related

- [[CVE-2026-32746]] — vulnerability being tested
- [[detect]] — safe probe tool
- [[exploit]] — confirmation tool
- [[Remediation]] — how to fix after testing
