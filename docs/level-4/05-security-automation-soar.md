# 05 · Security Automation & SOAR

A SOC that manually triages every alert doesn't scale — analyst time is
the scarcest resource in security operations. SOAR (Security
Orchestration, Automation, and Response) automates the repetitive parts
of detection and response so humans focus on judgment calls.

## 1. Orchestration, automation, and response — what each word means

```
Orchestration -- connecting disparate tools (SIEM, EDR, firewall, ticketing,
                 threat intel) so they can act together via APIs
Automation    -- executing predefined steps without human intervention
                 (enrichment, containment actions)
Response      -- the playbook logic that decides what automation to run
                 and when to hand off to a human
```

## 2. A concrete playbook: phishing triage

```
Trigger: user reports a suspicious email via the "Report Phishing" button

1. Automated enrichment:
   - Extract URLs/attachments, detonate in sandbox (Module 3, Level 3's malware analysis)
   - Check sender domain against threat intel feeds (Level 3 Module 4)
   - Check if other users received the same email (search mail logs)

2. Automated decision:
   - If sandbox confirms malicious AND multiple recipients found:
     -> auto-quarantine the email from all mailboxes
     -> auto-block sender domain at the email gateway
     -> auto-open a P2 incident ticket
   - If inconclusive:
     -> route to analyst queue with enrichment data already attached

3. Human step: analyst reviews ambiguous cases only, with the enrichment
   already done -- what used to take 20 minutes of manual lookups now
   takes 2 minutes of judgment
```

## 3. Playbook logic in code form (conceptual)

```python
def phishing_playbook(email_report):
    iocs = extract_iocs(email_report)
    intel_hits = check_threat_intel(iocs)          # Level 3 Module 4
    sandbox_verdict = detonate_in_sandbox(iocs)     # Level 3 Module 3

    if sandbox_verdict.malicious and intel_hits:
        quarantine_email_org_wide(email_report.message_id)
        block_sender_domain(email_report.sender_domain)
        create_incident(severity="P2", auto_contained=True)
    else:
        queue_for_analyst(email_report, enrichment=intel_hits)
```

## 4. Where automation helps most — and where it doesn't

```
Good automation targets:
  - Enrichment (IP/domain/hash reputation lookups)
  - Repetitive containment (isolate a host, disable an account, block an IOC)
  - Evidence gathering (pull logs, screenshot, memory triage trigger)
  - Notification and ticketing

Poor automation targets (need human judgment):
  - Deciding whether an ambiguous behavior is malicious intent or a
    legitimate but unusual business process
  - Any action with major business impact if wrong (isolating a
    production database server based on one alert)
  - Novel attack patterns not covered by an existing playbook
```

Automating account disablement for a *confirmed* compromised account is
safe and valuable; automating account disablement purely on an unverified
alert risks a self-inflicted outage — the automation boundary should sit
exactly where confidence is high enough that a human would make the same
call anyway.

## 5. Case management integration

SOAR platforms typically also serve as the incident case management
system, tracking the full IR lifecycle from Level 1 Module 9 with
structured, auditable data instead of scattered emails/tickets:

```
Incident #4471
  Status: Contained
  Severity: P2
  Timeline: Reported 09:14 -> Enriched (auto) 09:14 -> Contained (auto) 09:16
            -> Analyst review 09:40 -> Root cause confirmed 10:15
  Actions taken: [auto] quarantine email, [auto] block domain,
                 [analyst] reset 3 user credentials
```

## 6. Building vs. buying, and starting small

Full commercial SOAR platforms (Splunk SOAR, Palo Alto Cortex XSOAR) are
one path; a smaller team can start with scripted automation glued
together via SIEM webhooks and cloud functions, and mature into a
platform as playbook complexity grows. Start with the single
highest-volume, most repetitive alert type — usually phishing or
malware detonation triage — rather than trying to automate everything
at once.

## 7. Metrics that justify automation investment

```
Before automation: avg 22 minutes analyst time per phishing report,
  40 reports/week = ~14.7 analyst-hours/week
After automation:  avg 3 minutes analyst time (ambiguous cases only,
  ~30% of reports) = ~2 analyst-hours/week
```

This is the business case that gets SOAR investment approved — expressed
in analyst-hours reclaimed for higher-value work like threat hunting
(Module 2), not just "faster response."

## 8. Checklist

- [ ] Highest-volume, most repetitive alert type automated first
- [ ] Automation boundary set where human judgment adds no real value
- [ ] Every automated action is logged and auditable after the fact
- [ ] High-impact actions (isolating production systems) require human sign-off
- [ ] Playbooks version-controlled and reviewed like code
- [ ] Time-saved metrics tracked to justify and prioritize further automation

## What's next

Module 6 extends automation and DevSecOps practices specifically into
cloud-native environments.
