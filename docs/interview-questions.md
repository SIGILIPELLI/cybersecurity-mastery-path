---
title: "Cybersecurity Interview Questions and Answers (180 Questions)"
description: "180 Cybersecurity interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the Cybersecurity Mastery Path."
---

# Cybersecurity Interview Questions and Answers

**180 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free Cybersecurity Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Entry

### [01 · What Is Cybersecurity?](level-1/01-what-is-cybersecurity.md)

**Q1.** What do the letters of the CIA triad stand for?

**Answer:** Confidentiality, Integrity, Availability

Every control protects one or more of them.

**Q2.** A ransomware attack encrypts a hospital's records so staff can't read them. Which property is hit most directly?

**Answer:** Availability

Data exists but isn't usable.

**Q3.** Why is a system with no access at all not a good security outcome?

**Answer:** Security is a trade-off with usability; a system nobody can use fails its purpose

Decisions balance protection with usability, cost and speed.

**Q4.** What distinguishes a threat, a vulnerability and a risk?

**Answer:** A threat can exploit a vulnerability, and risk is the likelihood and impact of that happening

Risk combines them with impact.

### [02 · Networking Fundamentals for Security](level-1/02-networking-fundamentals.md)

**Q5.** Which OSI layer do IP addresses and routers operate at?

**Answer:** Layer 3 (Network)

TCP/UDP ports are layer 4.

**Q6.** What does a TCP three-way handshake consist of?

**Answer:** SYN, SYN-ACK, ACK

SYN floods abuse half-open connections.

**Q7.** Why is DNS a security-relevant protocol?

**Answer:** It's often unauthenticated, so spoofing or poisoning can redirect users, and it can be abused for exfiltration

DNSSEC and DoH/DoT help.

**Q8.** How many usable host addresses are in 192.168.1.0/24?

```python
import ipaddress
n=ipaddress.ip_network('192.168.1.0/24')
print(n.num_addresses-2)
```

**Answer:** 254

Minus network and broadcast addresses. Verified by running it.

### [03 · Operating System Security Basics](level-1/03-os-security-basics.md)

**Q9.** What is the principle of least privilege?

**Answer:** Give users and processes only the access they need to do their job

It limits blast radius.

**Q10.** What does the Linux permission string -rwxr-x--- mean for others?

**Answer:** No permissions for others

Owner rwx, group r-x, others ---.

**Q11.** Why shouldn't daily work be done as root/Administrator?

**Answer:** A mistake or malware runs with full power; use sudo/elevation when needed

Separate admin tasks.

**Q12.** What's a benefit of automatic patching?

**Answer:** Closes known vulnerabilities quickly

Unpatched systems are a top breach cause.

### [04 · Cryptography Fundamentals](level-1/04-cryptography-fundamentals.md)

**Q13.** What does this print?

```python
import hashlib
print(hashlib.sha256(b'abc').hexdigest()[:16])
```

**Answer:** ba7816bf8f01cfea

SHA-256 is deterministic: same input, same digest. Verified by running it.

**Q14.** Which is symmetric encryption?

**Answer:** AES, where one shared key encrypts and decrypts

Asymmetric uses key pairs.

**Q15.** Why hash passwords with a slow, salted algorithm?

**Answer:** Salts defeat precomputed tables and slow hashes make guessing expensive

bcrypt, scrypt, Argon2, PBKDF2.

**Q16.** What does a digital signature prove?

**Answer:** Integrity and the signer's identity (authenticity) with non-repudiation

Sign with the private key, verify with the public key.

### [05 · Authentication &amp; Access Control](level-1/05-authentication-access-control.md)

**Q17.** What's the difference between authentication and authorization?

**Answer:** Authentication proves who you are; authorization decides what you may do

AuthN then AuthZ.

**Q18.** Which is a second factor in MFA?

**Answer:** Something you have, such as a TOTP app or hardware key

Factors: know, have, are.

**Q19.** What does RBAC do?

**Answer:** Grants permissions through roles assigned to users

Simplifies least privilege.

**Q20.** Why are SMS codes weaker than hardware keys?

**Answer:** SIM swapping and interception; FIDO2 keys resist phishing

Prefer phishing-resistant MFA.

### [06 · Common Web Vulnerabilities — OWASP Top 10 Overview](level-1/06-owasp-top-10-overview.md)

**Q21.** What does this print?

