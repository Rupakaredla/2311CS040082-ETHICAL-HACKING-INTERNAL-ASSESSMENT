# EXPERIMENT: WEB APPLICATION SECURITY ASSESSMENT

**Course:** Ethical Hacking & Web Application Security Lab  
**Student Roll Number:** `2311CS040082`  
**Assessment Title:** Web Application Security Assessment (OWASP Top 10, Authentication Flaws & Security Misconfigurations)  
**Suggested Lab Target:** DVWA (Damn Vulnerable Web Application) / WebGoat / Local Isolated Sandbox  
**Standard Reference:** OWASP Top 10:2021 & OWASP Web Security Testing Guide (WSTG v4.2)  
**Date:** September 22, 2026  

---

## 1. AIM & OBJECTIVES
- **Aim:** To evaluate, detect, exploit, and remediate critical web application security vulnerabilities categorized under the **OWASP Top 10**, focusing specifically on **Authentication Flaws**, **Security Misconfigurations**, **Injection**, and **Broken Access Control** within an authorized sandbox environment (DVWA / Local Lab).
- **Objectives:**
  1. Identify authentication flaws including weak credential policies, credential stuffing susceptibility, and missing rate limiting.
  2. Detect security misconfigurations such as missing HTTP defense headers, verbose server banner disclosures, and default error handling leaks.
  3. Exploit vulnerabilities in a controlled, non-destructive manner to demonstrate real-world impact.
  4. Engineer defensive countermeasures: Modern adaptive password hashing (Bcrypt/Argon2id), HTTP security header middleware, and session access controls.
  5. Validate complete remediation through automated regression testing.

---

## 2. LAB ENVIRONMENT & TARGET PREREQUISITES
- **Target Platform A (Local WSGI/Python Sandbox):** `http://127.0.0.1:5000` / `http://127.0.0.1:5001`
- **Target Platform B (Dockerized DVWA):** `http://localhost:8080` (Run via `docker compose up -d`)
- **Assessment Tools:** Burp Suite / OWASP ZAP, cURL, Python 3.10+ security scripts, Nmap 7.94, Browser DevTools.

---

## 3. THEORETICAL CONCEPTS & OWASP TOP 10 TAXONOMY

```text
+-----------------------------------------------------------------------------------+
|                        OWASP TOP 10 RISK FOCUS AREAS                              |
+-----------------------------------------------------------------------------------+
  [A01:2021] Broken Access Control          --> IDOR & Unrestricted Administrative Routes
  [A02:2021] Cryptographic Failures         --> Unsalted/Weak Credential Hashing
  [A03:2021] Injection                      --> SQL Injection in Authentication & Search
  [A05:2021] Security Misconfiguration      --> Missing CSP, HSTS, X-Frame-Options, Server Banners
  [A07:2021] Auth & Identification Flaws    --> Brute-Force susceptibility, Auth Bypass
+-----------------------------------------------------------------------------------+
```

### Key Security Principles:
1. **Authentication Integrity**: Passwords must be hashed using adaptive, salted algorithms with work factors (Bcrypt / Argon2id). Authentication endpoints must enforce rate-limiting.
2. **Configuration Hardening**: Web servers must strip software identity banners and inject strict security headers to prevent browser-side exploitation.
3. **Principle of Least Privilege**: Access control must be validated on the server for every requested object and administrative action.

---

## 4. STEP-BY-STEP SECURITY TESTING & EXPLOITATION

### Category 1: Identification & Authentication Flaws (OWASP A07)
- **Vulnerability:** Unsalted Hash Storage & Authentication Bypass
- **Target:** `/api/vulnerable/login` / DVWA Brute Force & SQLi Module
- **Testing Method:**
  1. Tested password verification logic with payload `admin' OR '1'='1' --`.
  2. Inspected database schema: Passwords were saved as plain SHA-256 without cryptographic salt.
- **Exploitation Impact:** Unauthenticated attacker bypasses login and gains administrator privileges (`role: admin`).

```http
POST /api/vulnerable/login HTTP/1.1
Host: 127.0.0.1:5001
Content-Type: application/json

{
  "username": "admin' OR '1'='1' --",
  "password": "invalid_password"
}
```
*Result: HTTP 200 OK — Administrative session granted.*

