---
tags: [remediation, patch, mitigation, telnet, ssh]
cve: "[[CVE-2026-32746]]"
created: 2026-04-17
---

# Remediation — CVE-2026-32746

---

## Recommended actions (in order of preference)

1. **Disable telnetd and migrate to SSH** — telnet transmits credentials in plaintext; SSH is strictly preferred
2. **Upgrade** `inetutils-telnetd` to > 2.7 once a patched release is available
3. **Firewall** — restrict port 23 access to authorised management hosts only as an interim control

---

## Check installed version

```bash
# Debian / Ubuntu
dpkg -l inetutils-telnetd
```

Versions **≤ 2.7** are affected.

---

## Disable via xinetd (immediate mitigation)

```bash
sed -i 's/disable.*=.*no/disable = yes/' /etc/xinetd.d/telnet
systemctl reload xinetd
```

Verify the port is no longer listening:

```bash
ss -tlnp | grep :23
```

---

## Upgrade (when patch is available)

```bash
apt update
apt install --only-upgrade inetutils-telnetd
```

Confirm the new version:

```bash
dpkg -l inetutils-telnetd
```

---

## Verify remediation

Run [[detect]] after applying the fix to confirm the host no longer responds as vulnerable:

```bash
python3 detect.py <host_ip>
# Expected: [-] NOT vulnerable
```

---

## Related

- [[CVE-2026-32746]] — full vulnerability details
- [[scan_org]] — re-scan the org after patching to verify coverage
