---
title: "Cybersecurity Cheat Sheet: Quick Reference, Beginner to Advanced"
description: "A free Cybersecurity cheat sheet: 25 quick-reference tables from every level of the Cybersecurity Mastery Path, each linked to its full lesson."
---

# Cybersecurity Cheat Sheet

Quick-reference tables from **25 lessons** of the free Cybersecurity Mastery Path, in course order from beginner to advanced. Each section links to the full lesson with explanations and runnable examples.

## Level 1 · Entry

### [01 · What Is Cybersecurity?](level-1/01-what-is-cybersecurity.md)

*The CIA triad*

| Property | Meaning | Violated when | Example control |
|---|---|---|---|
| **Confidentiality** | Only authorized parties can read the data | An attacker reads your database, a coworker peeks at your screen | Encryption, access control, need-to-know |
| **Integrity** | Data is accurate and unaltered except by authorized action | A file is tampered with, a transaction amount is changed in transit | Hashing, digital signatures, checksums, version control |
| **Availability** | Authorized users can access the system when they need to | A DDoS attack takes down a website, ransomware encrypts your files | Redundancy, backups, rate limiting, DDoS protection |

### [02 · Networking Fundamentals for Security](level-1/02-networking-fundamentals.md)

*The OSI model, security-relevant version*

| Layer | Name | What it does | Security relevance |
|---|---|---|---|
| 7 | Application | HTTP, DNS, SMTP — what the user-facing protocol does | Most web app vulnerabilities (Module 6) live here |
| 4 | Transport | TCP/UDP — reliable vs. best-effort delivery, ports | Port scanning, firewalls, segmentation |
| 3 | Network | IP — addressing and routing between networks | IP spoofing, routing attacks, subnetting for segmentation |
| 2 | Data Link | MAC addresses, switches, local segment delivery | ARP spoofing, VLAN hopping |
| 1 | Physical | Cables, radio, physical medium | Wiretapping, rogue access points |

### [03 · Operating System Security Basics](level-1/03-os-security-basics.md)

*Users, groups, and the principle of least privilege*

| Bad practice | Why it violates least privilege | Better practice |
|---|---|---|
| Everyone logs in as root/Administrator daily | Any mistake or compromised process has full system control | Use a standard account; elevate only when needed (`sudo`, UAC) |
| A web server process runs as root | A vulnerability in the web app becomes a full system compromise | Run it as a dedicated, unprivileged service account |
| One shared admin password for the whole team | No accountability, can't revoke access for one person | Individual accounts, each granted only what's needed |

### [04 · Cryptography Fundamentals](level-1/04-cryptography-fundamentals.md)

*Symmetric encryption — one shared key*

| Algorithm | Key size | Status |
|---|---|---|
| **AES** (Advanced Encryption Standard) | 128/192/256-bit | Current standard — use this |
| DES | 56-bit | Broken (brute-forceable), do not use |
| 3DES | 112-bit effective | Deprecated, being phased out everywhere |
| RC4 | Variable | Broken, do not use |

### [05 · Authentication &amp; Access Control](level-1/05-authentication-access-control.md)

*Authentication vs. authorization — the distinction that matters*

| | Authentication | Authorization |
|---|---|---|
| Question answered | "Who are you?" | "What are you allowed to do?" |
| Happens | First, at login | After authentication, on every action |
| Example | Entering a correct password | Being allowed to view *this specific* invoice |
| Failure mode | Someone impersonates another identity | An authenticated user does something they shouldn't be able to |

### [06 · Common Web Vulnerabilities — OWASP Top 10 Overview](level-1/06-owasp-top-10-overview.md)

*The OWASP Top 10 (2021 edition), briefly*

| # | Category | In short |
|---|---|---|
| A01 | **Broken Access Control** | An authenticated user can act beyond their intended permissions (Module 5) |
| A02 | **Cryptographic Failures** | Sensitive data exposed due to weak/missing encryption (Module 4) |
| A03 | **Injection** | Untrusted input is interpreted as code/commands (SQL injection, this module) |
| A04 | **Insecure Design** | The flaw is architectural, not a coding bug — no amount of patching fixes a design that never considered abuse |
| A05 | **Security Misconfiguration** | Default credentials, verbose error messages, unnecessary features left enabled |
| A06 | **Vulnerable and Outdated Components** | Using a library/framework with a known, unpatched vulnerability |
| A07 | **Identification and Authentication Failures** | Weak login/session mechanisms (Module 5) |
| A08 | **Software and Data Integrity Failures** | Trusting unsigned code/updates/CI pipelines |
| A09 | **Security Logging and Monitoring Failures** | Attacks go undetected because nobody's watching (Module 9) |
| A10 | **Server-Side Request Forgery (SSRF)** | Tricking a server into making requests on the attacker's behalf |