```python
import sqlite3
c=sqlite3.connect(':memory:'); c.execute('create table u(n text,p text)'); c.execute("insert into u values('ada','secret')")
name="x' OR '1'='1"
bad=c.execute("select count(*) from u where n='"+name+"'").fetchone()[0]
good=c.execute('select count(*) from u where n=?',(name,)).fetchone()[0]
print(bad,good)
```

**Answer:** 1 0

String concatenation lets input change the query; parameters keep it data. Verified by running it.

**Q22.** What is XSS?

**Answer:** Injecting script that runs in other users' browsers, prevented by output encoding and CSP

Encode output for its context.

**Q23.** What does 'Broken Access Control' look like?

**Answer:** A user changing an ID in a URL to see another user's data (IDOR)

Enforce authorization server-side on every request.

**Q24.** Why is the OWASP Top 10 updated periodically?

**Answer:** It reflects real-world breach and vulnerability data

Use it as an awareness baseline.

### [07 · Malware &amp; Social Engineering Basics](level-1/07-malware-social-engineering.md)

**Q25.** What defines phishing?

**Answer:** Deceiving people (often via email) into revealing credentials or running malware

Targets people rather than technology.

**Q26.** What is ransomware?

**Answer:** Malware that encrypts data and demands payment

Backups and segmentation are key defenses.

**Q27.** Which is a red flag in a suspicious email?

**Answer:** Urgency, mismatched sender domain and unexpected attachments or links

Verify via another channel.

**Q28.** What is a worm?

**Answer:** Malware that spreads itself across networks without user action

Viruses need a host file.

### [08 · Security Tools Setup](level-1/08-security-tools-setup.md)

**Q29.** When is it legal to scan a network with nmap?

**Answer:** When you own it or have written authorization

Unauthorized scanning can be illegal.

**Q30.** What's a safe place to practice security skills?

**Answer:** A home lab or VM you own, isolated from other networks

Scope is everything.

**Q31.** What does Wireshark do?

**Answer:** Captures and analyzes network packets

Filter to inspect specific flows.

**Q32.** Why snapshot a lab VM before experiments?

**Answer:** So you can roll back after breaking things

Disposable environments.

### [09 · Incident Response Basics](level-1/09-incident-response-basics.md)

**Q33.** What's the typical IR lifecycle order?

**Answer:** Preparation, detection/analysis, containment, eradication, recovery, lessons learned

NIST SP 800-61.

**Q34.** Why contain before eradicating?

**Answer:** To stop spread and preserve evidence while you work

Isolate affected hosts.

**Q35.** What is a post-incident review for?

**Answer:** Learning and improving controls, blamelessly

Update playbooks.

**Q36.** Why keep an incident timeline?

**Answer:** Accurate records support analysis and accountability

Timestamp every action.

### [10 · Project — Home Lab Security Assessment](level-1/10-project-home-lab-assessment.md)

**Q37.** What shape does a professional assessment take?

**Answer:** Scope, findings, evidence, risk rating and remediation

Even for a home lab.

**Q38.** Why only assess machines you own?

**Answer:** Testing others' systems without permission is unauthorized access

Authorization is the legal basis.

**Q39.** How do you prioritize findings?

**Answer:** By risk: likelihood times impact

Fix the highest first.

**Q40.** How do you verify a hardening change worked?

**Answer:** Re-test and compare with the baseline

Evidence.

## Level 2 · Intermediate

### [01 · Network Security Deep Dive](level-2/01-network-security-deep-dive.md)

**Q41.** What is the default posture a firewall should take?

**Answer:** Default deny: allow only what is needed

Explicit allows.

**Q42.** What's the difference between an IDS and an IPS?

**Answer:** An IDS detects and alerts; an IPS can also block

Inline placement matters.

**Q43.** What does network segmentation do?

**Answer:** Limits lateral movement by separating zones

Contain breaches.

**Q44.** Why capture packets on a lab network?

**Answer:** To see actual traffic instead of trusting summaries

Authorized networks only.

### [02 · Web Application Security Hands-On](level-2/02-web-application-security-hands-on.md)

**Q45.** What does an intercepting proxy let a tester do?

**Answer:** Observe and modify requests between browser and server

e.g. ZAP or Burp against authorized targets.

**Q46.** Where must access control be enforced?

**Answer:** On the server for every request

Client checks are bypassable.

