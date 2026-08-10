# Windows 7 — MS17-010 (EternalBlue) Exploitation

> 🔜 **Status: In progress.** Reconnaissance is complete and documented below. Vulnerability testing and exploitation are still under way and will be added as the work continues.

**Target:** Windows 7 (`IEWIN7`) · `192.168.56.102`
**Attacker:** Kali Linux · `192.168.56.103`
**Network:** Isolated VirtualBox host-only network `192.168.56.0/24`, no internet route
**Vulnerability (under assessment):** MS17-010 / CVE-2017-0144 — SMBv1 Remote Code Execution

> **Authorization.** All testing was performed against a virtual machine I built and own, on an isolated network with no route to the internet or any external system. This is a personal lab set up for the purpose of learning penetration testing.

---

## 1. Scope & Environment Setup

The lab consists of two virtual machines on an isolated VirtualBox host-only network (`192.168.56.0/24`) with no gateway and no internet access.

| Machine | Role | IP |
| ------- | ---- | -- |
| Kali Linux | Attacker | 192.168.56.103 |
| Windows 7 (IEWIN7) | Target | 192.168.56.102 |

_(Note during setup: the Windows host firewall initially blocked SMB, and a host-only adapter mismatch initially prevented layer-3 connectivity between the VMs. Both were resolved before assessment.)_

---

## 2. Reconnaissance

### 2.1 Host Discovery

**Command:**
```bash
nmap -sn 192.168.56.0/24
```

**Result:** Three live hosts on the isolated network — the adapter (`192.168.56.100`), the Windows 7 target (`192.168.56.102`), and the Kali attacker (`192.168.56.103`).

![Host discovery](screenshots/01-host-discovery.png)

### 2.2 Aggressive Service, Version & OS Scan

An aggressive scan (`-A`) was run against the target to enumerate all services, versions, and the operating system without prior assumptions about what was running.

**Command:**
```bash
nmap -A 192.168.56.102
```

**Key results:**

| Port | Service | Detail |
| ---- | ------- | ------ |
| 22/tcp | OpenSSH 6.7 | Non-native — SSH was installed on the host |
| 135/tcp | msrpc | Microsoft Windows RPC |
| 139/tcp | netbios-ssn | NetBIOS session service |
| 445/tcp | microsoft-ds | **Windows 7 Enterprise 7601 SP1** — SMB |
| 5357/tcp | http | Microsoft HTTPAPI 2.0 (UPnP) |
| 49152–49157 | msrpc | Dynamic RPC ports |

**Operating system:** `smb-os-discovery` confirmed **Windows 7 Enterprise 7601 Service Pack 1**, computer name `IEWIN7`, workgroup `WORKGROUP`. Nmap's TCP/IP OS detection guessed Server 2008 R2, but the SMB banner is authoritative — Windows 7 and Server 2008 R2 share a kernel and fingerprint similarly, so the SMB script result was taken as correct.

![Service scan](screenshots/02-service-scan.png)

**Additional observation — weak SMB configuration.** The scan flagged `message_signing: disabled (dangerous, but default)` and `Message signing enabled but not required`. Disabled/optional SMB signing exposes the host to SMB relay attacks, independent of the RCE vulnerability being assessed. This will be followed up in the findings phase.

![SMB script results](screenshots/02-1-service-scan.png)

---

## 3. Next Steps — In Progress

Reconnaissance is complete. The following phases are under way and will be documented here as the engagement continues:

- [ ] **Vulnerability identification** — direct MS17-010 testing (`nmap --script smb-vuln-ms17-010`, Metasploit `auxiliary/scanner/smb/smb_ms17_010`)
- [ ] **Exploitation** — EternalBlue exploitation attempt against the SMBv1 attack surface
- [ ] **Additional findings** — SMB signing / relay exposure follow-up
- [ ] **Remediation & MITRE ATT&CK mapping**
- [ ] **Conclusion**

_🔜 To be continued._

---