### [07 · Malware &amp; Social Engineering Basics](level-1/07-malware-social-engineering.md)

*Malware categories*

| Type | Behavior | Example impact |
|---|---|---|
| **Virus** | Attaches to a host file/program, spreads when that file is run/shared | Corrupts or destroys files it infects |
| **Worm** | Self-replicates across a network with no human action needed | Can spread globally in hours (e.g., WannaCry via the SMB vulnerability from Module 2) |
| **Trojan** | Disguised as legitimate software; does something different from what it claims | Opens a backdoor while appearing to be a useful utility |
| **Ransomware** | Encrypts the victim's files, demands payment for the decryption key | Business-halting; modern variants also exfiltrate data first ("double extortion") |
| **Spyware** | Covertly monitors activity — keystrokes, screen, browsing | Credential theft, surveillance |
| **Adware** | Displays unwanted advertising, often bundled with legitimate-looking software | Usually more nuisance than damage, but can be a delivery vector for worse |
| **Rootkit** | Hides its own presence and other malware's, often at the OS/kernel level | Extremely hard to detect and remove once installed |
| **Botnet malware** | Turns the infected machine into a remotely controlled "bot" in a larger network | Used for DDoS attacks, spam, further malware distribution |

### [08 · Security Tools Setup](level-1/08-security-tools-setup.md)

*Set up a lab VM*

| Hypervisor | Platform | Notes |
|---|---|---|
| **VirtualBox** | Windows/macOS/Linux, free | Simplest starting point for this course |
| **VMware Workstation Player** | Windows/Linux, free for personal use | Slightly better performance than VirtualBox on some hosts |
| **UTM** | macOS (especially Apple Silicon) | Native ARM virtualization, free |

### [09 · Incident Response Basics](level-1/09-incident-response-basics.md)

*The incident response lifecycle*

| Phase | Goal | Typical activities |
|---|---|---|
| **1. Preparation** | Be ready before anything happens | IR plan, contact lists, tooling, backups, tabletop exercises |
| **2. Detection & Analysis** | Notice something happened and understand it | Alert triage, log review, confirming a true positive |
| **3. Containment** | Stop it from getting worse | Isolate affected systems, disable compromised accounts, block malicious IPs |
| **4. Eradication** | Remove the actual cause | Delete malware, close the vulnerability that was exploited, remove attacker footholds |
| **5. Recovery** | Return to normal operation | Restore from clean backups, rebuild systems, verify integrity before reconnecting |
| **6. Lessons Learned** | Improve for next time | Post-incident report, root cause, process/control changes |

### [10 · Project — Home Lab Security Assessment](level-1/10-project-home-lab-assessment.md)

*Findings*

