# Week 4 — Threat Intelligence & Reporting

**Platforms Used:** AlienVault OTX, VirusTotal, MITRE ATT&CK  

---

## 1. Objective

Collect and analyze threat intelligence from platforms including AlienVault OTX, VirusTotal, and MITRE ATT&CK. Identify ongoing attack trends targeting web applications, document Indicators of Compromise (IOCs), and provide recommended defense strategies based on real-world threat data.

---

## 2. Methodology

| Step | Action |
|------|--------|
| 1 | Searched AlienVault OTX for active threat pulses related to web application attacks |
| 2 | Identified UAC-0252 pulse — active campaign using XSS and social engineering (Jan 2026) |
| 3 | Extracted IOCs: domains and file hashes from the pulse indicators tab |
| 4 | Verified IOC (`security.digital-ua.digital`) on VirusTotal — confirmed malicious by 14+ vendors |
| 5 | Mapped attack techniques to MITRE ATT&CK framework TTPs |
| 6 | Documented findings with defense recommendations |

---

## 3. Threat Overview — UAC-0252 Campaign

### 3.1 Threat Summary

| Field | Details |
|-------|---------|
| Threat Actor | UAC-0252 |
| Campaign Name | SHADOWSNIFF & SALATSTEALER Campaign |
| First Observed | January 2026 |
| Source | CERT-UA / AlienVault OTX Pulse |
| Target | Government & military sector (Ukraine) |
| Attack Vector | Social Engineering + XSS + Malicious Executables |
| Malware Families | SHADOWSNIFF (stealer), SALATSTEALER (stealer) |
| TLP | Green — Limited sharing within community |

### 3.2 Attack Flow

The UAC-0252 threat group carries out targeted attacks using a multi-stage approach:

1. Attackers impersonate representatives from central government authorities via email or messaging
2. Victims are urged to update mobile applications commonly used in civilian and military sectors
3. Malicious communications include archives containing executable files OR links to legitimate-looking websites
4. Those websites contain XSS vulnerabilities — when visited, JavaScript executes and downloads malicious executables
5. Payloads include SHADOWSNIFF and SALATSTEALER — credential and data stealers
6. Stolen data is exfiltrated to attacker-controlled C2 infrastructure

---

## 4. Indicators of Compromise (IOCs)

### 4.1 Malicious Domain — VirusTotal Verified

| IOC Type | Indicator | Verdict | Detected By |
|----------|-----------|---------|-------------|
| Domain | `security.digital-ua.digital` | MALICIOUS | 16/94 AV Engines |

### 4.2 VirusTotal Analysis — `security.digital-ua.digital`

The domain was submitted to VirusTotal for multi-engine analysis. **16 out of 94 security vendors** flagged this domain as malicious. Last analysis: 1 hour ago. Domain creation: 2 months ago.

![VirusTotal Evidence — security.digital-ua.digital](screenshots/virustotal_evidence.png)

| Security Vendor | Detection | Category |
|-----------------|-----------|----------|
| ADMINUSLabs | Malicious | Confirmed Malicious |
| BitDefender | Malware | Confirmed Malicious |
| Certego | Malicious | Confirmed Malicious |
| Chong Lua Dao | Malicious | Confirmed Malicious |
| CRDF | Malicious | Confirmed Malicious |
| CyRadar | Malicious | Confirmed Malicious |
| ESET | Malware | Confirmed Malicious |
| Forcepoint ThreatSeeker | Malicious | Confirmed Malicious |
| Fortinet | Malware | Confirmed Malicious |
| G Data | Malware | Confirmed Malicious |
| Kaspersky | Malware | Confirmed Malicious |
| Lionic | Malware | Confirmed Malicious |
| Soclookup | Malicious | Confirmed Malicious |
| SOCRadar | Phishing | Confirmed Malicious |
| Sophos | Malware | Confirmed Malicious |
| VIPRE | Malware | Confirmed Malicious |
| Gridinsoft | Suspicious | Flagged |

### 4.3 File Hash IOCs (from OTX Pulse)

| Hash Type | Sample | Verdict |
|-----------|--------|---------|
| MD5 | UAC-0252 malicious payload samples (12 hashes) | Malicious |
| SHA1 | UAC-0252 malicious payload samples (4 hashes) | Malicious |
| SHA256 | UAC-0252 malicious payload samples (12 hashes) | Malicious |

