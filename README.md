# 🔐 Black Box Penetration Test – Mediroza General Hospital Web Portal

> **Scope:** Authorized black box web application penetration test  
> **Target:** Mediroza General Hospital patient portal (medirozahospital.com)  
> **Tester:** Emmanuel Ufuah  
> **Date:** September 2026  
> **Authorization:** Full written permission granted by Mediroza General Hospital

---

## 📌 Objectives
- Identify vulnerabilities in the patient-facing web portal
- Assess the impact of any discovered weaknesses
- Provide actionable remediation recommendations

---

## 🔍 Phase 1 – Reconnaissance

**Tools used:** `whois` · `nslookup` · `nmap` · `whatweb` · `curl` · `theHarvester` · `wafw00f`

**Key findings:**
- IP: `199.188.201.16` (hosted in the US)
- Web server: OpenResty 1.31.1.1 (Nginx-based)
- Domain registered via NameCheap, created August 2026
- WAF detected – blocking at connection/packet level (connection reset on probe)
- DNSSEC: unsigned

---

## 💉 Phase 2 – SQL Injection (Authentication Bypass)

**Vulnerability:** SQL Injection on patient portal login  
**Payload used:** `admin'--`  
**Result:** Successful authentication bypass — gained access to the patient portal without valid credentials

---

## 📄 Phase 3 – Data Exfiltration & PDF Password Cracking

After accessing the portal, three encrypted PDF pathology reports were available for download:

| Report | Patient | Lab Ref | Date |
|--------|---------|---------|------|
| patient_report_1.pdf | S. Dlamini | LR-2024-1187 | 2024-11-04 |
| patient_report_2.pdf | P. Reddy | LR-2024-1192 | 2024-11-05 |
| patient_report_3.pdf | E. Thompson | LR-2024-1205 | 2024-11-06 |

**Method:**
1. Uploaded PDFs to OnlineHashCrack PDF Hash Extractor (pdf2john)
2. Submitted extracted `$pdf$` hashes to dictionary attack tool
3. Recovered all three passwords

| File | Password | Wordlist Required |
|------|----------|-------------------|
| patient_report_1.pdf | `123456` | Built-in (100 entries) |
| patient_report_2.pdf | `password` | Built-in (100 entries) |
| patient_report_3.pdf | `!@#$%^&` | Extended (~3556 entries) |

**Post-decrypt:** Used `qpdf` to strip encryption and `exiftool` to extract metadata. Report 3 metadata revealed an internal comment: `DB backup moved to /old before site migration, do not delete`.

---

## 📂 Phase 4 – Sensitive Backup Discovery via robots.txt

**Command:** `curl -s https://medirozahospital.com/robots.txt`  
**Finding:** `/old/` directory listed as disallowed — indicating sensitive content

Enumeration of `/old/` revealed a downloadable MySQL database backup containing:

- **`staff` table** – 30 records including full names, job titles, departments, email addresses, phone numbers, national ID numbers, and monthly salaries
- **`shareholders` table** – 10 records including shareholder names, share percentages, shares held, and share class (Ordinary/Preferential)

Notable author from exiftool metadata on report 3: `j.malik` — matching a staff record (Jameel Malik, IT Systems Administrator).

---

## ⚠️ Vulnerability Summary

| # | Vulnerability | Severity | Impact |
|---|---------------|----------|--------|
| 1 | SQL Injection – Authentication Bypass | Critical | Unauthorized portal access |
| 2 | Unauthenticated PDF Download | High | Patient data exposure |
| 3 | Weak PDF Encryption Passwords | High | Patient health data decrypted |
| 4 | robots.txt Disclosing Sensitive Path | Medium | Backup directory exposed |
| 5 | Unprotected Legacy Backup Directory | Critical | Full staff & shareholder data leaked |
| 6 | PII & Financial Data in Plaintext Backup | Critical | Regulatory & legal risk |

---

## 🛠️ Tools Used

`whois` · `nslookup` · `nmap` · `whatweb` · `curl` · `theHarvester` · `wafw00f` · `OnlineHashCrack` · `qpdf` · `exiftool` · `Kali Linux`

---

## ✅ Recommendations

1. Implement parameterized queries / prepared statements to prevent SQLi
2. Enforce authentication before allowing file downloads
3. Use strong, unique, randomly generated PDF passwords
4. Remove or restrict access to `/old/` and all legacy directories
5. Never expose database backups via web-accessible paths
6. Sanitize metadata in documents before distribution
7. Enable DNSSEC
8. Conduct regular security audits and staff security awareness training

---

> *This test was conducted with full written authorization from Mediroza General Hospital. All findings were responsibly disclosed to the client's IT team for remediation.*