**Q47.** What does a CSP header help prevent?

**Answer:** XSS and data injection by restricting script sources

Defense in depth.

**Q48.** Why use a deliberately vulnerable local app?

**Answer:** To practice legally and safely

Like DVWA or Juice Shop locally.

### [03 · Linux Security Hardening](level-2/03-linux-security-hardening.md)

**Q49.** Which SSH setting reduces brute-force risk?

**Answer:** Disable password login and use keys

Also use fail2ban or rate limits.

**Q50.** What does a minimal install reduce?

**Answer:** Attack surface: fewer packages and services

Remove what isn't needed.

**Q51.** What does `ufw default deny incoming` do?

**Answer:** Blocks inbound connections unless allowed

Allow specific ports.

**Q52.** What does auditd record?

**Answer:** Security-relevant system calls and file access events

Supports detection.

### [04 · Windows Security &amp; Active Directory Basics](level-2/04-windows-security-active-directory.md)

**Q53.** Why is Active Directory a prime target?

**Answer:** Compromising a domain controller yields control of the domain's identities and machines

Protect and monitor DCs.

**Q54.** What is Kerberoasting a risk to?

**Answer:** Service account passwords, so use long random passwords or gMSAs

Defensive awareness.

**Q55.** What does tiered administration do?

**Answer:** Separates admin accounts by trust level to limit credential theft

Admin credentials never touch lower tiers.

**Q56.** What does Group Policy enforce?

**Answer:** Configuration and security settings across domain machines

Central control.

### [05 · Vulnerability Scanning](level-2/05-vulnerability-scanning.md)

**Q57.** What's the vulnerability management lifecycle?

**Answer:** Discover, assess, prioritize, remediate, verify, repeat

It's continuous.

**Q58.** What is a false positive?

**Answer:** A reported vulnerability that doesn't actually exist

Validate findings.

**Q59.** What does CVSS provide?

**Answer:** A severity score for vulnerabilities, to be combined with context

Prioritize by exposure too.

**Q60.** Why use authenticated scans?

**Answer:** They see installed software and config, finding more accurate results

Credentialed scans.

### [06 · SIEM &amp; Log Analysis Basics](level-2/06-siem-log-analysis-basics.md)

**Q61.** What does a SIEM do?

**Answer:** Centralizes logs, correlates events and alerts

Detection platform.

**Q62.** Which log pattern suggests brute forcing?

**Answer:** Many failed logins from one source followed by a success

Write a correlation rule.

**Q63.** What does this print?

```python
fails=['10.0.0.5']*6+['10.0.0.9']*2
from collections import Counter
print([ip for ip,c in Counter(fails).items() if c>=5])
```

**Answer:** ['10.0.0.5']

Threshold-based detection. Verified by running it.

**Q64.** Why normalize logs?

**Answer:** So fields from different sources can be queried uniformly

Parsing is the foundation.

### [07 · Secure Coding Practices](level-2/07-secure-coding-practices.md)

**Q65.** What's the core principle of secure coding?

**Answer:** Never trust input: validate, encode and parameterize

Every trust boundary.

**Q66.** Which is the correct way to avoid SQL injection?

**Answer:** Parameterized queries

Keeps data separate from code.

**Q67.** How should secrets be handled in code?

**Answer:** Kept out of source control in secret stores or environment variables

Rotate if leaked.

**Q68.** What should error messages to users avoid?

**Answer:** Leaking stack traces and internals

Log details server-side.

### [08 · Cloud Security Basics](level-2/08-cloud-security-basics.md)

**Q69.** Who is responsible for configuring a customer's S3 bucket permissions?

**Answer:** The customer (shared responsibility)

Provider secures the infrastructure.

**Q70.** What's the top cloud breach cause?

**Answer:** Misconfiguration, such as publicly exposed storage

Use CSPM and least privilege.

**Q71.** What does IAM least privilege mean?

**Answer:** Grant narrow permissions to identities, avoiding wildcard policies

Review permissions regularly.

**Q72.** Why enable MFA on the root account?

**Answer:** It has unlimited power; protect it and avoid daily use

Use roles instead.

### [09 · Penetration Testing Methodology](level-2/09-penetration-testing-methodology.md)

**Q73.** What makes a penetration test legal?

**Answer:** Written authorization and a defined scope/Rules of Engagement

Without it it's unauthorized access.

