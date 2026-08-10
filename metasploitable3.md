# Metasploitable 3 — Windows Server 2008 R2 Exploitation

> 🔜 **Status: In progress.** This document maps the target and its attack surface. Individual exploitation write-ups will be added here as each service is worked through.

**Target:** Metasploitable 3 (Windows Server 2008 R2) · `192.168.56.105` *(adjust to your VM's IP)*
**Attacker:** Kali Linux · `192.168.56.103`
**Network:** Isolated VirtualBox host-only network `192.168.56.0/24`, no internet route
**Focus:** Windows service exploitation, credentialed lateral movement, privilege escalation to `NT AUTHORITY\SYSTEM`

> **Authorization.** All testing is performed against a virtual machine I built and own, on an isolated network with no route to the internet or any external system. This is a personal lab set up for the purpose of learning penetration testing.

---

## 1. About the Target

Unlike Metasploitable 2, which shipped as a ready-made disk image, **Metasploitable 3 is built on demand** using Packer and Vagrant. The build scripts provision a base OS and then install and misconfigure a long list of vulnerable services on top of it. Rapid7 publishes two editions:

| Edition | Base OS | Theme |
| ------- | ------- | ----- |
| **Windows** (this write-up) | Windows Server 2008 R2 | Windows services, web apps, SMB/WinRM |
| Linux | Ubuntu 14.04 | Linux daemons and web apps (ProFTPD mod_copy, Docker, Drupal, etc.) |

The Windows edition also hides a set of **capture-the-flag tokens** (playing-card themed) around the file system, meant to be recovered as proof of access after exploitation.

---

## 2. Default Credentials

The build ships with well-known weak credentials, which are the intended starting point for credentialed attacks (SMB PsExec, WinRM, etc.):

| Account | Password |
| ------- | -------- |
| `administrator` | `vagrant` |
| `vagrant` | `vagrant` |

It also creates a number of **weak, themed local accounts** that are useful practice targets for SMB/WinRM password brute-forcing (e.g. with `medusa`, `hydra`, or the Metasploit `smb_login` / `winrm_login` scanners).

---

## 3. Attack Surface — Vulnerable Services

The Windows edition exposes many services across a wide range of ports. Note that several services deliberately **share ports** (e.g. 8282 hosts Tomcat / Struts / Axis2; 8585 hosts WebDAV / WordPress / phpMyAdmin).

| # | Service | Port | Weakness → Path to code execution | Reference |
| - | ------- | ---- | --------------------------------- | --------- |
| 1 | **IIS FTP** | 21 | Weak credentials → authenticated FTP access | `auxiliary/scanner/ftp/ftp_login` |
| 2 | **IIS HTTP (HTTP.sys)** | 80 | HTTP.sys range integer overflow → DoS / potential RCE | **MS15-034 / CVE-2015-1635** |
| 3 | **China Chopper web shell** | 80 | Planted web shell, weak password → brute force login | `auxiliary/scanner/http/caidao_bruteforce_login` |
| 4 | **SNMP** | 161/UDP | Default community string → system info disclosure | `auxiliary/scanner/snmp/snmp_enum` |
| 5 | **SMB / NetBIOS (PsExec)** | 445 / 139 | Valid credentials (`vagrant`) → PsExec → `SYSTEM` | `exploit/windows/smb/psexec` |
| 6 | **JMX (Java)** | 1617 | Unauthenticated MBean → arbitrary Java class load → RCE | `exploit/multi/misc/java_jmx_server` |
| 7 | **Ruby on Rails** | 3000 | Web Console remote code execution | `exploit/multi/http/rails_web_console_v2_code_exec` |
| 8 | **MySQL** | 3306 | Weak credentials → MOF technique → RCE as `SYSTEM` | `exploit/windows/mysql/mysql_mof` |
| 9 | **GlassFish** | 4848 / 8080 / 8181 | Weak admin login → authenticated WAR deploy → RCE | `exploit/multi/http/glassfish_deployer` |
| 10 | **WinRM** | 5985 | Weak credentials → remote command execution | `exploit/windows/winrm/winrm_script_exec` |
| 11 | **ManageEngine Desktop Central 9** | 8020 | `statusUpdate` file-upload flaw → RCE as `SYSTEM` | **CVE-2015-8249** |
| 12 | **Apache Tomcat** | 8282 | Manager weak credentials → WAR upload → RCE | `exploit/multi/http/tomcat_mgr_upload` |
| 13 | **Apache Struts 2** | 8282 | DMI REST plugin remote code execution | **CVE-2016-3087** |
| 14 | **Apache Axis2** | 8282 | Weak admin login → malicious service deploy → RCE | `exploit/multi/http/axis2_deployer` |
| 15 | **Jenkins** | 8484 | Unauthenticated Groovy script console → RCE | `exploit/multi/http/jenkins_script_console` |
| 16 | **WebDAV (IIS)** | 8585 | HTTP `PUT` enabled → upload web shell → RCE | `auxiliary/scanner/http/http_put` |
| 17 | **WordPress** | 8585 | Ninja Forms unauthenticated file upload → RCE | `wp_ninja_forms_unauthenticated_file_upload` |
| 18 | **phpMyAdmin** | 8585 | `preg_replace` `/e` modifier → RCE | **CVE-2013-3238** |
| 19 | **ElasticSearch** | 9200 | MVEL dynamic scripting → RCE | **CVE-2014-3120** |

> **Note on CVEs/modules:** where a service maps cleanly to a published CVE it's listed; otherwise the relevant Metasploit module is given as the reference. Module and CVE details should be verified against the live box during testing rather than assumed.

---

## 4. Planned Methodology

The engagement will follow the same phased approach as the rest of this lab. Progress is tracked below:

- [ ] **Reconnaissance** — full `nmap -sV -p-` scan, service/version enumeration, OS fingerprinting
- [ ] **Credential attacks** — SMB / WinRM / FTP / database login brute-forcing against the weak accounts
- [ ] **Web application exploitation** — Tomcat, Jenkins, GlassFish, Struts, ManageEngine, phpMyAdmin, WebDAV
- [ ] **Service exploitation** — ElasticSearch, JMX, MySQL, Rails
- [ ] **Foothold → SYSTEM** — PsExec with recovered credentials, privilege escalation
- [ ] **Post-exploitation** — flag recovery, credential harvesting, persistence review
- [ ] **Remediation & MITRE ATT&CK mapping**
- [ ] **Conclusion**

_🔜 To be continued._

---