---

## 5. MITRE ATT&CK Mapping

| Technique ID | Technique Name | Tactic | Observed In Campaign |
|--------------|----------------|--------|----------------------|
| T1566 | Phishing | Initial Access | Impersonation emails with malicious attachments |
| T1190 | Exploit Public-Facing Application | Initial Access | XSS in linked websites triggers download |
| T1059.007 | JavaScript | Execution | JS executes on page visit to download payload |
| T1027 | Obfuscated Files or Information | Defense Evasion | Payloads disguised as legitimate app updates |
| T1555 | Credentials from Password Stores | Credential Access | SHADOWSNIFF steals stored credentials |
| T1041 | Exfiltration Over C2 Channel | Exfiltration | Stolen data sent to attacker C2 infrastructure |
| T1071 | Application Layer Protocol | Command & Control | C2 communication over standard HTTP/HTTPS |

---

## 6. Connection to Week 2 Findings

This real-world campaign directly mirrors the vulnerabilities identified and exploited during Week 2 lab testing:

| Week 2 Lab Finding | Real-World Equivalent | Threat Actor |
|--------------------|-----------------------|--------------|
| XSS in search/feedback form | XSS on government-linked sites used to trigger downloads | UAC-0252 |
| Broken Authentication | Credential stealing via SHADOWSNIFF stealer | UAC-0252 |
| CSRF via cross-origin form | Social engineering to trigger unintended actions | UAC-0252 |
| No input sanitization | Unsanitized XSS payloads executing JS in victims' browsers | UAC-0252 |

---

## 7. Defense Recommendations

### 7.1 For Organizations

- Deploy Web Application Firewalls (WAF) to detect and block XSS payloads in real time
- Implement strict Content Security Policy (CSP) headers to prevent inline JavaScript execution
- Train employees to identify phishing and social engineering attempts — especially impersonation of government authorities
- Use email filtering and sandboxing to detect and quarantine malicious attachments before delivery
- Block known malicious domains at the DNS/firewall level using threat intelligence feeds
- Enforce multi-factor authentication (MFA) to reduce impact of credential theft

### 7.2 For Developers

- Sanitize all user input and encode output — use DOMPurify client-side, server-side validation for backend
- Implement CSRF tokens on all state-changing requests
- Use `SameSite=Strict` on all session cookies
- Regularly scan application endpoints for XSS and injection vulnerabilities using DAST tools
- Keep all dependencies updated to patch known CVEs that threat actors exploit

### 7.3 IOC-Based Blocking

- Block domain `security.digital-ua.digital` at DNS and firewall level immediately
- Add UAC-0252 file hashes to EDR blocklists for automatic quarantine
- Subscribe to CERT-UA intelligence feed for ongoing UAC-0252 campaign updates

---

## 8. Weekly Threat Intelligence Summary

| # | Threat | Severity | Status |
|---|--------|----------|--------|
| 1 | UAC-0252 XSS Campaign | HIGH | Active — Jan 2026 onwards |
| 2 | SHADOWSNIFF Stealer | HIGH | Active — Credential theft |
| 3 | SALATSTEALER Stealer | HIGH | Active — Data exfiltration |
| 4 | Malicious Domain IOC | CRITICAL | Confirmed by 16/94 AV vendors |

---

## 9. Conclusion

This week's threat intelligence exercise demonstrated the practical application of OSINT-based threat research. Using AlienVault OTX, a real and active threat campaign (UAC-0252) was identified that directly leverages the same attack techniques practiced in Week 2 — specifically Cross-Site Scripting (XSS) and social engineering for initial access.

The VirusTotal analysis of the domain `security.digital-ua.digital` confirmed its malicious nature with detection by 14+ security vendors including Kaspersky, BitDefender, ESET, and Fortinet. Mapping the campaign to MITRE ATT&CK provided a structured understanding of the attacker's tactics, techniques, and procedures (TTPs).

This exercise demonstrates the complete threat intelligence lifecycle:

**Collection (OTX) → Verification (VirusTotal) → Contextualization (MITRE ATT&CK) → Reporting & Defense**

---

*Report prepared by: Ihsan Anwar*  
*Tools: AlienVault OTX, VirusTotal, MITRE ATT&CK*  
*Week 4 — Threat Intelligence & Reporting*
