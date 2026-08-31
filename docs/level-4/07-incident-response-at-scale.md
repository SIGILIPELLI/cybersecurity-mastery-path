# 07 · Incident Response at Scale

Level 1 Module 9 covered IR fundamentals for a single incident.
Enterprise-scale IR must handle multiple simultaneous incidents,
cross-team coordination, legal/regulatory obligations, and incidents
spanning thousands of hosts — a fundamentally different operational
challenge.

## 1. Incident command structure

Borrowed from emergency management (Incident Command System), a large
incident needs clear roles so decisions don't bottleneck on one person
or get made by everyone at once:

```
Incident Commander -- owns the overall response, makes final calls,
                       coordinates across teams
Technical Lead      -- directs the hands-on-keyboard investigation/containment
Communications Lead -- manages internal updates, customer comms, PR/legal liaison
Scribe              -- maintains the authoritative incident timeline in real time
Subject matter experts -- pulled in per-system as needed (network, cloud, app owners)
```

Without this structure, a major incident tends to devolve into dozens of
people in one call, duplicating effort and losing the timeline record
needed for both the response and the eventual forensic report
(Level 3 Module 2).

## 2. Severity classification and escalation

```
SEV1 -- active, confirmed breach with material business/customer impact;
        Incident Commander + exec bridge activated immediately
SEV2 -- confirmed compromise, contained or limited scope; standard IR team
SEV3 -- suspicious activity under investigation, not yet confirmed
SEV4 -- policy violation or low-risk anomaly, handled via normal ticketing
```

Pre-defined severity criteria and escalation paths mean the first
responder doesn't have to improvise who to call at 2 AM — that decision
was made calmly, in advance.

## 3. Coordinating containment across many hosts

At scale, "isolate the affected host" (Level 1 Module 9) becomes
"isolate 200 potentially affected hosts without taking down the
business":

```bash
# EDR-driven mass containment via API, not manual per-host action
curl -X POST https://edr.internal/api/v1/hosts/isolate \
  -d '{"host_ids": ["h1","h2","...","h200"], "reason": "incident-4471"}'
```

```
Containment strategy decision tree:
  - Can we isolate at the network layer (VLAN/segment) instead of
    per-host, to move faster with fewer errors?
  - Is the business-critical path affected? If so, coordinate the
    containment window with business stakeholders, don't act unilaterally
    on revenue-critical systems without that conversation
```

## 4. Legal, regulatory, and communications coordination

A large incident triggers obligations Level 1's basic IR process doesn't
need to cover:

```
- Breach notification laws (varies by jurisdiction and data type --
  e.g. GDPR's 72-hour regulator notification window)
- Cyber insurance carrier notification (often required within a specific
  window to preserve coverage)
- Law enforcement engagement, when appropriate
- Customer/public communications, reviewed by legal before release
- Preserving evidence in a way that survives later legal proceedings
  (chain of custody, Level 3 Module 2)
```

Legal counsel should be looped in early, not after technical work is
done — some jurisdictions extend attorney-client privilege to incident
investigation materials only when structured correctly from the start.

## 5. Managing multiple concurrent incidents

```
Incident queue triage (similar to a hospital ER):
  - Multiple SEV2s active: assign separate technical leads, shared
    Incident Commander only if resources genuinely overlap
  - Resource contention: a scarce specialist (e.g. the one person who
    understands the legacy mainframe) becomes the bottleneck --
    identify and plan around single points of failure in the response
    team itself, not just in the infrastructure
```

## 6. Retrospectives and post-incident review

A blameless post-incident review, done for every significant incident,
turns each one into organizational learning rather than a closed ticket:

```
Post-Incident Review structure:
  1. Factual timeline (from the scribe's real-time log)
  2. What went well
  3. What went poorly / took too long
  4. Root cause (technical AND process — e.g. "alert existed but no
     one was on call to see it")
  5. Action items, each with an owner and a deadline
  6. Follow-up: were the action items actually completed?
```

Skipping the follow-up step is the most common failure mode — action
items from a post-incident review that never get tracked to completion
guarantee the same gap causes the next incident.

## 7. Tabletop exercises for scale readiness

Before a real large-scale incident, rehearse the command structure
itself with a tabletop exercise — a scenario walked through verbally by
the actual response roles, without touching real systems:

```
Scenario: "Ransomware has encrypted file shares across three regional
offices simultaneously. Backups in two regions are also affected."
Walk through: who declares SEV1? Who's the Incident Commander? What's
the first containment decision? Who talks to customers, and what do
they say before root cause is even known?
```

## 8. Checklist

- [ ] Incident command roles defined and staffed before an incident, not during
- [ ] Severity classification criteria and escalation paths pre-defined
- [ ] Mass-containment capability (EDR API, network-layer isolation) in place
- [ ] Legal/regulatory notification obligations known in advance per jurisdiction
- [ ] Blameless post-incident reviews conducted for every significant incident
- [ ] Post-incident action items tracked to actual completion
- [ ] Tabletop exercises rehearse the command structure, not just technical steps

## What's next

Module 8 looks at building the SOC organization that runs this incident
response capability day to day.