| Field | Value |
|---|---|
| Description | Two ports beyond the expected minimal baseline are open and listening: 88 (Kerberos) and 445 (SMB/microsoft-ds) |
| Risk rating | Medium — port 445 (SMB) is a well-documented target for lateral movement and worm-style propagation (Module 7's WannaCry example); exposure risk depends heavily on whether this interface is reachable from outside the loopback/LAN |
| Recommendation | Confirm what service is actually bound to these ports and why (`lsof -i :445` on macOS/Linux); disable file sharing services not in active use; ensure the host firewall blocks inbound connections to these ports from any untrusted network |

## Level 2 · Intermediate

### [01 · Network Security Deep Dive](level-2/01-network-security-deep-dive.md)

*Stateful vs. stateless firewalls*

| Type | Decision basis | Example |
|---|---|---|
| Stateless | Each packet alone | Classic ACLs on old routers |
| Stateful | Connection state table | iptables `-m state`, AWS Security Groups |
| Application-layer (NGFW/WAF) | Payload content, not just headers | Blocks a SQLi pattern even on an "allowed" port 443 |

### [02 · Web Application Security Hands-On](level-2/02-web-application-security-hands-on.md)

*Automated scanning (and its limits)*

| Automated scanners are good at | Automated scanners miss |
|---|---|
| Known signature patterns (reflected XSS, obvious SQLi) | Business-logic flaws (e.g. price manipulation, coupon abuse) |
| Fast coverage across large sites | Multi-step workflows (e.g. an IDOR only exploitable after a specific sequence) |
| Consistent, repeatable baselines | Anything requiring authentication state or CSRF token juggling |
| Producing a paper trail for compliance | Context — a scanner can't judge whether an "info leak" actually matters |

### [03 · Linux Security Hardening](level-2/03-linux-security-hardening.md)

*Key terms*

| Term | Meaning |
|---|---|
| **Attack surface** | The sum of everything an attacker could potentially interact with or exploit |
| **Least privilege** | Every account/process gets only the access it strictly needs |
| **MAC (Mandatory Access Control)** | Kernel-enforced policy restricting process behavior beyond file permissions (AppArmor, SELinux) |
| **SUID/SGID** | File permission bits that run a binary as its owner/group rather than the caller — a common privilege-escalation vector if misused |
| **CIS Benchmark** | Industry-standard, vendor-neutral hardening checklist |
| **Unattended upgrades** | Automatic application of security patches without manual intervention |

### [04 · Windows Security &amp; Active Directory Basics](level-2/04-windows-security-active-directory.md)

*Active Directory core concepts*

| Term | Meaning |
|---|---|
| **Domain** | A logical grouping of users/computers under one authority |
| **Domain Controller (DC)** | Server holding the AD database, authenticates every login |
| **Organizational Unit (OU)** | Container for organizing objects and applying Group Policy |
| **Group Policy Object (GPO)** | Centrally-managed configuration pushed to users/computers |
| **Kerberos** | The authentication protocol AD uses (ticket-based, no passwords sent over the wire after initial login) |
| **NTLM** | Older, weaker Windows authentication protocol, still enabled by default in many environments for compatibility |
| **Forest / Trust** | Multiple domains grouped together, optionally with trust relationships between them |

### [05 · Vulnerability Scanning](level-2/05-vulnerability-scanning.md)

*Reading and prioritizing results*

| Factor | Why it matters |
|---|---|
| **CVSS base score** | Theoretical severity in isolation |
| **Exploit availability** | Is there a public, working exploit (check Exploit-DB)? |
| **Exposure** | Internet-facing vs. internal-only, behind a firewall |
| **Asset criticality** | A critical vuln on a disposable test box ≠ one on the payment server |
| **Compensating controls** | Is it already mitigated by network segmentation, WAF, etc.? |

### [06 · SIEM &amp; Log Analysis Basics](level-2/06-siem-log-analysis-basics.md)

*Log sources that matter most for detection*

| Source | Why it matters |
|---|---|
| **Authentication logs** (`/var/log/auth.log`, Windows Security log) | Every login attempt, success or failure |
| **Firewall/network logs** | Connections allowed/denied, unusual destinations |
| **DNS query logs** | Malware frequently uses DNS for command-and-control; unusual domains stand out |
| **Web server logs** | Requests, status codes, user agents — reveals scanning/exploitation attempts |
| **EDR/process logs** | What actually ran on an endpoint, parent/child process relationships |

### [07 · Secure Coding Practices](level-2/07-secure-coding-practices.md)

*Key terms*

| Term | Meaning |
|---|---|
| **Trust boundary** | Any point where data crosses from an untrusted source into your system |
| **Allowlist / denylist** | Defining what's explicitly permitted vs. trying to block what's forbidden |
| **Output encoding** | Transforming data for the specific context (HTML/JS/URL) it's rendered into |
| **SAST** | Static Application Security Testing — analyzing source code for vulnerabilities without running it |
| **Supply chain attack** | Compromise introduced via a trusted third-party dependency |
| **Slow hash** | A deliberately computation-expensive hash function (bcrypt/argon2) used for password storage |

### [08 · Cloud Security Basics](level-2/08-cloud-security-basics.md)

*Key terms*

| Term | Meaning |
|---|---|
| **Shared Responsibility Model** | Division of security duties between cloud provider and customer |
| **IAM** | Identity and Access Management — who/what can do what in a cloud account |
| **Security group** | Cloud-native virtual firewall for an instance/network interface |
| **Bastion host** | A hardened, tightly-controlled single entry point for administrative access |
| **CloudTrail** | AWS's API activity logging service (Azure Activity Log, GCP Audit Logs are equivalents) |
| **Public bucket exposure** | Cloud storage misconfigured to allow unauthenticated public access |

### [09 · Penetration Testing Methodology](level-2/09-penetration-testing-methodology.md)

*Key terms*

| Term | Meaning |
|---|---|
| **Rules of Engagement (RoE)** | Signed document defining scope, authorization, and constraints before testing |
| **Reconnaissance** | Information gathering, passive or active, before scanning/exploitation |
| **Enumeration** | Extracting detailed, actionable information about discovered services |
| **Exploitation** | Proving a vulnerability is real by actually exploiting it, with evidence |
| **Post-exploitation** | Assessing what an attacker could reach/do from a foothold |
| **PTES** | Penetration Testing Execution Standard — a widely used methodology framework |

### [10 · Project — Web App Security Assessment](level-2/10-project-web-app-security-assessment.md)

*Key terms*

| Term | Meaning |
|---|---|
| **Rules of Engagement** | Written scope/authorization document preceding any test |
| **Finding** | A single documented vulnerability with evidence and remediation |
| **Executive summary** | Non-technical overview of risk and top findings for leadership |
| **Remediation roadmap** | Prioritized, timed plan for fixing findings |
| **Account enumeration** | Inferring valid usernames/accounts from differing system responses |

## Level 3 · Advanced

### [01 · Advanced Penetration Testing](level-3/01-advanced-penetration-testing.md)

*Key terms*

| Term | Meaning |
|---|---|
| **Privilege escalation** | Gaining higher-level access than initially obtained |
| **Lateral movement** | Moving from one compromised host to another within a network |
| **Pivoting** | Using a compromised host as a relay to reach otherwise-unreachable networks |
| **Attack chain / kill chain** | The full sequence of steps from initial access to ultimate objective |
| **Password reuse** | Using the same credential across multiple systems/accounts |
| **SOCKS proxy** | A relay allowing arbitrary traffic to be routed through a compromised host |

### [02 · Digital Forensics Basics](level-3/02-digital-forensics-basics.md)

*Windows artifacts every analyst should know*

| Artifact | What it shows |
|---|---|
| Prefetch (`C:\Windows\Prefetch`) | Programs that were executed, and when |
| `$MFT` (Master File Table) | File creation/modification even after deletion |
| Registry `NTUSER.DAT`, `SYSTEM`, `SAM` | User activity, USB history, installed software |
| Event Logs (Security, System) | Logons, service starts, policy changes |
| ShimCache / AmCache | Historical evidence of executed binaries |

### [04 · Threat Intelligence](level-3/04-threat-intelligence.md)

*Strategic, operational, and tactical intelligence*

| Level | Audience | Example |
|---|---|---|
| Strategic | Executives, board | "Ransomware targeting our industry rose 40% this quarter" |
| Operational | SOC managers, IR leads | "Group X is using phishing lures referencing invoice payments" |
| Tactical | SOC analysts, engineers | IOCs: specific hashes, IPs, domains, YARA/Sigma rules |

### [08 · Red Team vs Blue Team Concepts](level-3/08-red-team-vs-blue-team.md)

*Purple team debrief structure*

| ATT&CK Technique | Detected? | Time to Detect | Gap / Fix |
|---|---|---|---|
| T1566.001 Phishing | Yes | 4 min (email gateway) | none |
| T1059.001 PowerShell | No | — | Add Sigma rule for EncodedCommand |
| T1053.005 Scheduled Task | Yes | 22 min (manual review) | Automate detection, currently manual only |
| T1021.002 Lateral movement | No | — | No SMB logon monitoring — add rule |

### [09 · Compliance &amp; Frameworks](level-3/09-compliance-frameworks.md)

*Major frameworks at a glance*

| Framework | Focus | Typical audience |
|---|---|---|
| **NIST CSF 2.0** | Risk-based, voluntary, broadly applicable | Any organization |
| **ISO/IEC 27001** | Certifiable Information Security Management System | Global, especially enterprise/vendor requirements |
| **PCI DSS** | Cardholder data protection | Anyone storing/processing/transmitting card data |
| **HIPAA** | Protected health information | US healthcare and business associates |
| **SOC 2** | Trust service criteria (security, availability, etc.) | SaaS/service providers, driven by customer demand |
| **GDPR** | Personal data of EU residents | Any org handling EU resident data, regardless of location |

---

Go deeper with the full [Cybersecurity Mastery Path](index.md) curriculum or test yourself with the [interview questions](interview-questions.md).
