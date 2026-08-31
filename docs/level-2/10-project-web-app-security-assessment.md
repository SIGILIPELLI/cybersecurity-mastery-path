# 10 · Project — Web App Security Assessment

This capstone project for Level 2 combines everything from this level —
web app testing (Module 2), vulnerability scanning (Module 5), secure
coding review (Module 7), and pentest methodology (Module 9) — into one
end-to-end assessment against a legal local target, delivered as a real
report.

!!! warning "Scope"
    This project targets **OWASP Juice Shop** running locally via Docker,
    on `127.0.0.1` only. This is exactly the same authorization boundary
    used throughout Level 2 — never point any tool below at a system you
    don't own.

## 1. Project scope and objective

You are simulating a real, small-scope engagement: assess Juice Shop's
web application security, produce a professional report a stakeholder
could act on. Deliverables:

1. A signed-style Rules of Engagement document (even though it's a lab)
2. A vulnerability scan report
3. At least 5 manually confirmed findings with reproduction steps
4. A final report with executive summary and remediation guidance

## 2. Step 1 — Rules of Engagement

Write a one-page RoE following the Level 2 Module 9 template. Scope: Juice
Shop running at `http://127.0.0.1:3000`. Out of scope: nothing else on
your machine. Testing window: your own choosing. No destructive testing
(don't attempt to actually delete the Docker container's data via the app
in a way that corrupts your ability to keep testing).

## 3. Step 2 — Reconnaissance and scanning

```bash
docker run --rm -d -p 3000:3000 bkimminich/juice-shop
nmap -sV -p 3000 127.0.0.1
nikto -h http://127.0.0.1:3000
```

Run a ZAP automated scan (Level 2 Module 2) and export the HTML report as
your baseline evidence artifact.

## 4. Step 3 — Manual testing against the OWASP Top 10

Work through each category deliberately, using ZAP as your proxy:

- **A01 Broken Access Control** — attempt IDOR on any endpoint referencing
  an ID (basket items, orders, user profile).
- **A02 Cryptographic Failures** — check if sensitive data (passwords,
  tokens) is transmitted or stored insecurely; inspect JWT tokens issued
  after login (jwt.io can decode, never submit real secrets to third-
  party decoders in a non-lab context).
- **A03 Injection** — test the login form and search box for SQL/NoSQL
  injection patterns.
- **A05 Security Misconfiguration** — check HTTP response headers for
  missing `Content-Security-Policy`, `X-Frame-Options`, verbose error
  pages.
- **A06 Vulnerable Components** — Juice Shop deliberately ships outdated
  dependencies; if you have its source, run `npm audit` against it.
- **A07 Authentication Failures** — test for account enumeration (does
  the login error differ for "user doesn't exist" vs. "wrong password"?),
  weak password policy, missing rate limiting on login attempts.

Juice Shop ships with a public scoreboard of intentional challenges
(`/#/score-board`) — use it to confirm you're finding real, intended
vulnerabilities rather than chasing scanner noise, but write up your
findings in your own words and reproduction steps, not copied from any
solution guide.

## 5. Step 4 — Document each finding

Use the finding template from Level 2 Module 2 for every confirmed issue:

```
Title / Severity (CVSS) / Location / Description / Reproduction /
Impact / Remediation
```

Aim for at least 5 distinct, manually-verified findings across at least 3
different OWASP Top 10 categories — breadth across categories demonstrates
you understand the methodology, not just one lucky find.

## 6. Step 5 — Write the final report

Structure:

```
1. Executive Summary
   - One paragraph: scope, overall risk posture, most critical finding
   - A summary table: Finding | Severity | Category

2. Methodology
   - Tools used, phases followed (reference Level 2 Module 9)

3. Detailed Findings (one section per finding, using the template)

4. Remediation Roadmap
   - Prioritized by severity: what to fix first, second, third
   - Realistic timeline suggestion (Critical: immediate, High: 30 days,
     Medium: 90 days -- matching the pattern from Level 2 Module 5)

5. Appendix
   - Raw scanner output (ZAP report, Nikto output)
```

## 7. Self-review checklist before calling it done

- [ ] RoE document written and dated
- [ ] At least 5 manually confirmed findings, spanning 3+ OWASP categories
- [ ] Every finding has concrete reproduction steps someone else could follow
- [ ] Every finding has a specific, actionable remediation (not just "fix it")
- [ ] Executive summary is understandable to someone non-technical
- [ ] Raw scan output is attached as supporting evidence, not the primary content

## Key terms

| Term | Meaning |
|---|---|
| **Rules of Engagement** | Written scope/authorization document preceding any test |
| **Finding** | A single documented vulnerability with evidence and remediation |
| **Executive summary** | Non-technical overview of risk and top findings for leadership |
| **Remediation roadmap** | Prioritized, timed plan for fixing findings |
| **Account enumeration** | Inferring valid usernames/accounts from differing system responses |

## Deliverable

Produce a single Markdown or PDF report following section 6's structure,
covering your Juice Shop assessment end to end. This report is the
direct template you'll scale up in Level 3 Module 10's full penetration
test report.
