# 04 · Threat Intelligence

Threat intelligence turns raw indicators (from malware analysis,
incident response, and external feeds) into decisions: what to block,
what to hunt for, and which threats actually matter to *your*
organization instead of every threat that exists in theory.

## 1. The intelligence cycle

```
Direction -> Collection -> Processing -> Analysis -> Dissemination -> Feedback
```

- **Direction** — what questions does the business need answered? ("Are
  we being targeted by ransomware groups active in our sector?")
- **Collection** — gather raw data from feeds, logs, OSINT, vendor reports.
- **Processing** — normalize, deduplicate, enrich.
- **Analysis** — turn data into an assessment with confidence levels.
- **Dissemination** — get it to the people who need it, in the format
  they can act on (a SOC analyst needs an IOC list; a CISO needs a
  one-page brief).
- **Feedback** — did it help? Adjust direction accordingly.

## 2. Strategic, operational, and tactical intelligence

| Level | Audience | Example |
|---|---|---|
| Strategic | Executives, board | "Ransomware targeting our industry rose 40% this quarter" |
| Operational | SOC managers, IR leads | "Group X is using phishing lures referencing invoice payments" |
| Tactical | SOC analysts, engineers | IOCs: specific hashes, IPs, domains, YARA/Sigma rules |

## 3. The Pyramid of Pain

Not all indicators are equally valuable to defenders — some cost the
attacker nothing to change, others force a complete retooling:

```
       TTPs (hardest for attacker to change) -- highest value
      Tools
     Network/Host Artifacts
    Domain Names
   IP Addresses
  Hash Values (easiest for attacker to change) -- lowest value
```

Blocking a single file hash inconveniences an attacker for seconds
(rebuild, re-hash). Detecting the *technique* — e.g. "PowerShell spawned
from Office with an encoded command" — forces them to change their whole
approach. This is why mature programs invest in TTP-level detection
(Module 5, Module 8) rather than hash-blocklisting alone.

## 4. Structured formats: STIX/TAXII and MITRE ATT&CK

```json
{
  "type": "indicator",
  "spec_version": "2.1",
  "id": "indicator--example-uuid",
  "pattern": "[domain-name:value = 'evil-c2-domain-from-lab.example']",
  "pattern_type": "stix",
  "valid_from": "2026-08-31T00:00:00Z",
  "labels": ["malicious-activity"]
}
```

STIX is the standard data format for sharing intel; TAXII is the
transport protocol for exchanging it between organizations and feeds.
MITRE ATT&CK maps observed behavior to standardized tactic/technique IDs
(e.g. `T1059.001` — PowerShell) so intelligence from different sources
speaks the same language.

```bash
# Map a finding to ATT&CK for consistent reporting
# T1566.001 - Phishing: Spearphishing Attachment
# T1059.001 - Command and Scripting Interpreter: PowerShell
# T1071.001 - Application Layer Protocol: Web Protocols (C2 over HTTPS)
```

## 5. Sourcing intelligence

- **Open source (OSINT)**: vendor blogs, CISA advisories, AlienVault
  OTX, abuse.ch (URLhaus, MalwareBazaar, ThreatFox).
- **Commercial feeds**: paid threat intel platforms with curated,
  higher-confidence indicators and analyst context.
- **ISACs/ISAOs**: sector-specific sharing communities (Financial
  Services ISAC, Healthcare ISAC, etc.) — often the highest-relevance
  source for sector-targeted threats.
- **Internal**: your own IR case history and honeypot/deception data are
  intelligence too, and the most directly relevant to your environment.

```bash
# Example: pull a public IOC feed for enrichment
curl -s https://feodotracker.abuse.ch/downloads/ipblocklist.json \
  | jq '.[] | {ip_address, malware, first_seen}'
```

## 6. Operationalizing intelligence

Raw indicators only matter once they change detection or blocking:

```bash
# Feed IOCs into a SIEM watchlist / lookup table (example: Splunk)
| inputlookup threat_ioc_list.csv
| where src_ip=ioc_ip OR dest_ip=ioc_ip

# Or push into a firewall/proxy blocklist automatically via API
curl -X POST https://firewall.internal/api/blocklist \
  -d '{"indicator": "evil-c2-domain-from-lab.example", "action": "block"}'
```

A threat intelligence platform (TIP) such as MISP centralizes this:
ingesting feeds, deduplicating, scoring confidence, and pushing
actionable indicators out to SIEM, EDR, and firewalls automatically.

## 7. Assessing confidence and relevance

Every piece of intelligence should carry a confidence level (source
reliability × corroboration) and a relevance judgment for your specific
environment. A well-sourced report about a threat actor targeting a
different industry, region, and technology stack than yours is
interesting reading, not an action item — chasing every published IOC
regardless of relevance burns analyst time without reducing real risk.

## 8. Checklist

- [ ] Intelligence requirements defined by business stakeholders, not just IT
- [ ] Indicators mapped to ATT&CK techniques, not stored as raw hashes only
- [ ] Feeds deduplicated and scored for confidence before action
- [ ] High-value (TTP-level) detections prioritized over hash-only blocking
- [ ] Intelligence disseminated in a format each audience can act on
- [ ] Feedback loop exists to retire stale or irrelevant indicators

## What's next

Module 5 puts this intelligence to work inside a SIEM at scale.