**Q74.** What are typical pentest phases?

**Answer:** Reconnaissance, scanning/enumeration, exploitation, post-exploitation, reporting

Methodology, not randomness.

**Q75.** What distinguishes a pentest from a vulnerability scan?

**Answer:** A pentest manually validates and proves impact

Scan lists; pentest demonstrates.

**Q76.** What should a pentest do upon finding critical issues mid-test?

**Answer:** Notify the client per RoE rather than waiting for the report

Communication.

### [10 · Project — Web App Security Assessment](level-2/10-project-web-app-security-assessment.md)

**Q77.** What should each finding in the report include?

**Answer:** Description, evidence, impact, risk rating and remediation

Actionable.

**Q78.** Why test only the legal local target?

**Answer:** Authorization: assess systems you own or are permitted to test

Scope.

**Q79.** How are findings prioritized?

**Answer:** By likelihood and impact, with exploitability context

Risk-based.

**Q80.** What's the value of remediation advice?

**Answer:** It turns a finding into a fix developers can apply

Be specific.

## Level 3 · Advanced

### [01 · Advanced Penetration Testing](level-3/01-advanced-penetration-testing.md)

**Q81.** What does lateral movement mean?

**Answer:** An attacker moving from one compromised system to others inside a network

Segmentation and monitoring reduce it.

**Q82.** Why do testers document every action?

**Answer:** For reproducibility and so the client can validate and remediate

Evidence.

**Q83.** What is privilege escalation?

**Answer:** Gaining higher access than initially granted

Least privilege limits it.

**Q84.** What is persistence?

**Answer:** Maintaining access across reboots or credential changes

Testers clean up what they install.

### [02 · Digital Forensics Basics](level-3/02-digital-forensics-basics.md)

**Q85.** Why make a forensic image and work on the copy?

**Answer:** To preserve the original evidence unaltered

Use write blockers.

**Q86.** What does chain of custody document?

**Answer:** Who handled evidence, when and how, to keep it admissible

Integrity matters.

**Q87.** Why record a hash of evidence?

**Answer:** To prove it hasn't changed

SHA-256 before and after.

**Q88.** What is volatile data?

**Answer:** Data lost at power-off, like RAM; collect it first

Order of volatility.

### [03 · Malware Analysis Basics](level-3/03-malware-analysis-basics.md)

**Q89.** What's the difference between static and dynamic analysis?

**Answer:** Static inspects the sample without running; dynamic observes it executing in a sandbox

Do it in isolation.

**Q90.** Why analyze malware in an isolated VM?

**Answer:** To prevent infection and spread; snapshot and no shared network

Safety.

**Q91.** What are IOCs?

**Answer:** Indicators of compromise, like hashes, domains and IPs

Share them for detection.

**Q92.** Why compute a file hash first?

**Answer:** To identify and search for known samples

Compare with threat feeds.

### [04 · Threat Intelligence](level-3/04-threat-intelligence.md)

**Q93.** What is threat intelligence?

**Answer:** Evidence-based knowledge about adversaries and their methods to inform defense

Actionable context.

**Q94.** What is MITRE ATT&amp;CK?

**Answer:** A knowledge base of adversary tactics and techniques

Map detections to it.

**Q95.** What makes an IOC short-lived?

**Answer:** Attackers change infrastructure easily

TTPs last longer.

**Q96.** What is the Pyramid of Pain idea?

**Answer:** Hurting attackers is harder when you block TTPs than hashes

Prioritize behaviors.

### [05 · Advanced SIEM (Splunk/ELK)](level-3/05-advanced-siem.md)

**Q97.** Which is a correlation use case?

**Answer:** Impossible travel: logins from distant locations in a short time

Combine events.

**Q98.** What does tuning a SIEM rule reduce?

**Answer:** False positives and alert fatigue

Iterate.

**Q99.** Why enrich alerts with context?

**Answer:** To speed triage (asset, user, threat intel)

Analyst efficiency.

**Q100.** What's a detection-as-code benefit?

**Answer:** Version control, review and testing of detection rules

Treat rules like software.

### [06 · Cloud Security Deep Dive](level-3/06-cloud-security-deep-dive.md)

**Q101.** What is a CSPM tool for?

**Answer:** Continuously finding cloud misconfigurations

Posture management.

**Q102.** Why enable cloud audit logging (e.g. CloudTrail)?

