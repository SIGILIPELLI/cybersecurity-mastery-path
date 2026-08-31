# 09 · Penetration Testing Methodology

A penetration test is an *authorized*, structured simulation of a real
attack, designed to find exploitable weaknesses before a real attacker
does — and to prove impact, not just list findings. This module covers the
standard methodology and phases, applied to a legal lab target.

!!! warning "Authorization is not optional — it's the entire legal basis"
    A penetration test without a signed **Rules of Engagement (RoE)**
    document and explicit written scope is not a penetration test — it is
    unauthorized computer access, a crime, full stop. Every technique
    below is demonstrated only against Metasploitable2/DVWA/Juice Shop
    running in your own local lab.

## 1. The standard methodology (PTES / OWASP-aligned)

```
1. Pre-engagement  -- scope, RoE, authorization, rules on what's off-limits
2. Reconnaissance  -- passive/active information gathering
3. Scanning        -- port/service/vulnerability enumeration
4. Exploitation    -- proving a vulnerability is actually exploitable
5. Post-exploitation -- what could an attacker do once in? (impact)
6. Reporting       -- findings, evidence, risk, remediation
7. Remediation verification -- re-test after fixes are applied
```

This sequence — recon → scan → exploit → post-exploit → report — is the
backbone every framework (PTES, OSSTMM, OWASP Testing Guide, NIST
SP 800-115) organizes around, even when their terminology differs.

## 2. Phase 1: Pre-engagement and scope

A Rules of Engagement document defines, in writing, before any testing
begins:

```
Scope:            IP ranges / domains explicitly in-scope
Out of scope:      Third-party systems, production payment systems, etc.
Testing window:    Dates/times testing is permitted
Contact:           Emergency contact if something breaks
Data handling:     How findings/evidence will be stored and disposed of
Rules:             e.g. "no denial-of-service testing", "social
                   engineering permitted only against listed employees"
Authorization:     Signed by someone with actual authority over the systems
```

This document is what legally distinguishes a penetration test from a
crime — never begin testing without it, in a real engagement.

## 3. Phase 2: Reconnaissance

**Passive** (no direct interaction with the target):

```bash
whois example.com
dig example.com ANY
# theHarvester -- gathers public emails, subdomains, employee names
theHarvester -d example.com -b google,bing
```

**Active** (direct interaction, still pre-exploitation):

```bash
nmap -sn 192.168.56.0/24          # host discovery
nmap -sV -sC 192.168.56.101       # service/version + safe scripts
```

## 4. Phase 3: Scanning and enumeration

```bash
# Enumerate specific services found in recon
nmap -p 21 --script ftp-anon 192.168.56.101      # anonymous FTP allowed?
enum4linux -a 192.168.56.101                     # SMB/NetBIOS enumeration
nikto -h http://192.168.56.101                   # web server misconfig scan
```

Enumeration turns "port 21 is open" into "port 21 runs vsFTPd 2.3.4 and
allows anonymous login" — the specific, actionable detail exploitation
depends on.

## 5. Phase 4: Exploitation

Using Metasploit against the deliberately vulnerable Metasploitable2 lab
target (a known, intentional vulnerability, in an isolated lab only):

```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
run
```

The point of exploitation in a *professional* pentest is never "cause
damage" — it's to convert "this looks vulnerable" into concrete, provable
evidence: did you get a shell, read a file you shouldn't have, or escalate
privilege? Evidence is what makes a finding credible to the people who
have to prioritize fixing it.

## 6. Phase 5: Post-exploitation — measuring real impact

```bash
# Once you have a shell, the question isn't "can I do more damage" --
# it's "what could a real attacker reach from here?"
whoami; id                        # what privilege level did we land at?
cat /etc/passwd                   # what other accounts exist?
ss -tulpn                         # what else is reachable from this host?
```

Post-exploitation defines the actual **business impact** of a finding —
"an anonymous FTP misconfiguration" sounds minor; "an anonymous FTP
misconfiguration that gave us root shell access and a path to the internal
database" is what actually gets budget approved to fix.

## 7. Phase 6: Reporting

Every finding uses the format introduced in Level 2 Module 2, expanded
with an **executive summary** for non-technical stakeholders:

```
Executive Summary:
  Testing identified 3 critical and 5 medium findings. The most severe
  (VSF-01) allowed an unauthenticated attacker to obtain root-level
  access to the target host via an outdated FTP service, from which
  the internal network became fully reachable. Immediate remediation
  of VSF-01 is recommended before any production deployment.

Technical Findings (one per finding):
  Title / CVSS / Affected host / Description / Reproduction steps /
  Evidence (screenshots, command output) / Business impact / Remediation
```

The executive summary is often the *only* section leadership reads —
writing it clearly, in business-impact terms, is what actually gets
findings fixed.

## 8. Phase 7: Remediation verification

A pentest isn't complete when the report is delivered — it's complete
when fixes are verified:

```bash
# Re-run the exact exploit/scan that found the issue originally
msfconsole -x "use exploit/unix/ftp/vsftpd_234_backdoor; set RHOSTS 192.168.56.101; run; exit"
```

If remediation was "patched vsftpd to a fixed version," this should now
fail — confirming closure rather than assuming it.

## Key terms

| Term | Meaning |
|---|---|
| **Rules of Engagement (RoE)** | Signed document defining scope, authorization, and constraints before testing |
| **Reconnaissance** | Information gathering, passive or active, before scanning/exploitation |
| **Enumeration** | Extracting detailed, actionable information about discovered services |
| **Exploitation** | Proving a vulnerability is real by actually exploiting it, with evidence |
| **Post-exploitation** | Assessing what an attacker could reach/do from a foothold |
| **PTES** | Penetration Testing Execution Standard — a widely used methodology framework |

## Exercise

1. Write a one-page Rules of Engagement document for a hypothetical
   internal pentest of your own home lab, following section 2's template.
2. Run passive and active recon (sections 3) against your Metasploitable2
   lab VM and document every finding.
3. Enumerate at least two services with dedicated tools (section 4).
4. Run the vsftpd exploitation walkthrough in section 5 and capture proof
   (shell prompt, `whoami` output) as evidence.
5. Write a two-finding mini-report (section 7 format, including an
   executive summary) covering the vsftpd finding and one enumeration
   finding of your choice.
