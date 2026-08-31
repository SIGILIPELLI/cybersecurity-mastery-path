# 05 · Advanced SIEM (Splunk/ELK)

Level 2 Module 6 covered SIEM fundamentals — collecting and searching
logs. This module goes further: building real detections, correlation
across data sources, and tuning a SIEM so analysts trust its alerts
instead of drowning in them.

## 1. Data onboarding and normalization

A SIEM is only as good as what feeds it and how consistently. The
Common Information Model (Splunk CIM) or Elastic Common Schema (ECS)
normalizes fields across sources so a single search works whether the
log came from a Windows host, a firewall, or a cloud API:

```
# Without normalization, "source IP" might be:
src_ip, srcip, source_address, client_ip ...
# With ECS/CIM, it's always:
source.ip
```

```yaml
# Elastic: a Filebeat module maps vendor-specific fields to ECS automatically
filebeat.modules:
  - module: cisco
    asa:
      enabled: true
```

## 2. Building correlation searches

The real power of a SIEM is correlating *across* sources — a single
failed login means nothing; the same failed login followed by success
from a different country ten seconds later is a strong signal.

```spl
# Splunk SPL: impossible travel detection
index=auth sourcetype=okta action=login
| iplocation src_ip
| stats earliest(_time) as t1 latest(_time) as t2 values(Country) as countries by user
| where mvcount(countries) > 1 AND (t2-t1) < 3600
```

```json
// Elastic detection rule (simplified): brute force followed by success
{
  "name": "Brute Force then Successful Login",
  "query": "event.action:\"logon-failed\" or event.action:\"logon-success\"",
  "threshold": { "field": "user.name", "value": 5 }
}
```

## 3. MITRE ATT&CK-aligned detection engineering

Instead of writing ad-hoc rules, mature SIEM programs map every
detection to an ATT&CK technique, then track *coverage* — which
techniques an adversary could use undetected in your environment:

```spl
# T1110.001 - Brute Force: Password Guessing
index=auth action=failure
| stats count by user, src_ip
| where count > 10
```

Sigma is a vendor-agnostic rule format for exactly this — write once,
convert to Splunk SPL, Elastic Query DSL, or other backends:

```yaml
title: Suspicious PowerShell Encoded Command
status: stable
logsource:
  product: windows
  category: process_creation
detection:
  selection:
    Image|endswith: '\powershell.exe'
    CommandLine|contains: '-EncodedCommand'
  condition: selection
level: high
tags:
  - attack.execution
  - attack.t1059.001
```

## 4. Tuning to reduce alert fatigue

An unfiltered SIEM produces so many alerts that analysts start ignoring
them — the single biggest cause of real incidents being missed. Tuning
techniques:

- **Suppression** — silence known-benign repeat triggers (a vulnerability
  scanner's own traffic, a documented admin script).
- **Risk-based alerting** — score events instead of firing a discrete
  alert per rule; only alert when accumulated risk for an entity crosses
  a threshold.
- **Baselining** — flag deviation from an entity's *own* normal behavior
  (User and Entity Behavior Analytics, UEBA) instead of a single static
  threshold for everyone.

```spl
# Risk-based: add points instead of alerting per event
index=* tag=risk
| eval risk_score=case(
    signature="brute_force", 20,
    signature="impossible_travel", 40,
    signature="known_malware_hash", 100,
    true(), 5)
| stats sum(risk_score) as total_risk by user
| where total_risk > 80
```

## 5. Dashboards for different audiences

- **SOC analyst dashboard** — live queue of prioritized, unresolved alerts.
- **Threat hunting dashboard** — raw searchable data, not pre-filtered.
- **Executive dashboard** — trend lines: alert volume, mean time to
  detect/respond, top attacked assets — no raw log data.

## 6. Search performance at scale

```spl
# Bad: scans everything, every time
index=* "error"

# Good: narrow index/sourcetype and time range first, filter fields early
index=web_proxy sourcetype=squid earliest=-1h
| search status=403
```

In Elastic, use index lifecycle management (ILM) to age data from hot
(fast, expensive) to warm/cold/frozen tiers as it ages, keeping recent
detection searches fast while retaining older data for investigations.

## 7. Checklist

- [ ] Logs normalized to a common schema (CIM/ECS) at ingest
- [ ] Detections mapped to ATT&CK techniques with tracked coverage
- [ ] Alert volume actively tuned (suppression, risk scoring, baselining)
- [ ] Detection rules version-controlled (Sigma or equivalent) not ad-hoc
- [ ] Dashboards tailored per audience (analyst vs. exec)
- [ ] Data tiering configured so hot-path searches stay fast at scale

## What's next

Module 6 applies these same log-and-detection principles specifically
to cloud environments.
