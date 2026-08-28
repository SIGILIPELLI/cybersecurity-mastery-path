# 10 · Project — Home Lab Security Assessment

This project pulls together every module in Level 1 into one real
deliverable: a security assessment of a machine you actually own, following
the same shape as a professional assessment — scope, findings, evidence, risk
rating, and remediation — plus a hardening pass you actually carry out and
verify.

!!! warning "Scope: your own machine or lab VM only"
    This assessment targets a machine you own or a lab VM you control — never
    a shared, corporate, or third-party system without explicit written
    authorization. That authorization requirement (Module 2's "written
    authorization" framing, made concrete here) is the same one that governs
    every real penetration test.

## Deliverable structure

Produce one document (Markdown or plain text is fine) with these sections.
Below is a worked example for each section, built from a real scan of a real
machine — use it as a template, then replace every finding with your own.

### 1. Scope and objective

State exactly what's in scope, when the assessment was performed, and why.

```
Target: personal MacBook (macOS), home network
Date: 2026-08-28
Objective: identify unnecessary network exposure and OS hardening gaps
           before this machine is used for further coursework in this program.
Authorization: this is my own personal machine.
```

### 2. Methodology

List what you actually did, referencing the modules/tools used:

```
1. Enumerated listening network services with nmap (Module 8)
2. Reviewed OS account/permission configuration (Module 3)
3. Checked MFA coverage on primary accounts (Module 5)
4. Reviewed patch status (Module 3)
5. Reviewed firewall configuration (Module 2/8)
```

### 3. Findings

Each finding gets: what was found, the evidence, a risk rating, and a
recommendation. Here is a real finding from an actual scan run for this
assessment:

**Finding 1 — Unnecessary network services exposed on loopback/LAN interface**

Evidence (from `nmap -sT -p 1-1024 127.0.0.1`, Module 8):

```
PORT    STATE SERVICE
88/tcp  open  kerberos-sec
445/tcp open  microsoft-ds
```

| Field | Value |
|---|---|
| Description | Two ports beyond the expected minimal baseline are open and listening: 88 (Kerberos) and 445 (SMB/microsoft-ds) |
| Risk rating | Medium — port 445 (SMB) is a well-documented target for lateral movement and worm-style propagation (Module 7's WannaCry example); exposure risk depends heavily on whether this interface is reachable from outside the loopback/LAN |
| Recommendation | Confirm what service is actually bound to these ports and why (`lsof -i :445` on macOS/Linux); disable file sharing services not in active use; ensure the host firewall blocks inbound connections to these ports from any untrusted network |

Use this exact format — description, evidence, risk rating, recommendation —
for every finding you produce. A finding without evidence is an opinion; a
finding without a recommendation isn't actionable.

### 4. Risk rating scale

Define the scale you're using so ratings are consistent and someone else
could reproduce your reasoning (tie this back to Module 1's risk model):

| Rating | Meaning |
|---|---|
| **Critical** | Actively exploitable, severe impact, remediate immediately |
| **High** | Exploitable with moderate effort, significant impact |
| **Medium** | Requires specific conditions to exploit, or impact is limited |
| **Low** | Minor exposure, unlikely to be practically exploited |
| **Informational** | Not a vulnerability itself, but worth documenting |

### 5. Remediation summary

A table mapping each finding to its status after your hardening pass:

| # | Finding | Rating | Status | Action taken |
|---|---|---|---|---|
| 1 | SMB/Kerberos exposed | Medium | Remediated | Confirmed loopback-only exposure; verified host firewall blocks LAN access |
| 2 | *(yours)* | | | |
| 3 | *(yours)* | | | |

## What to actually do

1. **Run the assessment.** Work through Modules 2, 3, 5, and 8's tools and
   checklists against your own machine or lab VM. At minimum:
   - An `nmap` scan of your own machine (Module 8)
   - A check of your OS account/permission hardening (Module 3's checklist)
   - An MFA audit of your key accounts (Module 5's exercise, formalized here)
   - A patch status check (Module 3 section 4)

2. **Produce at least five findings**, each following the format in section 3
   above, with real evidence (actual command output, not hypothetical).
   Include at least one finding from at least three different modules'
   subject matter (e.g., one networking finding, one OS finding, one
   access-control finding) — a real assessment doesn't stop at one category.

3. **Remediate at least three findings** you have the ability to fix
   yourself (closing an unnecessary service, enabling MFA on an account,
   applying pending patches, tightening a file permission). For each, capture
   **before** and **after** evidence — re-run the same check and show the
   difference, exactly as the worked example does.

4. **Write the risk ratings honestly.** Resist the urge to rate everything
   "Critical" — a big part of professional assessment work is calibrating
   severity so the reader can prioritize. Use section 4's scale and justify
   each rating in one sentence.

5. **Finish with a one-paragraph executive summary** at the top of the
   document — written for someone who won't read the details: what was
   assessed, how many findings at each severity, and the overall posture in
   plain language. This is the part a decision-maker actually reads; the
   detailed findings below it are for whoever implements the fixes.

## Exercise

Complete the full assessment document described above and keep it — you will
extend this exact skill set into a full web application security assessment
in Level 2's capstone project, and this document's format (scope, methodology,
findings with evidence, risk ratings, remediation) is the same shape used in
every subsequent project in this program through Level 3's full penetration
test report.
