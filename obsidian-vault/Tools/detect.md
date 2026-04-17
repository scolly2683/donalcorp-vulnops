---
tags: [tool, detection, safe, telnet]
cve: "[[CVE-2026-32746]]"
destructive: false
created: 2026-04-17
---

# detect.py — Non-destructive LINEMODE Probe

Safe to run against **live production systems** with authorisation. Does **not** send any overflow payload — only negotiates LINEMODE SLC to check if the server responds in a pattern consistent with vulnerable GNU InetUtils telnetd.

---

## Usage

```bash
# Single host, default port 23
python3 detect.py 192.168.1.10

# Custom port (e.g. lab on 2323)
python3 detect.py 127.0.0.1 2323

# Adjust timeout
python3 detect.py 192.168.1.10 23 -t 5
```

---

## Output

**Likely vulnerable:**
```
[!] LIKELY VULNERABLE to CVE-2026-32746
    Server supports LINEMODE with SLC negotiation
    GNU InetUtils telnetd through 2.7 is affected
    CVSS: 9.8 (Critical) | No patch available
```

**Not vulnerable / no telnet:**
```
[-] NOT vulnerable (LINEMODE not accepted)
[-] NO_TELNET — port closed or no banner
```

---

## How it works

1. Opens TCP connection to target on port 23 (or specified port)
2. Sends RFC 854 telnet `DO LINEMODE` option
3. If server responds with `WILL LINEMODE`, sends a minimal SLC suboption
4. Checks response size — vulnerable telnetd returns a characteristic oversized SLC reply
5. Does **not** send ≥ 35 triplets; no overflow is triggered

---

## Related

- [[CVE-2026-32746]] — vulnerability details
- [[exploit]] — destructive confirmation tool
- [[scan_org]] — wraps this logic for org-wide scanning
