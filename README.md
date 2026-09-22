# Ethical Hacking & Web Application Security Assessment

**Course:** Ethical Hacking & Web Security Lab  
**Student Roll Number:** `2311CS040082`  
**Assessment Identifier:** `2311CS040082-ETHICAL-HACKING-INTERNAL-ASSESSMENT`  
**Repository Contents:** Complete Academic Lab Writeups, Methodologies, Exploit Proofs, Evidence Indices & Technical Reports  

---

## 📌 Repository Purpose & Overview

This repository contains the comprehensive academic lab writeups, methodologies, and technical documentation for the **Ethical Hacking Internal Assessment**. 

The documentation covers two primary experimental investigations conducted within an authorized, isolated sandbox environment:

1. **[Experiment 01: SQL Injection Detection and Prevention](docs/EXPERIMENT_01_SQLI_WRITEUP.md)** (`ASSIGNMENT-01-SQLI`)
2. **[Experiment 02: Integrated Ethical Hacking Assessment](docs/EXPERIMENT_02_ETHICAL_HACKING_WRITEUP.md)** (`ASSIGNMENT-02-ETHICAL-HACKING`)

> [!NOTE]
> **Academic Note**: This repository serves as the centralized academic submission containing the complete, verified lab writeups, technical data flow, vulnerability ledgers, code comparisons, and cryptographic evidence hashes.

---

## 📖 Direct Links to Full Lab Writeups

- 📄 **[Experiment 01 Full Writeup (.md)](docs/EXPERIMENT_01_SQLI_WRITEUP.md)**: Deep-dive analysis of SQL Injection (CWE-89), attack mechanics (Authentication Bypass, UNION-based extraction, Numeric SQLi), Insecure vs. Remediated code comparison, prepared statements implementation, and regression verification results.
- 📄 **[Experiment 02 Full Writeup (.md)](docs/EXPERIMENT_02_ETHICAL_HACKING_WRITEUP.md)**: Complete 7-Phase penetration testing lifecycle (Reconnaissance $\to$ Scanning $\to$ Enumeration $\to$ Vuln Analysis $\to$ Controlled Exploitation $\to$ Defense Hardening $\to$ Retesting), 8 vulnerability findings with CVSS v3.1 scoring, false positive triage, and verification matrix.

---

## 🔬 How the Experiments Were Conducted

### 🧪 Experiment 01: SQL Injection Detection & Prevention
- **Target Surface:** An isolated SQLite database sandbox (`security_lab.db`) with user, account, and product tables.
- **Exploitation Phase:**
  - **Auth Bypass:** Executed `' OR '1'='1' --` on `/api/vulnerable/login` to bypass password checks and gain admin access.
  - **UNION Extraction:** Injected `' UNION SELECT ... FROM accounts` on `/api/vulnerable/search` to extract confidential account balances and SSN fragments.
  - **Numeric Injection:** Injected `1 OR 1=1` on `/api/vulnerable/user/<id>` to dump all database records.
- **Remediation Phase:**
  - Implemented **Parameterized Queries (Prepared Statements with `?` binding)** to separate data from SQL instructions.
  - Applied **Strict Regular Expression Input Whitelisting** (`^[a-zA-Z0-9_\-]{3,30}$`).
  - Added **Generic Error Masking** to prevent technical database schema leakage.
- **Verification Result:** Automated test harnesses confirmed **100% of attack payloads were neutralized** on remediated code.

```text
Insecure Query Architecture:
User Input ───► Direct String Interpolation ───► SQL Interpreter (VULNERABLE: CWE-89)

Secure Query Architecture:
User Input ───► Regex Validation ───► Parameter Binding (?) ───► Pre-Compiled Query (SECURE)
```

---

### 🛡️ Experiment 02: Integrated Ethical Hacking Assessment
The assessment was executed following the **PTES (Penetration Testing Execution Standard)** and **NIST SP 800-115** guidelines across 7 structured phases:

```text
[1. Reconnaissance] ──► Host & DNS footprinting, technology stack profiling (Python/WSGI)
        │
[2. Port Scanning]  ──► Multi-threaded TCP port scan discovering ports 5000, 5001, and 8080
        │
[3. Enumeration]    ──► Web directory fuzzing uncovering /login, /search, and /user routes
        │
[4. Vuln Analysis]  ──► CVSS v3.1 scoring; triaged 8 confirmed flaws & discarded 4 false positives
        │
[5. Exploitation]   ──► Non-destructive Proof-of-Concepts (Auth bypass, UNION extraction, IDOR)
        │
[6. Remediation]    ──► Deployed HTTP Security Headers (CSP, HSTS, X-Frame-Options) & code patches
        │
[7. Retesting]      ──► Automated regression suite verifying 100% of findings are REMEDIATED
```

---

## 📊 Summary Findings & Remediation Ledger

| Finding ID | Vulnerability Title | Severity | CVSS v3.1 | Status |
| :--- | :--- | :--- | :--- | :--- |
| **VULN-001** | SQL Injection in Authentication Service | **CRITICAL** | 9.8 | **REMEDIATED** |
| **VULN-002** | UNION-Based Data Extraction in Search | **HIGH** | 8.6 | **REMEDIATED** |
| **VULN-003** | Numeric SQL Injection in User Service | **HIGH** | 7.5 | **REMEDIATED** |
| **VULN-004** | Missing HTTP Security Headers (HSTS/CSP) | **MEDIUM** | 6.5 | **REMEDIATED** |
| **VULN-005** | Verbose Server Software Banner Disclosure | **MEDIUM** | 5.3 | **REMEDIATED** |
| **VULN-006** | Unauthenticated Administrative Route Access | **HIGH** | 8.2 | **REMEDIATED** |
| **VULN-007** | Insecure Direct Object Reference (IDOR) | **HIGH** | 7.5 | **REMEDIATED** |
| **VULN-008** | Legacy Unsalted Cryptographic Hash Usage | **MEDIUM** | 5.9 | **REMEDIATED** |

---

## 🔒 Cryptographic Evidence Registry (SHA-256)

| Evidence ID | Phase / Target | Artifact File | Cryptographic SHA-256 Checksum |
| :--- | :--- | :--- | :--- |
| `EV-SQLI-001` | Assignment 1 (Auth Bypass) | `EV-SQLI-001_auth_bypass.md` | `a762251afc33...` |
| `EV-SQLI-002` | Assignment 1 (UNION Extraction) | `EV-SQLI-002_union_extract.md` | `2cc64a5d0f04...` |
| `EV-SQLI-003` | Assignment 1 (Numeric SQLi) | `EV-SQLI-003_blind_boolean.md` | `c5a349d3e0b7...` |
| `EV-RECON-001`| Assignment 2 (Phase 1 Recon) | `EV-RECON-001_whois_dns.md` | `4bc5ebd46d5a...` |
| `EV-SCAN-001` | Assignment 2 (Phase 2 Scan) | `EV-SCAN-001_nmap_full.md` | `44b5ea57dcc7...` |
| `EV-ENUM-001` | Assignment 2 (Phase 3 Enum) | `EV-ENUM-001_dir_fuzzing.md` | `6444ddb362a2...` |
| `EV-EXPLOIT-001`| Assignment 2 (Phase 5 Exploit) | `EV-EXPLOIT-001_poc_auth.md` | `5ff3ef335f5f...` |
| `EV-RETEST-001`| Assignment 2 (Phase 7 Retest) | `EV-RETEST-001_post_patch.md` | `210f14c46635...` |

---

## 📑 Formal PDF & HTML Audit Deliverables

- 📄 **[Assignment 1 PDF Report](reports/Assignment_01_SQL_Injection_Report.pdf)**
- 📄 **[Assignment 2 PDF Report](reports/Assignment_02_Ethical_Hacking_Assessment_Report.pdf)**
- 🌐 **[Final Audit Report (HTML)](reports/final_report.html)**
- 📝 **[Final Technical Report (Markdown)](reports/final_technical_report.md)**

---

## ⚖️ Ethical & Educational Disclaimer

> [!CAUTION]
> All vulnerability demonstrations, tools, proof-of-concepts, and remediation workflows documented in this repository were conducted strictly within an authorized, isolated local sandbox environment for academic research and educational evaluation. Unauthorized testing against third-party systems without prior explicit written permission is illegal.