**Answer:** To record API activity for detection and forensics

Send to a protected account.

**Q103.** What does separating accounts/projects achieve?

**Answer:** Blast-radius containment and clearer boundaries

Landing zones.

**Q104.** Why encrypt data with customer-managed keys?

**Answer:** Control over access and rotation and auditability

KMS.

### [07 · Container &amp; Kubernetes Security](level-3/07-container-kubernetes-security.md)

**Q105.** Why run containers as non-root?

**Answer:** It limits damage if the container is compromised

Also drop capabilities.

**Q106.** What does image scanning find?

**Answer:** Known vulnerabilities in packages within images

Scan in CI.

**Q107.** What do Kubernetes NetworkPolicies do?

**Answer:** Restrict pod-to-pod traffic

Default deny.

**Q108.** Why not store secrets in image layers?

**Answer:** They're recoverable by anyone with the image

Use secret stores.

### [08 · Red Team vs Blue Team Concepts](level-3/08-red-team-vs-blue-team.md)

**Q109.** What does a red team do?

**Answer:** Emulates adversaries to test detection and response

Blue defends.

**Q110.** What is purple teaming?

**Answer:** Red and blue collaborating to improve detections

Shared learning.

**Q111.** What's a goal of red teaming vs. pentesting?

**Answer:** Test the whole defense (people, process, tech) against objectives

Broader than vulnerabilities.

**Q112.** What should blue teams measure?

**Answer:** Time to detect and respond and coverage

Metrics drive improvement.

### [09 · Compliance &amp; Frameworks](level-3/09-compliance-frameworks.md)

**Q113.** What does compliance not guarantee?

**Answer:** Actual security: it sets a baseline

Compliant isn't equal to secure.

**Q114.** What is ISO 27001?

**Answer:** A standard for an information security management system

Certifiable.

**Q115.** What does PCI DSS cover?

**Answer:** Protection of payment card data

Applies to card handlers.

**Q116.** What do auditors want?

**Answer:** Evidence that controls operate effectively

Document and retain evidence.

### [10 · Project — Full Penetration Test Report](level-3/10-project-full-pentest-report.md)

**Q117.** What does an executive summary do?

**Answer:** States scope, overall risk and key recommendations in plain language

For decision makers.

**Q118.** Why include reproduction steps?

**Answer:** So the client can verify and fix

Evidence.

**Q119.** What should findings be rated by?

**Answer:** A consistent method like CVSS plus business context

Be consistent.

**Q120.** What happens after the report?

**Answer:** Remediation tracking and retesting

Close the loop.

## Level 4 · Master

### [01 · Security Architecture](level-4/01-security-architecture.md)

**Q121.** What is defense in depth?

**Answer:** Layered controls so one failure doesn't compromise everything

Overlap.

**Q122.** What does threat modeling do?

**Answer:** Identifies threats and mitigations early in design (e.g. STRIDE)

Design-stage.

**Q123.** What is 'secure by design'?

**Answer:** Building security into architecture from the start

Cheaper than retrofitting.

**Q124.** What does a trust boundary mark?

**Answer:** Where data crosses levels of trust and must be validated

Controls go there.

### [02 · Advanced Threat Hunting](level-4/02-advanced-threat-hunting.md)

**Q125.** What is threat hunting?

**Answer:** Proactively searching for adversaries that evaded detections, driven by hypotheses

Hypothesis-driven.

**Q126.** What starts a hunt?

**Answer:** A hypothesis, e.g. based on ATT&amp;CK or intel

Then query data.

**Q127.** What should a successful hunt produce?

**Answer:** New detections or hardening

Operationalize.

**Q128.** What data do hunters need?

**Answer:** Rich, retained telemetry (endpoint, network, identity)

Visibility.

### [03 · Zero Trust Architecture](level-4/03-zero-trust-architecture.md)

**Q129.** What is the zero trust principle?

**Answer:** Never trust, always verify, regardless of network location

Verify identity and device.

**Q130.** Which is a zero-trust control?

**Answer:** Continuous authentication with device posture and least-privilege access

Microsegmentation.

**Q131.** Why is a VPN alone not zero trust?

**Answer:** It grants broad network access once connected

Prefer per-app access.

**Q132.** What do policy engines decide?

**Answer:** Access based on identity, device and context

Dynamic.

### [04 · Advanced Cryptography &amp; PKI](level-4/04-advanced-cryptography-pki.md)

