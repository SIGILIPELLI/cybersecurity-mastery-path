# 01 · What Is Cybersecurity?

Cybersecurity is the practice of **protecting systems, networks, and data from
unauthorized access, disruption, or damage**, while keeping them usable for the
people who are supposed to use them. That last clause is the part beginners
skip and the part that makes security genuinely hard: a system with no access
at all is perfectly secure and perfectly useless. Every real security decision
is a trade-off between protection and usability, cost, and speed.

!!! note "Security is a process, not a product"
    You cannot buy a box that "makes you secure." Firewalls, antivirus, and
    encryption are tools. Security is the ongoing discipline of using them
    correctly, watching for failures, and improving — which is why this course
    spends as much time on concepts and process as on tools.

## 1. The CIA triad

Almost every security control exists to protect one (or more) of three
properties:

| Property | Meaning | Violated when | Example control |
|---|---|---|---|
| **Confidentiality** | Only authorized parties can read the data | An attacker reads your database, a coworker peeks at your screen | Encryption, access control, need-to-know |
| **Integrity** | Data is accurate and unaltered except by authorized action | A file is tampered with, a transaction amount is changed in transit | Hashing, digital signatures, checksums, version control |
| **Availability** | Authorized users can access the system when they need to | A DDoS attack takes down a website, ransomware encrypts your files | Redundancy, backups, rate limiting, DDoS protection |

