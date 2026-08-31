# 08 · Building a SOC

Every capability built so far — SIEM (Level 3 Module 5), threat hunting
(Module 2), IR at scale (Module 7), automation (Module 5) — needs an
organizational structure to run continuously. This module covers
designing that Security Operations Center.

## 1. SOC operating models

```
In-house SOC       -- full control, deep organizational context, highest cost
MSSP (outsourced)  -- lower cost, faster to stand up, less organizational context
Hybrid/co-managed  -- MSSP provides 24/7 tier-1 monitoring, in-house team
                      handles tier-2/3, hunting, and incident ownership
```

Most mid-size organizations land on hybrid: 24/7 coverage is expensive
to staff in-house (a true follow-the-sun model needs 3+ shifts,
multiplied by redundancy), but incident ownership and deep context
should stay in-house.

## 2. Tiered analyst structure

```
Tier 1 (Triage)     -- initial alert review, follows runbooks, escalates
                        anything ambiguous or confirmed malicious
Tier 2 (Investigation) -- deeper analysis, correlates across sources,
                        handles escalations, executes IR playbook steps
Tier 3 (Expert/Hunt) -- threat hunting (Module 2), builds new detections,
                        handles the most complex/novel incidents
SOC Manager          -- staffing, metrics, process improvement, stakeholder reporting
```

A healthy SOC has a career path through these tiers, not a permanent
tier-1 population — tier 1 burnout from purely repetitive alert
triage is one of the leading causes of SOC analyst attrition industry-wide.

## 3. Coverage model and staffing math

```
24/7/365 coverage with 8-hour shifts needs:
  3 shifts/day x 7 days x minimum 1 analyst per shift = at least 3 FTEs
  minimum, but realistically 5-6 to cover PTO, sick leave, and avoid
  single-analyst-on-shift risk (no backup if that person is unavailable
  mid-incident)
```

Follow-the-sun models (handing off between regional SOCs at shift
change) avoid pure night-shift staffing but require rigorous handoff
documentation so context isn't lost between teams.

## 4. Runbooks and playbooks

The difference between a SOC that scales and one that doesn't is
whether tier-1 analysts have clear, tested runbooks rather than relying
on institutional memory that leaves when someone does:

```
Runbook: Suspected Phishing Alert
1. Verify sender domain against known-good list and threat intel (Lvl 3 Mod 4)
2. Check if email was opened/link clicked (mail gateway logs)
3. If clicked: initiate SOAR playbook (Module 5) — auto-enrichment,
   auto-containment if confirmed malicious
4. If uncertain after step 1-2: escalate to Tier 2 with findings attached
5. Document outcome in ticket regardless of verdict
```

## 5. Metrics that matter

```
MTTD (Mean Time to Detect)  -- how long from compromise to first detection
MTTR (Mean Time to Respond) -- how long from detection to containment
Alert-to-analyst ratio      -- alerts per analyst per shift (too high =
                                fatigue and missed detections)
False positive rate         -- per detection rule, tracked over time to
                                justify tuning (Level 3 Module 5)
Coverage (ATT&CK)           -- techniques with a working detection,
                                tracked from Level 3-4 hunting/detection work
```

Reporting MTTD/MTTR trend lines to leadership, alongside coverage
growth, makes the SOC's value visible in terms executives already
understand — this is the same principle as Level 3 Module 9's compliance
framing: translate technical work into business language.

## 6. Tooling stack for a SOC

```
SIEM               -- central detection and search (Level 3 Module 5)
EDR                -- endpoint telemetry and containment actions
SOAR               -- playbook automation (Module 5)
Threat intel platform -- IOC/TTP enrichment (Level 3 Module 4)
Case management    -- incident tracking (often integrated into SOAR)
Communication      -- a dedicated, tested incident bridge/channel separate
                      from day-to-day chat, so it's available even if
                      normal tools are affected by the incident itself
```

## 7. Analyst wellbeing and retention

SOC work is high-vigilance, often repetitive, and 24/7 — burnout risk is
structural, not incidental. Practical mitigations: rotate analysts
between triage and hunting/project work, cap consecutive night shifts,
invest in tooling that reduces noise (Level 3 Module 5's tuning work
directly reduces analyst fatigue), and build a real career ladder so
tier-1 is a stepping stone, not a dead end.

## 8. Checklist

- [ ] Operating model (in-house/MSSP/hybrid) matches actual coverage needs and budget
- [ ] Tiered structure with a real career progression path exists
- [ ] Staffing accounts for PTO/sick leave, not bare minimum shift coverage
- [ ] Runbooks exist and are kept current for the highest-volume alert types
- [ ] MTTD/MTTR and ATT&CK coverage tracked and reported to leadership
- [ ] Analyst wellbeing actively managed, not left to attrition to reveal problems

## What's next

Module 9 moves up a level further — the leadership and risk management
responsibilities that own the SOC's budget, mandate, and business alignment.
