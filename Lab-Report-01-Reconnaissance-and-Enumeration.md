# Lab Report 01 — Reconnaissance, Scanning & Enumeration
**Author:** Frank Arhin  
**GitHub:** Frank-CyberSec  
**Date:** 01 May 2026  
**Classification:** Personal Lab — Educational Purposes Only  

---

## 1. Executive Summary

This report documents the findings of a penetration testing lab exercise conducted in a controlled virtual environment. The objective was to perform reconnaissance, network scanning, and enumeration against a Windows 11 target machine using Kali Linux 2026.1 as the attacking platform. The exercise covers CEH v13 Domains 2, 3, 4, and 5.

---

## 2. Lab Environment

| Component | Details |
|---|---|
| **Attacker Machine** | Kali Linux 2026.1 |
| **Target Machine** | Windows 11 Enterprise 24H2 |
| **Hypervisor** | Oracle VirtualBox |
| **Network Type** | Bridged Adapter |
| **Attacker IP** | 10.149.40.x |
| **Target IP** | 10.149.40.195 |

---

## 3. Methodology

The following phases were conducted in accordance with the CEH v13 penetration testing methodology:

1. Passive Reconnaissance
2. Active Reconnaissance
3. Network Scanning
4. Enumeration
5. Vulnerability Analysis

---

## 4. Tools Used

| Tool | Purpose | CEH Domain |
|---|---|---|
| **Nmap 7.99** | Network scanning and OS detection | Domain 3 |
| **enum4linux 0.9.1** | SMB enumeration | Domain 4 |
| **dns-brute** | DNS enumeration | Domain 2 |
| **Nmap vuln scripts** | Vulnerability scanning | Domain 5 |

---

## 5. Findings

### 5.1 Initial Nmap Scan

**Command Used:**
```bash
nmap -sV 10.149.40.195
```

**Results:**

| Port | State | Service | Version |
|---|---|---|---|
| 135/tcp | Open | MSRPC | Microsoft Windows RPC |
| 139/tcp | Open | NetBIOS-SSN | Microsoft Windows NetBIOS |
| 445/tcp | Open | Microsoft-DS | SMB |

---

### 5.2 Aggressive Nmap Scan

**Command Used:**
```bash
nmap -A -T4 10.149.40.195
```

**Results:**

| Finding | Detail |
|---|---|
| **OS Detected** | Microsoft Windows 11 24H2 |
| **Hostname** | FRANKARHIN |
| **Workgroup** | WORKGROUP |
| **MAC Address** | 08:00:27:F7:27:76 (Oracle VirtualBox) |
| **SMB Signing** | Enabled and Required |
| **Clock Skew** | -11 seconds |
| **Network Distance** | 1 hop |

---

### 5.3 SMB Enumeration

**Command Used:**
```bash
enum4linux -a 10.149.40.195
```

**Results:**

| Finding | Detail |
|---|---|
| **Computer Name** | FRANKARHIN |
| **Workgroup** | WORKGROUP |
| **File Server** | Active (port 20) |
| **MAC Address** | 08-00-27-F7-27-76 |

**Known Usernames Discovered:**

| Username | Significance |
|---|---|
| administrator | Default Windows administrator account |
| guest | Default guest account |
| krbtgt | Kerberos ticket granting account |
| domain admins | Administrative group |
| root | Superuser account |

---

### 5.4 Vulnerability Scan

**Command Used:**
```bash
nmap --script vuln 10.149.40.195
```

**Results:**

| Vulnerability | CVE | Result |
|---|---|---|
| MS10-061 | CVE-2010-2729 | Could not negotiate connection |
| Samba vulnerability | CVE-2012-1182 | Could not negotiate connection |
| MS10-054 | CVE-2010-2550 | Not Vulnerable |

---

## 6. Risk Assessment

| Finding | Risk Level | Reason |
|---|---|---|
| Port 445 SMB Open | 🔴 High | SMB historically exploited |
| Administrator account found | 🔴 High | Default account is a target |
| Guest account found | 🟡 Medium | Could allow unauthorised access |
| Windows 11 24H2 | 🟢 Low | Modern OS, well patched |
| SMB Signing enabled | 🟢 Low | Protects against relay attacks |

---

## 7. Recommendations

1. **Disable SMB if not required** — Port 445 should be closed if file sharing is not needed
2. **Rename the Administrator account** — Default account names are easily guessed
3. **Disable the Guest account** — Prevents unauthorised access
4. **Enable Windows Firewall** — Was disabled during testing, should be re-enabled
5. **Keep Windows updated** — Continue applying security patches regularly
6. **Enable SMB signing** — Already enabled, maintain this configuration

---

## 8. Conclusion

The penetration test successfully identified several open ports and services on the Windows 11 target machine. The target was found to be running modern software and was not vulnerable to older exploits. However, the open SMB port and discoverable user accounts present potential attack vectors that should be addressed.

This exercise demonstrated practical skills in reconnaissance, scanning, enumeration and vulnerability analysis, covering CEH v13 Domains 2, 3, 4 and 5.

---

## 9. References

- EC-Council CEH v13 Official Courseware
- Nmap Documentation: https://nmap.org/docs.html
- OWASP Testing Guide: https://owasp.org/www-project-web-security-testing-guide/
- CVE Database: https://cve.mitre.org

---

*This report was produced in a controlled lab environment for educational purposes only. All testing was conducted on machines owned and operated by the author.*
