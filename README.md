# VAPT Home Lab

A self-built, isolated VirtualBox environment where I practise the full Vulnerability Assessment and Penetration Testing (VAPT) cycle: reconnaissance, scanning, vulnerability identification, exploitation, and reporting.

> **Authorization & scope.** Every machine here is one I built and own, running on an isolated host-only network (`192.168.56.0/24`) with no route to the internet or any external system. This is a personal lab for learning penetration testing. Nothing was tested against any system I do not control.

---

## Lab Environment

| Machine | Role | Purpose |
| ------- | ---- | ------- |
| Kali Linux 2026.2 | Attacker | All tooling runs here |
| Windows 7 Enterprise SP1 | Target | Windows enumeration, SMB, legacy attacks |
| Metasploitable 2 | Target | Vulnerable Linux services |
| Metasploitable 3 | Target | Vulnerable Windows Server services |
| Windows 10 | Target | Modern endpoint, privilege escalation |

Hypervisor: **Oracle VirtualBox** · Network: **Host-Only** `192.168.56.0/24`, no internet route.

---

## Write-Ups

| Target | Focus | Status |
| ------ | ----- | ------ |
| [Windows 7 — MS17-010 Assessment](windows7-ms17-010.md) | SMB enumeration, MS17-010 testing, SMB signing finding | ✅ Complete |
| Metasploitable 2 | Service enumeration and exploitation | 🔜 In progress |
| Metasploitable 3 | Windows service exploitation | 🔜 Planned |
| Windows 10 | Privilege escalation | 🔜 Planned |

---

## Methodology

Each assessment follows the same cycle:

1. **Reconnaissance** — host discovery and full service/version enumeration (Nmap)
2. **Vulnerability Identification** — targeted testing, with manual verification of every result
3. **Exploitation** — validated exploitation of confirmed vulnerabilities (Metasploit, manual)
4. **Post-Exploitation** — proving impact and access
5. **Reporting** — findings, evidence, and remediation, mapped to MITRE ATT&CK

A note on approach: I document real outcomes, including negative ones. The Windows 7 assessment, for example, confirmed the target was *not* exploitable via MS17-010 because SMBv1 was disabled — and investigating *why* the check failed is itself a documented part of the work.

---

## Contact

**Yousef Alturki** — [LinkedIn](https://www.linkedin.com/in/reachyousefalturki) · reachyousefalturki@gmail.com