**Q133.** What does a certificate authority do?

**Answer:** Signs certificates binding identities to public keys

Trust chain.

**Q134.** Why do private keys need strong protection?

**Answer:** Whoever holds it can impersonate the owner; use HSMs

Rotate and restrict.

**Q135.** What does perfect forward secrecy provide?

**Answer:** Past sessions stay secure even if long-term keys leak later

Ephemeral keys.

**Q136.** Why is rolling your own crypto discouraged?

**Answer:** Subtle mistakes break security; use vetted libraries

Use standard algorithms.

### [05 · Security Automation &amp; SOAR](level-4/05-security-automation-soar.md)

**Q137.** What does SOAR automate?

**Answer:** Repetitive response steps via playbooks and integrations

Speed and consistency.

**Q138.** What should be automated first?

**Answer:** High-volume, low-risk, well-understood tasks

Start small.

**Q139.** Why add human approval for destructive actions?

**Answer:** Automation errors can cause outages

Guardrails.

**Q140.** How do you maintain playbooks?

**Answer:** Version, test and review them like code

Keep them current.

### [06 · Advanced Cloud Security &amp; DevSecOps](level-4/06-advanced-cloud-security-devsecops.md)

**Q141.** What does shift-left security mean?

**Answer:** Finding issues earlier in development (SAST, dependency, IaC scans)

Cheaper fixes.

**Q142.** What does IaC scanning catch?

**Answer:** Insecure infrastructure definitions before deploy

Policy as code.

**Q143.** What does SBOM stand for and help with?

**Answer:** Software Bill of Materials, tracking components for vulnerability response

Supply-chain visibility.

**Q144.** Why gate pipelines on critical findings?

**Answer:** To stop known-bad changes from reaching production

Balance with developer experience.

### [07 · Incident Response at Scale](level-4/07-incident-response-at-scale.md)

**Q145.** What helps coordinate a large incident?

**Answer:** A defined command structure with roles and communication channels

Incident commander.

**Q146.** Why prepare communication templates ahead?

**Answer:** Speed and consistency under pressure, including legal and PR

Pre-approved.

**Q147.** What do tabletop exercises test?

**Answer:** Plans and decision-making without live systems

Find gaps cheaply.

**Q148.** Why define severity levels?

**Answer:** To prioritize response and escalation

Shared language.

### [08 · Building a SOC](level-4/08-building-a-soc.md)

**Q149.** Which are SOC pillars?

**Answer:** People, process and technology

All three.

**Q150.** What does tiering analysts allow?

**Answer:** Routing alerts by complexity and escalating appropriately

L1 triage, L2 investigation, L3 hunting.

**Q151.** What metric reflects SOC effectiveness?

**Answer:** Mean time to detect and respond

Outcome measures.

**Q152.** Why prevent analyst burnout?

**Answer:** Alert fatigue leads to missed detections; tune and rotate

Retention matters.

### [09 · Security Leadership &amp; Risk Management](level-4/09-security-leadership-risk-management.md)

**Q153.** How is risk commonly expressed?

**Answer:** Likelihood times impact, with treatment (mitigate, accept, transfer, avoid)

Decisions belong to business owners.

**Q154.** Why communicate in business terms to executives?

**Answer:** Decisions rest on impact to goals, not technical detail

Translate risk.

**Q155.** What is a risk register?

**Answer:** A tracked list of risks with owners and treatments

Review regularly.

**Q156.** What is risk appetite?

**Answer:** The level of risk an organization is willing to accept

Guides decisions.

### [10 · Capstone — Enterprise Security Program Design](level-4/10-capstone-enterprise-security-program.md)

**Q157.** What does an enterprise security program combine?

**Answer:** Governance, architecture, operations, response and metrics

Holistic.

**Q158.** What should drive priorities?

**Answer:** Risk assessment against business objectives

Risk-based roadmap.

**Q159.** How do you show the program works?

**Answer:** Metrics, exercises and independent assessment

Evidence.

**Q160.** Why plan a roadmap in phases?

**Answer:** Limited resources: deliver high-risk improvements first

Iterate.

## Mixed questions across all levels

**Q161.** A hospital's files are encrypted by malware and staff can't open them. Which CIA property is hit most directly?

**Answer:** Availability

The data exists but isn't usable.

**Q162.** Which description best separates a threat from a vulnerability?

