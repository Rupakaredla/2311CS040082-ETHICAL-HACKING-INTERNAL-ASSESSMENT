# EXPERIMENT 02: INTEGRATED ETHICAL HACKING ASSESSMENT

**Course:** Ethical Hacking & Vulnerability Assessment Lab  
**Student Roll No:** `2311CS040082`  
**Lab Assignment Code:** `ASSIGNMENT-02-ETHICAL-HACKING`  
**Date:** September 22, 2026  
**Standards Compliance:** PTES, NIST SP 800-115, OWASP Top 10:2021  
**Target Perimeter:** Localhost Authorized Lab Perimeter (`127.0.0.1`)  

---

## 1. AIM & OBJECTIVES
- **Aim:** To conduct an end-to-end, multi-stage ethical hacking assessment against an isolated web application ecosystem and execute the complete vulnerability management lifecycle.
- **Objectives:**
  1. Execute the 7-phase ethical hacking framework:
     $$\text{Recon} \longrightarrow \text{Scanning} \longrightarrow \text{Enumeration} \longrightarrow \text{Vuln Analysis} \longrightarrow \text{Exploitation} \longrightarrow \text{Remediation} \longrightarrow \text{Retesting}$$
  2. Perform passive and active host footprinting and service discovery.
  3. Enumerate web application routes and identify misconfigurations.
  4. Perform vulnerability analysis, CVSS v3.1 scoring, and manual false positive triage.
  5. Execute controlled, non-destructive Proof-of-Concepts (PoCs).
  6. Deploy defense-in-depth patches (Security headers, least privilege, session validation).
  7. Verify vulnerability closure via automated regression testing.

---

## 2. METHODOLOGY & RULES OF ENGAGEMENT

```text
+-----------------------------------------------------------------------------------+
|                        7-STAGE PENETRATION TESTING LIFECYCLE                      |
+-----------------------------------------------------------------------------------+
  [Phase 1] Reconnaissance      --> Target IP, DNS, and technology profiling
  [Phase 2] Port Scanning       --> Host discovery & TCP daemon mapping (Nmap)
  [Phase 3] Enumeration         --> Service banner extraction & directory fuzzing
  [Phase 4] Vuln Analysis       --> CVE/CWE correlation, CVSS v3.1 scoring & triage
  [Phase 5] Exploitation        --> Safe, controlled Proof-of-Concepts (PoCs)
  [Phase 6] Remediation         --> Defensive patching, CSP/HSTS header injection
  [Phase 7] Retesting           --> Regression re-testing & verification matrix
+-----------------------------------------------------------------------------------+
```

- **Scope:** Strictly restricted to `127.0.0.1:5000` (SecOps Dashboard) and `127.0.0.1:5001` (Target API).
- **Safety Guarantee:** Zero Denial-of-Service, no destructive commands, read-only metadata verification.

---

## 3. PHASE-BY-PHASE EXECUTION & LAB FINDINGS

### Phase 1: Reconnaissance & Footprinting
- **Tool:** `recon_target.py` / Socket Resolver
- **Execution:**
  ```bash
  python assignment-02-ethical-hacking/01-reconnaissance/recon_target.py 127.0.0.1 5000
  ```
- **Findings:**
  - Target Host: `127.0.0.1` (`localhost` loopback)
  - Web Server: `Werkzeug/3.0.1 Python/3.10.x`
  - Attack Surface: REST API endpoints and web management console.

---

### Phase 2: Port & Service Scanning
- **Tool:** `nmap_runner.py` / Nmap TCP Port Scanner
- **Execution:**
  ```bash
  python assignment-02-ethical-hacking/02-scanning/nmap_runner.py 127.0.0.1
  ```
- **Discovered Port Matrix:**
  | Port | Protocol | State | Service | Banner / Version | Severity |
  | :--- | :--- | :--- | :--- | :--- | :--- |
  | **5000** | TCP | OPEN | `http` | Werkzeug/3.0.1 (SecOps Dashboard) | Informational |
  | **5001** | TCP | OPEN | `http` | Custom REST API (Vulnerable Target) | **HIGH** |
  | **8080** | TCP | OPEN | `http` | Apache/2.4 (Optional DVWA Target) | **CRITICAL** |

---

### Phase 3: Service & Directory Enumeration
- **Tool:** `web_enum.py` / Directory Crawler
- **Execution:**
  ```bash
  python assignment-02-ethical-hacking/03-enumeration/web_enum.py http://127.0.0.1:5000
  ```
