# Cybersecurity Internship — Weekly Lab Reports

**Intern:** Ihsan Anwar  
**Duration:** April 2026  
**Target Application:** OWASP Juice Shop (http://127.0.0.1:3000)  
**Focus:** Web Application Penetration Testing, Secure Coding, Threat Intelligence  

---

## Repository Structure

```
📁 cybersecurity-internship/
│
├── 📄 Week1_Recon_Report.md           — Reconnaissance & endpoint mapping
├── 📄 Week2_OWASP_Testing_Report.md   — Vulnerability testing & exploitation
├── 📄 Week3_SecureCoding_Report.md    — Patch implementation & re-testing
├── 📄 Week4_ThreatIntelligence_Report.md — Threat intel & IOC analysis
│
├── 📁 Evidence/
│   └── 🖼️ virus total.gif             — VirusTotal scan: security.digital-ua.digital
│
└── 📄 csrf.html                       — CSRF proof-of-concept exploit file
```

---

## Week 1 — Reconnaissance

**Report:** [Week1_Recon_Report.md](Week1_Recon_Report.md)  
**Tools:** Burp Suite Community, FoxyProxy, Manual Browser Inspection  
**Goal:** Map all pages, API endpoints, and entry points of OWASP Juice Shop

### Key Findings

| # | Finding | Details |
|---|---------|---------|
| 1 | Pages Discovered | Login, Register, Search, Feedback, Photo Wall |
| 2 | API Endpoints | `/rest/user/login`, `/api/Feedbacks/`, `/rest/admin/...` |
| 3 | Entry Points | Login form, Search bar, File upload, Feedback form |
| 4 | Admin endpoints exposed | `/rest/admin/application-configuration` accessible |
| 5 | Potential IDOR | `/rest/products/{id}/reviews` — ID in URL |
| 6 | Open Redirect | `/redirect?to=` parameter found |

---

## Week 2 — OWASP Top 10 Vulnerability Testing

**Report:** [Week2_OWASP_Testing_Report.md](Week2_OWASP_Testing_Report.md)  
**Tools:** FoxyProxy, Firefox, Browser DevTools, Manual Payloads  
**Goal:** Exploit 4 OWASP Top 10 vulnerabilities with proof-of-concept payloads

### Vulnerabilities Confirmed

| # | Vulnerability | OWASP Category | Severity | Status |
|---|---------------|----------------|----------|--------|
| 1 | SQL Injection | A03:2021 – Injection | CRITICAL | ✅ CONFIRMED |
| 2 | Cross-Site Scripting (XSS) | A03:2021 – Injection | HIGH | ✅ CONFIRMED |
| 3 | Broken Authentication | A07:2021 – Auth Failures | CRITICAL | ✅ CONFIRMED |
| 4 | CSRF | A01:2021 – Broken Access Control | HIGH | ✅ CONFIRMED |

### Proof-of-Concept Payloads Used

```
SQL Injection:   ' OR 1=1--
Reflected XSS:  <script>alert('XSS')</script>
Image XSS:      <img src=x onerror=alert('XSS')>
CSRF:           csrf.html auto-submits POST from port 3002 → port 3000
Weak Creds:     admin@juice-sh.op / admin123
```

---

## Week 3 — Secure Coding & Patch Implementation

**Report:** [Week3_SecureCoding_Report.md](Week3_SecureCoding_Report.md)  
**Tools:** FoxyProxy, Burp Suite, Browser DevTools, npm packages  
**Goal:** Fix all 4 Week 2 vulnerabilities and re-test to confirm patches

### Patch Summary

| # | Vulnerability | Fix Applied | Result |
|---|---------------|-------------|--------|
| 1 | SQL Injection | Parameterized queries (Sequelize replacements) | ✅ BLOCKED |
| 2 | XSS | DOMPurify sanitization + CSP header | ✅ BLOCKED |
| 3 | Broken Auth | express-rate-limit + strong password regex | ✅ BLOCKED |
| 4 | CSRF | csurf middleware + SameSite=Strict cookie | ✅ BLOCKED |

### npm Packages Installed

```bash
npm install dompurify jsdom          # XSS sanitization
npm install express-rate-limit       # Brute force protection
npm install csurf cookie-parser      # CSRF token middleware
```

---

## Week 4 — Threat Intelligence & Reporting

**Report:** [Week4_ThreatIntelligence_Report.md](Week4_ThreatIntelligence_Report.md)  
**Platforms:** AlienVault OTX, VirusTotal, MITRE ATT&CK  
**Goal:** Collect real-world threat intelligence, document IOCs, map to MITRE ATT&CK

### Threat Identified — UAC-0252 Campaign

| Field | Details |
|-------|---------|
| Threat Actor | UAC-0252 |
| Campaign | SHADOWSNIFF & SALATSTEALER |
| First Observed | January 2026 |
| Target | Government & military sector (Ukraine) |
| Attack Vector | Social Engineering + XSS + Malicious Executables |
| Source | CERT-UA / AlienVault OTX |

### IOC Evidence

| IOC Type | Indicator | Verdict |
|----------|-----------|---------|
| Domain | `security.digital-ua.digital` | MALICIOUS — 16/94 AV engines |

**VirusTotal Scan Evidence:**

![VirusTotal — security.digital-ua.digital](Evidence/virus%20total.gif)

### MITRE ATT&CK Techniques

| Technique ID | Technique | Tactic |
|--------------|-----------|--------|
| T1566 | Phishing | Initial Access |
| T1190 | Exploit Public-Facing Application | Initial Access |
| T1059.007 | JavaScript | Execution |
| T1027 | Obfuscated Files or Information | Defense Evasion |
| T1555 | Credentials from Password Stores | Credential Access |
| T1041 | Exfiltration Over C2 Channel | Exfiltration |
| T1071 | Application Layer Protocol | Command & Control |

---

## Skills Demonstrated

| Skill | Week |
|-------|------|
| Web application reconnaissance | Week 1 |
| API endpoint mapping | Week 1 |
| SQL Injection exploitation | Week 2 |
| XSS (Reflected + Stored) exploitation | Week 2 |
| Authentication bypass | Week 2 |
| CSRF proof-of-concept | Week 2 |
| Parameterized query implementation | Week 3 |
| Input sanitization with DOMPurify | Week 3 |
| Rate limiting with express-rate-limit | Week 3 |
| CSRF token middleware (csurf) | Week 3 |
| OSINT threat research (OTX) | Week 4 |
| IOC verification (VirusTotal) | Week 4 |
| MITRE ATT&CK framework mapping | Week 4 |
| Threat intelligence reporting | Week 4 |

---

## Ethical Statement

All testing documented in these reports was conducted exclusively against OWASP Juice Shop — a deliberately vulnerable application designed for security education. All work was performed in a local, isolated lab environment (127.0.0.1) with no impact on any real-world systems, users, or networks. These reports are produced solely for academic and educational purposes.

---

*Ihsan Anwar — Cybersecurity Internship 2026*