!!! tip "Use the triad to classify any incident in one line"
    A leaked customer database → confidentiality breach. A defaced website →
    integrity breach. A ransomware attack → primarily an availability breach
    (with a confidentiality angle if data was also stolen — "double
    extortion," now the norm). Being able to name *which* property was
    violated is the first step of any incident report, and you'll use it again
    in Module 9.

Some frameworks extend CIA with **authenticity** (is this really who it claims
to be?) and **non-repudiation** (can the sender later deny sending it?) — both
matter enormously for digital signatures (Module 4) but are usually treated as
refinements of integrity rather than a fourth pillar.

## 2. Threat, vulnerability, and risk

These three words are used interchangeably by beginners and mean precisely
different things to a security professional — and mixing them up is the
single most common mistake in a security report.

| Term | Definition | Example |
|---|---|---|
| **Asset** | Something worth protecting | A customer database, a web server, an employee's laptop |
| **Threat** | A potential cause of harm — an actor or event | A ransomware gang, a disgruntled employee, a flood, a misconfigured cloud bucket left public by mistake |
| **Vulnerability** | A weakness that a threat can exploit | Unpatched software, a weak password policy, an open port with no firewall rule |
| **Risk** | The likelihood and impact of a threat exploiting a vulnerability | "There's a high chance our unpatched web server gets compromised, and the impact would be severe" |

The relationship is often written as a rough formula:

```
Risk ≈ Threat × Vulnerability × Impact
```

This is not arithmetic you can plug numbers into — it's a mental model. If any
factor is zero, the risk is zero: a vulnerability with no threat capable of
reaching it (an air-gapped machine) carries little practical risk; a severe
threat against a system with no vulnerability (fully patched, well-configured)
also carries little risk. Security work is spent reducing whichever factor is
cheapest to reduce: patching (vulnerability), segmenting a network so a threat
can't reach an asset (threat), or backing up data so a successful attack does
less damage (impact).

!!! example "Applying the model"
    **Asset**: a small business's customer order database.
    **Threat**: opportunistic attackers scanning the internet for exposed
    databases.
    **Vulnerability**: the database is exposed to the internet with a default
    admin password.
    **Risk**: very high — an easy-to-execute threat meets an easy-to-exploit
    vulnerability on a valuable asset. The fix (close the port, or at minimum
    change the password and require a VPN) addresses the vulnerability
    directly, and is why "close unnecessary exposure" is nearly always the
    highest-value first move in security.

## 3. Security domains — the map of the field

Cybersecurity is not one skill; it's a collection of specializations that this
program touches on in roughly this order:

| Domain | What it covers | Where in this program |
|---|---|---|
| **Network security** | Protecting data in transit, firewalls, segmentation | Module 2, Level 2 Module 1 |
| **Endpoint / OS security** | Hardening individual machines, patch management | Module 3, Level 2 Module 3–4 |
| **Application security** | Secure coding, finding and fixing vulnerabilities in software | Module 6, Level 2 Modules 2 & 7 |
| **Identity & access management (IAM)** | Who can do what — authentication, authorization | Module 5 |
| **Cryptography** | Protecting confidentiality and integrity mathematically | Module 4, Level 4 Module 4 |
| **Security operations (SecOps)** | Monitoring, detection, log analysis, SIEM | Level 2 Module 6, Level 3 Module 5 |
| **Incident response & forensics** | Handling breaches after they happen | Module 9, Level 3 Module 2 |
| **Governance, risk & compliance (GRC)** | Policy, frameworks, legal/regulatory requirements | Level 3 Module 9 |
| **Offensive security (pentesting/red team)** | Finding weaknesses by simulating attackers — always with authorization | Level 2 Module 9, Level 3 Modules 1 & 8 |
| **Cloud security** | Securing workloads in AWS/Azure/GCP-style environments | Level 2 Module 8, Level 3 Module 6 |

No single person is deeply expert in all ten. Most careers specialize in two
or three while keeping working knowledge of the rest — which is exactly the
breadth this Level 1 is designed to give you before you specialize.

## 4. Who you're defending against

Understanding *why* an attack happens shapes how you defend against it. Not
every threat actor has the same goals, resources, or patience.

| Actor | Motivation | Typical target | Sophistication |
|---|---|---|---|
| **Script kiddies** | Curiosity, bragging rights | Whatever is easy to find with automated scanners | Low — uses existing tools |
| **Cybercriminals** | Financial gain (ransomware, fraud, data theft for resale) | Anyone with money or valuable data | Low to high — increasingly professionalized |
| **Hacktivists** | Ideological or political statement | Organizations aligned with an opposing cause | Variable |
| **Insider threats** | Revenge, financial gain, or simple negligence | Their own employer | Variable, but access is already granted |
| **Nation-state actors (APTs)** | Espionage, sabotage, geopolitical advantage | Governments, critical infrastructure, large corporations | Very high, well-resourced, patient |

!!! tip "The insider threat is the one most teams underestimate"
    A firewall does nothing against someone who already has a valid login. A
    large share of real-world breaches trace back to a legitimate account —
    phished, careless, or malicious — rather than a wall being broken down.
    This is a large part of why Module 5 (least privilege) and Module 9
    (detection) matter as much as perimeter defenses.

## 5. Defense in depth

No single control is perfect, so security is built in layers — if one fails,
another catches the problem. This is the organizing principle behind almost
every architecture you'll see in this program.

```
Internet
   │
   ▼
[ Firewall ]           ← blocks unwanted network traffic
   │
   ▼
[ IDS/IPS ]            ← detects/blocks known attack patterns
   │
   ▼
[ Network segmentation ]  ← limits what a breach can reach
   │
   ▼
[ OS hardening + patching ]  ← reduces exploitable surface on each host
   │
   ▼
[ Application security controls ]  ← input validation, auth checks
   │
   ▼
[ Encryption at rest ]  ← data unreadable even if stolen
   │
   ▼
[ Monitoring + logging ]  ← catches what got through
   │
   ▼
[ Backups + incident response ]  ← recovery when everything else fails
```

A single misconfigured firewall rule is a bad day. A single misconfigured
firewall rule *plus* an unpatched server *plus* no monitoring *plus* no
backups is a company-ending event. Every layer you add is another chance to
catch or contain a problem before it becomes the latter.

## Key terms

| Term | Meaning |
|---|---|
| **Asset** | Anything of value worth protecting |
| **Threat** | A potential cause of harm |
| **Vulnerability** | A weakness a threat can exploit |
| **Risk** | Likelihood × impact of a threat exploiting a vulnerability |
| **CIA triad** | Confidentiality, Integrity, Availability |
| **Attack surface** | The sum of all points where an attacker could try to get in |
| **Defense in depth** | Layering independent controls so no single failure is catastrophic |
| **APT** | Advanced Persistent Threat — a well-resourced, patient adversary, often nation-state |
| **Zero-day** | A vulnerability unknown to the vendor, with no patch available yet |

## Exercise

No tools needed — this is an analysis exercise, and it's the foundation the
rest of the level builds on.

Pick a system you use every day (your phone, your home Wi-Fi router, your
personal email account, or your workplace laptop) and write a short document
covering:

1. **List three assets** it holds or protects (e.g., "photos," "saved
   passwords," "access to my bank app").
2. **For each asset**, name one realistic threat and one realistic
   vulnerability that, together, would put it at risk. Be specific — "hackers"
   is not a threat, "an attacker on the same public Wi-Fi network running a
   packet sniffer" is.
3. **Classify each risk** using the CIA triad — which property would be
   violated if the risk materialized?
4. **Propose one defense-in-depth layer** you could add that you are not
   currently using (this can be as simple as "enable MFA" or "set up automatic
   OS updates").
5. Finally, name which of the ten security domains from section 3 each of your
   three defenses belongs to.

Keep this document — you'll build on the same personal-systems thinking in
Module 8's home lab setup and Module 10's project.
