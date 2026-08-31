# 02 · Web Application Security Hands-On

Level 1 Module 6 introduced the OWASP Top 10 conceptually and demonstrated
SQL injection and reflected XSS against a local Flask app. This module goes
deeper into hands-on **defensive** web application testing: how to run a
legal local vulnerable target, how to use a proxy to observe and modify
requests, and how to systematically check for the most common flaw classes
using free, industry-standard tooling — the same tooling defenders and
authorized pentesters use to find issues *before* attackers do.

!!! warning "Authorized targets only"
    Every technique here is demonstrated against **OWASP Juice Shop** or
    **DVWA (Damn Vulnerable Web Application)** running on your own machine
    via Docker, `127.0.0.1` only. Never run a scanner, proxy-intercept, or
    fuzzer against a site you do not own or have written authorization to
    test — doing so is illegal in most jurisdictions (Computer Fraud and
    Abuse Act in the US, Computer Misuse Act in the UK, and equivalents
    elsewhere) regardless of intent.

## 1. Standing up a legal target

```bash
# OWASP Juice Shop -- a deliberately vulnerable modern web app, built for training
docker run --rm -p 3000:3000 bkimminich/juice-shop

# DVWA -- classic vulnerable PHP/MySQL app, covers OWASP Top 10 basics
docker run --rm -p 8080:80 vulnerables/web-dvwa
```

Both are maintained specifically for security training and CTF practice.
Browse to `http://127.0.0.1:3000` (Juice Shop) or `http://127.0.0.1:8080`
(DVWA, set security level to "low" first for module 1 exercises, then raise
it as you progress).

## 2. Intercepting traffic with a proxy

A **web proxy** sits between your browser and the target, letting you see
and edit every request before it's sent — essential for understanding what
a web app actually does, not just what its UI shows you.

**OWASP ZAP** (free, open-source) is the standard choice for this course:

```bash
# Install (macOS)
brew install --cask owasp-zap
```

1. Launch ZAP, configure your browser to proxy through `127.0.0.1:8080`
   (ZAP's default listener).
2. Import ZAP's root CA certificate into your browser so HTTPS traffic can
   be decrypted for inspection (this is why proxy interception on a network
   you don't control, without consent, is a serious violation — you are
   literally breaking TLS trust for that session).
3. Browse the target normally; watch ZAP's **History** tab populate with
   every request/response.
4. Right-click any request → **Open/Resend with Request Editor** to modify
   parameters, headers, or cookies and replay it — this is how you turn "I
   see a parameter" into "I can test what that parameter does."

## 3. Automated scanning (and its limits)

ZAP's **Automated Scan** (Quick Start tab, enter target URL, click Attack)
runs a spider to discover pages, then an active scanner that throws known
attack patterns (SQLi, XSS, path traversal, etc.) at every parameter it
found.

```bash
# headless CLI scan, useful in CI pipelines
docker run --rm -t owasp/zap2docker-stable zap-baseline.py \
  -t http://host.docker.internal:3000 -r zap-report.html
```

**Why automated scanning is necessary but not sufficient:**

| Automated scanners are good at | Automated scanners miss |
|---|---|
| Known signature patterns (reflected XSS, obvious SQLi) | Business-logic flaws (e.g. price manipulation, coupon abuse) |
| Fast coverage across large sites | Multi-step workflows (e.g. an IDOR only exploitable after a specific sequence) |
| Consistent, repeatable baselines | Anything requiring authentication state or CSRF token juggling |
| Producing a paper trail for compliance | Context — a scanner can't judge whether an "info leak" actually matters |

This is why professional penetration testing (Level 2 Module 9) always
pairs automated scanning with manual testing.

## 4. Manually testing the OWASP Top 10 against Juice Shop

Work through these checks manually, using ZAP's request editor to modify
each request:

- **Broken Access Control (A01):** Log in as a low-privilege user, capture
  a request to a resource by ID (e.g. `/api/BasketItems/6`), then change
  the ID to one that isn't yours. If the response returns another user's
  data, that's an **Insecure Direct Object Reference (IDOR)**.
- **Injection (A03):** Try `' OR 1=1--` in the login form's email field, the
  same technique as Level 1 Module 6, now through a proxy so you can see the
  exact request/response instead of a curl one-liner.
- **Security Misconfiguration (A05):** Check response headers for
  `Server`, `X-Powered-By` — do they leak framework/version info an
  attacker could use to look up known CVEs?
- **Vulnerable/Outdated Components (A06):** Run `npm audit` (if you have the
  Juice Shop source) or check the app's declared dependency versions
  against the National Vulnerability Database.

## 5. Reporting what you find — the defender's real deliverable

A finding without a clear writeup is not actionable. Every finding should
include:

```
Title:        Reflected XSS in product search
Severity:     Medium (CVSS 6.1)
Location:     GET /rest/products/search?q=<payload>
Description:  The `q` parameter is reflected into the results page HTML
              without output encoding, allowing arbitrary script execution
              in the victim's browser.
Reproduction: 1. Navigate to /search?q=<script>alert(1)</script>
              2. Observe the alert box fires
Impact:       An attacker could craft a link that steals session cookies
              or performs actions as the victim (session hijacking, CSRF).
Remediation:  Apply output encoding on all reflected parameters (server
              template auto-escaping or a library like DOMPurify on any
              client-rendered content). Add a Content-Security-Policy
              header restricting inline script execution as defense in depth.
```

This format — Title / Severity / Location / Description / Reproduction /
Impact / Remediation — is the backbone of every real pentest report you'll
write in this course (see Level 2 Module 10 and Level 3 Module 10).

## Key terms

| Term | Meaning |
|---|---|
| **Intercepting proxy** | Tool (ZAP, Burp Suite) that sits between client and server to inspect/modify traffic |
| **IDOR** | Insecure Direct Object Reference — accessing another user's data by guessing/changing an identifier |
| **Spider/crawler** | Automated discovery of an app's pages and parameters |
| **Active scan** | Sending crafted attack payloads to test for vulnerabilities |
| **CVSS** | Common Vulnerability Scoring System — standardized 0–10 severity score |
| **Defense in depth** | Layering multiple independent controls so one failure doesn't mean full compromise |

## Exercise

1. Stand up Juice Shop locally via Docker and complete the built-in
   "Score Board" first three challenges (hidden at `/#/score-board` —
   find it yourself as your first reconnaissance exercise).
2. Configure ZAP as your proxy, import its CA cert, and capture a full
   login request/response pair in your History tab.
3. Run ZAP's Automated Scan against Juice Shop and export the HTML report.
4. Manually find one IDOR by changing a numeric ID in a captured request.
5. Write up your IDOR finding using the report format in section 5 above.