**Answer:** A threat is something that could exploit a weakness; a vulnerability is the weakness itself

Risk combines likelihood and impact.

**Q163.** How many usable hosts are in 10.0.0.0/22?

```python
import ipaddress
print(ipaddress.ip_network('10.0.0.0/22').num_addresses-2)
```

**Answer:** 1022

2^10 addresses minus network and broadcast. Verified by running it.

**Q164.** A user submits `x' OR '1'='1` into a query built by string concatenation versus a parameterized query. What do the two counts show?

```python
import sqlite3
c=sqlite3.connect(':memory:'); c.execute('create table u(n)'); c.execute("insert into u values('ada')")
v="x' OR '1'='1"
print(c.execute("select count(*) from u where n='"+v+"'").fetchone()[0], c.execute('select count(*) from u where n=?',(v,)).fetchone()[0])
```

**Answer:** 1 0

Parameterization keeps input as data. Use it everywhere. Verified by running it.

**Q165.** An IDOR lets a user view /invoice/1002 by changing the number from 1001. What's the fix?

**Answer:** Enforce authorization server-side on each request (check ownership), not just hide links

Broken access control is the top OWASP risk.

**Q166.** Why hash passwords with salted, slow algorithms such as Argon2 or bcrypt?

**Answer:** Salts defeat precomputed tables and slow hashing makes mass guessing costly

Never use fast unsalted hashes.

**Q167.** Phishing-resistant MFA is best implemented with...

**Answer:** FIDO2/WebAuthn hardware keys or passkeys

They bind to the real site.

**Q168.** A user reports a suspicious email. What should the SOC do first?

**Answer:** Check headers and links safely, search for other recipients and block indicators

Contain and hunt.

**Q169.** Which log pattern suggests a successful brute-force attack?

**Answer:** Many failures from one IP followed by a success

Correlation rule.

**Q170.** During an incident, why contain before eradicating?

**Answer:** To stop spread and preserve evidence while you analyze

Isolate affected systems.

**Q171.** When making a forensic copy of a disk, why hash it?

**Answer:** To prove the evidence hasn't changed

Document chain of custody.

**Q172.** A cloud storage bucket is publicly readable by mistake. Who is responsible for that configuration?

**Answer:** The customer, under the shared responsibility model

Misconfiguration is the leading cause of cloud breaches.

**Q173.** A pentest is requested. What must exist before testing starts?

**Answer:** Written authorization with scope and Rules of Engagement

Otherwise it's unauthorized access.

**Q174.** A vulnerability scanner reports a critical CVE on a server not reachable from the internet. How should you prioritize?

**Answer:** Combine CVSS with context: exposure, exploitability and asset value

Risk-based prioritization.

**Q175.** Which is a zero-trust design choice?

**Answer:** Per-request verification of identity and device posture with least-privilege access

Never trust, always verify.

**Q176.** Containers run as root with host mounts. What's the risk and fix?

**Answer:** A compromise can escape to the host; run non-root, drop capabilities and avoid broad mounts

Least privilege for workloads.

**Q177.** A detection rule fires 500 false alerts a day. What do you do?

**Answer:** Tune it with context and thresholds, and measure precision

Alert fatigue hides real attacks.

**Q178.** An executive asks 'are we secure?'. How should you answer?

**Answer:** Express residual risk in business terms with evidence, not a yes or no

Risk management.

**Q179.** Which compliance statement is accurate?

**Answer:** Compliance sets a baseline but doesn't guarantee security

Evidence of effective operation matters.

**Q180.** Why plan a security roadmap in phases?

**Answer:** Resources are limited, so fix highest risks first and show measurable progress

Prioritize by risk.

## Practical coding questions

### Store and verify passwords safely

Implement hash_password() with PBKDF2-HMAC-SHA256 and verify() using a constant-time comparison. Show that the right password verifies, a similar one fails, and that a different salt yields a different hash.

Starter code:

```
import hashlib, hmac

def hash_password(password, salt, rounds=100_000):
    # derive a key with PBKDF2-HMAC-SHA256 and return it as a hex string
    return ""

def verify(password, salt, stored, rounds=100_000):
    # recompute and compare in constant time (hmac.compare_digest)
    return False

salt = bytes.fromhex("00112233445566778899aabbccddeeff")
stored = hash_password("correct horse", salt)
print(len(stored), stored[:16])
print(verify("correct horse", salt, stored))
print(verify("Correct horse", salt, stored))
print(hash_password("correct horse", salt) == hash_password("correct horse", bytes(16)))
```