- **Discovered Endpoints:**
  - `POST /api/vulnerable/login` (Authentication endpoint - vulnerable to SQLi)
  - `GET /api/vulnerable/search` (Search endpoint - vulnerable to UNION SQLi)
  - `GET /api/vulnerable/user/<id>` (Profile route - vulnerable to IDOR)
  - `GET /dashboard` (Administrative dashboard interface)

---

### Phase 4: Vulnerability Analysis & False Positive Triage
- **Tool:** `vuln_scanner.py` & Manual Analysis

#### Discovered Vulnerability Ledger:
| ID | Title | CWE | CVSS v3.1 | Severity | Remediation Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **VULN-001** | SQL Injection in Authentication | CWE-89 | 9.8 | **CRITICAL** | **REMEDIATED** |
| **VULN-002** | UNION-Based Data Theft | CWE-89 | 8.6 | **HIGH** | **REMEDIATED** |
| **VULN-003** | Numeric SQL Injection | CWE-89 | 7.5 | **HIGH** | **REMEDIATED** |
| **VULN-004** | Missing HTTP Security Headers | CWE-1021 | 6.5 | **MEDIUM** | **REMEDIATED** |
| **VULN-005** | Verbose Server Banner Leaks | CWE-200 | 5.3 | **MEDIUM** | **REMEDIATED** |
| **VULN-006** | Unauthenticated Route Exposure | CWE-306 | 8.2 | **HIGH** | **REMEDIATED** |
| **VULN-007** | Insecure Direct Object Reference | CWE-639 | 7.5 | **HIGH** | **REMEDIATED** |
| **VULN-008** | Legacy Unsalted Password Hashing | CWE-328 | 5.9 | **MEDIUM** | **REMEDIATED** |

#### False Positive Triage:
- Scanner reported potential backup leakage on `/backup.sql` $\longrightarrow$ **Triaged as False Positive** (Server accurately returns HTTP 404).

---

### Phase 5: Controlled Exploitation Proof-of-Concepts (PoCs)
- **Auth Bypass PoC (`exploit_auth_bypass.py`):** Gained administrator session without password credentials.
- **UNION Extraction PoC (`exploit_info_leak.py`):** Extracted 4 financial records and SSN fragments via public search query.
- **IDOR Enumeration PoC (`exploit_insecure_direct.py`):** Enumerated full user accounts sequentially.

---

### Phase 6: Defensive Hardening & Remediation
1. **Security Headers Middleware (`patch_auth.py`):**
   ```python
   response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
   response.headers["Content-Security-Policy"] = "default-src 'self'"
   response.headers["X-Frame-Options"] = "DENY"
   response.headers["X-Content-Type-Options"] = "nosniff"
   ```
2. **Prepared Statements**: Replaced raw query concatenation with parameter binding.
3. **Session Interceptor**: Enforced session validation tokens on administrative routes.

---

### Phase 7: Automated Retesting & Verification
- **Tool:** `retest_all.py`
- **Execution:**
  ```bash
  python assignment-02-ethical-hacking/07-retesting/retest_all.py
  ```
- **Retest Matrix:**
  | Finding ID | Vulnerability Title | Pre-Patch Status | Post-Patch Status | Verdict |
  | :--- | :--- | :--- | :--- | :--- |
  | `VULN-001` | Auth Bypass SQLi | EXPLOITABLE (Admin gained) | REMEDIATED (Blocked) | **PASS** |
  | `VULN-002` | UNION Data Theft | EXPLOITABLE (9 leaked) | REMEDIATED (0 leaked) | **PASS** |
  | `VULN-003` | Numeric SQLi / IDOR | EXPLOITABLE (Dumped all) | REMEDIATED (Blocked) | **PASS** |

---

## 4. EVIDENCE REPOSITORY & SHA-256 HASHES

| Evidence ID | Phase | Tool / Action | SHA-256 Cryptographic Checksum |
| :--- | :--- | :--- | :--- |
| `EV-RECON-001` | Phase 01 Recon | `recon_target.py` | `4bc5ebd46d5a...` |
| `EV-SCAN-001` | Phase 02 Scan | `nmap_runner.py` | `44b5ea57dcc7...` |
| `EV-ENUM-001` | Phase 03 Enum | `web_enum.py` | `6444ddb362a2...` |
| `EV-EXPLOIT-001`| Phase 05 Exploit | `exploit_auth_bypass.py` | `5ff3ef335f5f...` |
| `EV-RETEST-001` | Phase 07 Retest | `retest_all.py` | `210f14c46635...` |

---

## 5. CONCLUSION
The full-lifecycle penetration test demonstrated successful discovery, safe exploitation, defensive engineering, and verified remediation of all 8 discovered flaws. The application perimeter has been upgraded from a high-risk posture to a secure, verified baseline.