---

### Category 2: Security Misconfigurations (OWASP A05)
- **Vulnerability:** Missing Defensive Security Headers & Verbose Banner Leaks
- **Target:** `http://127.0.0.1:5000` / DVWA HTTP Response Headers
- **Testing Method:**
  ```bash
  python assignment-02-ethical-hacking/04-vulnerability-analysis/vuln_scanner.py http://127.0.0.1:5000
  ```
- **Identified Misconfigurations:**
  - `Server: Werkzeug/3.0.1 Python/3.10.x` (Exposes exact framework version)
  - Missing `Content-Security-Policy` (Vulnerable to Cross-Site Scripting / XSS)
  - Missing `X-Frame-Options` (Vulnerable to UI Redressing / Clickjacking)
  - Missing `Strict-Transport-Security` (Vulnerable to SSL Stripping / Man-in-the-Middle)
  - Missing `X-Content-Type-Options: nosniff` (Vulnerable to MIME Sniffing)

---

### Category 3: Broken Access Control & IDOR (OWASP A01)
- **Vulnerability:** Sequential Insecure Direct Object Reference (IDOR)
- **Target:** `/api/vulnerable/user/<id>`
- **Testing Method:**
  - Authenticated as standard user and sent requests to `/api/vulnerable/user/1`, `/api/vulnerable/user/2`, `/api/vulnerable/user/3`.
  - Server returned other users' confidential email addresses, roles, and profiles without verifying ownership or session authorization.

---

## 5. DEFENSIVE ENGINEERING & REMEDIATION

### A. Remediating Security Misconfigurations (HTTP Security Headers Middleware)
```python
# assignment-02-ethical-hacking/06-remediation/patch_auth.py
@app.after_request
def apply_security_headers(response):
    response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains; preload"
    response.headers["Content-Security-Policy"] = "default-src 'self'; script-src 'self'"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
    response.headers["Server"] = "Hardened-Gateway/2.0"
    return response
```

### B. Remediating Authentication Flaws (Salted Adaptive Hashing & Parameterization)
```python
# assignment-01-sqli/secure/secure_queries.py
import bcrypt
import re

def secure_login(username_input, password_input):
    # 1. Strict regex input whitelisting
    if not re.match(r"^[a-zA-Z0-9_\-]{3,30}$", username_input):
        return {"status": "validation_error", "error": "Invalid format"}
    
    # 2. Parameterized prepared statement
    query = "SELECT id, username, password_hash, role FROM users WHERE username = ?"
    cursor.execute(query, (username_input,))
    user = cursor.fetchone()
    
    # 3. Constant-time salted cryptographic hash verification
    if user and bcrypt.checkpw(password_input.encode('utf-8'), user['password_hash'].encode('utf-8')):
        return {"authenticated": True, "user": dict(user)}
    return {"authenticated": False, "error": "Invalid credentials"}
```

---

## 6. VERIFICATION & RETEST MATRIX

| Vulnerability Category | OWASP Tag | Baseline Test (Pre-Patch) | Verification (Post-Patch) | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Authentication Bypass** | OWASP A07 | Exploitable (Admin gained) | Blocked (HTTP 400/401) | **REMEDIATED** |
| **Missing Security Headers** | OWASP A05 | 6 Headers Missing | All 6 Headers Present | **REMEDIATED** |
| **Server Banner Leak** | OWASP A05 | Exposed Werkzeug/Python | Masked to `Hardened-Gateway` | **REMEDIATED** |
| **IDOR / Access Control** | OWASP A01 | Arbitrary record access | Enforced session authorization | **REMEDIATED** |
| **SQL Injection** | OWASP A03 | Exploitable via UNION/Tautology | Parameterized & neutralized | **REMEDIATED** |

---

## 7. CONCLUSION
Through structured vulnerability discovery in DVWA / Local Lab environments, critical OWASP Top 10 vulnerabilities were identified, exploited under controlled conditions, and hardened using defense-in-depth engineering. The retesting phase verified that all attack vectors were neutralized without degrading legitimate application workflow.
