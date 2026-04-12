# Week 1 — Web Application Reconnaissance Report

**Intern Name:** Ihsan Anwar  
**Date:** April 2026  
**Target Application:** OWASP Juice Shop  
**Target URL:** http://127.0.0.1:3000  
**Tools Used:** Burp Suite Community Edition, FoxyProxy, Manual Browser Inspection  

---

## 1. Objective

Perform reconnaissance on a sample web application using Burp Suite and manual inspection. Map out the application's pages, inputs, and APIs. Identify potential entry points like login forms, query parameters, and file uploads.

---

## 2. Methodology

- Configured FoxyProxy in browser to route traffic through Burp Suite (127.0.0.1:8080)
- Manually browsed all accessible pages of the application
- Monitored HTTP History in Burp Suite to capture all requests and responses
- Identified all input fields, API endpoints, and potential entry points

---

## 3. Pages Discovered

| Page | URL |
|------|-----|
| Home | http://127.0.0.1:3000/#/ |
| Login | http://127.0.0.1:3000/#/login |
| Registration | http://127.0.0.1:3000/#/register |
| Photo Wall | http://127.0.0.1:3000/#/photo-wall |
| Contact/Feedback | http://127.0.0.1:3000/#/contact |
| Product Search | http://127.0.0.1:3000/#/search |

---

## 4. API Endpoints Discovered

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | /rest/user/login | User login |
| GET | /rest/user/whoami | Get current user info |
| GET | /rest/user/whoami?fields=email | Get user email |

### Admin Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /rest/admin/application-configuration | App config |
| GET | /rest/admin/application-version | App version |

### Data Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /rest/products/{id}/reviews | Product reviews |
| GET | /rest/memories/ | User memories/photos |
| GET | /api/Feedbacks/ | Customer feedback |
| POST | /api/Feedbacks/ | Submit feedback |
| GET | /rest/captcha/ | CAPTCHA for forms |
| GET | /rest/languages | Available languages |
| GET | /api/Challenges/ | Challenge list |

### Static Assets
| Path | Description |
|------|-------------|
| /assets/public/images/uploads/ | User uploaded files |
| /assets/public/images/products/ | Product images |

### WebSocket
| Protocol | Endpoint | Description |
|----------|----------|-------------|
| WS | /socket.io/ | Real-time communication |

---

## 5. Entry Points Identified

| # | Entry Point | Location | Type |
|---|-------------|----------|------|
| 1 | Login Form | /#/login | Email + Password fields |
| 2 | Registration Form | /#/register | Name, Email, Password |
| 3 | Search Bar | /#/search | Query parameter ?q= |
| 4 | File Upload | /#/photo-wall | Image upload |
| 5 | Feedback Form | /#/contact | Text input + CAPTCHA |
| 6 | Product Reviews | /rest/products/{id}/reviews | ID parameter |

---

## 6. Interesting Findings

1. **Admin endpoints publicly visible** — `/rest/admin/application-configuration` accessible without authentication check observed
2. **File upload present** — `/assets/public/images/uploads/` contains user uploaded files — potential for malicious file upload
3. **WebSocket active** — Real-time socket connection established on page load
4. **Redirect endpoint** — `/redirect?to=` parameter found — potential open redirect vulnerability
5. **User ID in URL** — `/rest/products/{id}/reviews` — potential IDOR vulnerability

---

## 7. Potential Vulnerabilities (To Test in Week 2)

| # | Vulnerability | Entry Point | OWASP Category |
|---|---------------|-------------|----------------|
| 1 | SQL Injection | Login form | A03: Injection |
| 2 | Broken Access Control | Admin endpoints | A01: Broken Access Control |
| 3 | File Upload Bypass | Photo wall upload | A04: Insecure Design |
| 4 | Open Redirect | /redirect?to= | A10: SSRF |
| 5 | IDOR | /rest/products/{id}/ | A01: Broken Access Control |

---

## 8. Conclusion

Reconnaissance of OWASP Juice Shop successfully completed. Multiple entry points and API endpoints have been identified and documented. Several potentially vulnerable areas have been noted for further testing in Week 2.

---

*Report prepared by: Ihsan Anwar*  
*Tools: Burp Suite Community, FoxyProxy, Manual Testing*