Expected output:

```text
64 349385d8369097aa
True
False
False
```

??? success "Show a solution"

    ```
    import hashlib, hmac

    def hash_password(password, salt, rounds=100_000):
        return hashlib.pbkdf2_hmac("sha256", password.encode(), salt, rounds).hex()

    def verify(password, salt, stored, rounds=100_000):
        candidate = hash_password(password, salt, rounds)
        return hmac.compare_digest(candidate, stored)

    salt = bytes.fromhex("00112233445566778899aabbccddeeff")
    stored = hash_password("correct horse", salt)
    print(len(stored), stored[:16])
    print(verify("correct horse", salt, stored))
    print(verify("Correct horse", salt, stored))
    print(hash_password("correct horse", salt) == hash_password("correct horse", bytes(16)))
    ```

    A unique random salt per user and a slow key-derivation function make offline guessing expensive, and compare_digest avoids timing leaks. The salt here is fixed only so the output is reproducible. Verified by running the solution.

### Detect a brute-force that succeeded

Parse SSH auth log lines and flag a source IP only when a successful login follows at least `threshold` failures from that IP. Return {ip: failures_before_success}. A separate IP with just two failures must not be flagged.

Starter code:

```
import re
from collections import defaultdict

LOG = """\
Nov 21 10:00:01 sshd[1]: Failed password for root from 203.0.113.7 port 51111 ssh2
Nov 21 10:00:03 sshd[1]: Failed password for admin from 203.0.113.7 port 51112 ssh2
Nov 21 10:00:05 sshd[1]: Failed password for root from 203.0.113.7 port 51113 ssh2
Nov 21 10:00:07 sshd[1]: Failed password for ada from 198.51.100.4 port 40001 ssh2
Nov 21 10:00:08 sshd[1]: Failed password for root from 203.0.113.7 port 51114 ssh2
Nov 21 10:00:09 sshd[1]: Accepted password for root from 203.0.113.7 port 51115 ssh2
Nov 21 10:03:00 sshd[1]: Failed password for ada from 198.51.100.4 port 40002 ssh2
"""

PATTERN = re.compile(r"(?P<status>Failed|Accepted) password for (?P<user>\S+) from (?P<ip>\S+)")

def suspicious(log, threshold=4):
    # count failed logins per source IP; flag an IP only if a successful login
    # follows at least `threshold` failures. Return {ip: failures_before_success}.
    return {}

print(suspicious(LOG))
```

Expected output:

```text
{'203.0.113.7': 4}
```

??? success "Show a solution"

    ```
    import re
    from collections import defaultdict

    LOG = """\
    Nov 21 10:00:01 sshd[1]: Failed password for root from 203.0.113.7 port 51111 ssh2
    Nov 21 10:00:03 sshd[1]: Failed password for admin from 203.0.113.7 port 51112 ssh2
    Nov 21 10:00:05 sshd[1]: Failed password for root from 203.0.113.7 port 51113 ssh2
    Nov 21 10:00:07 sshd[1]: Failed password for ada from 198.51.100.4 port 40001 ssh2
    Nov 21 10:00:08 sshd[1]: Failed password for root from 203.0.113.7 port 51114 ssh2
    Nov 21 10:00:09 sshd[1]: Accepted password for root from 203.0.113.7 port 51115 ssh2
    Nov 21 10:03:00 sshd[1]: Failed password for ada from 198.51.100.4 port 40002 ssh2
    """

    PATTERN = re.compile(r"(?P<status>Failed|Accepted) password for (?P<user>\S+) from (?P<ip>\S+)")

    def suspicious(log, threshold=4):
        fails = defaultdict(int)
        flagged = {}
        for line in log.splitlines():
            m = PATTERN.search(line)
            if not m:
                continue
            ip = m["ip"]
            if m["status"] == "Failed":
                fails[ip] += 1
            elif fails[ip] >= threshold:
                flagged[ip] = fails[ip]
        return flagged

    print(suspicious(LOG))
    ```

    Counting failures per source and alerting on a success after a burst is the classic brute-force correlation rule. In a SIEM you'd also window it by time and enrich it. Verified by running the solution.

---

More free practice: the [Cybersecurity Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
