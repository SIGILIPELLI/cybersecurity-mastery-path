# 02 · Advanced Threat Hunting

SIEM alerts (Level 3 Module 5) catch what you already thought to detect.
Threat hunting is the proactive discipline of looking for adversaries
who evaded every existing alert — starting from a hypothesis, not a
triggered rule.

## 1. Hunting vs. alerting

```
Alerting:  "Tell me when X happens" -- reactive, scales, but only catches
            known patterns.
Hunting:    "Assume X already happened undetected -- go find evidence" --
            proactive, human-led, catches what alerting missed.
```

A mature program needs both: hunting findings that prove valuable become
new alerts (feeding back into Level 3 Module 5), continuously raising the
bar for what's caught automatically.

## 2. The hunting process

```
1. Hypothesis   -- "If an attacker has a foothold, they'd likely use
                    living-off-the-land binaries for discovery"
2. Data gathering -- pull relevant logs (process creation, network, auth)
3. Analysis      -- look for the hypothesized behavior or its absence
4. Findings      -- confirmed malicious activity, a new detection gap,
                    or nothing found (still valuable -- documents coverage)
5. Feedback      -- new Sigma/SIEM rule, updated baseline, or closed hunt
```

## 3. Hypothesis-driven hunting with ATT&CK

```
Hypothesis: "Adversaries may be using PsExec or WMI for lateral movement,
             bypassing our RDP-focused monitoring" (T1021.002, T1047)

Hunt query (Splunk):
index=wineventlog EventCode=4688
| search CommandLine="*wmic*" OR CommandLine="*psexec*"
| stats count by host, user, CommandLine
| where count > 0
```

Every hunt should map to specific ATT&CK techniques not yet well-covered
by existing detections (from the coverage tracking in Level 3 Module 5),
prioritizing gaps over techniques already well-alerted.

## 4. Data-driven / baseline hunting

Instead of a specific hypothesis, look for statistical outliers against
an established baseline of "normal" for your environment:

```spl
# Find processes that ran on exactly one host in the last 90 days --
# rare execution is a strong (not certain) signal of something unusual
index=wineventlog EventCode=4688 earliest=-90d
| stats dc(host) as host_count by Image
| where host_count=1
| sort host_count
```

```spl
# Beaconing detection: connections at suspiciously regular intervals
# (a hallmark of malware C2 checking in on a timer)
index=network
| streamstats current=f last(_time) as prev_time by src_ip, dest_ip
| eval delta=_time-prev_time
| stats stdev(delta) as jitter, avg(delta) as avg_interval by src_ip, dest_ip
| where jitter < 2 AND avg_interval > 30
```

## 5. Hunting across the kill chain

```
Reconnaissance:  unusual DNS queries for internal hostnames from a single host
Delivery:        rare/newly-registered sender domains in email logs
Execution:       parent-child process anomalies (Word spawning PowerShell)
C2:              long-lived connections, beaconing intervals, DNS tunneling
                 patterns (high query volume, unusual TXT record sizes)
Lateral movement: authentication to hosts a user has never accessed before
Exfiltration:    large outbound transfers to rare destinations, off-hours
```

## 6. Tooling for advanced hunting

```bash
# EDR query languages (example: CrowdStrike/Elastic-style) let hunters
# search raw telemetry beyond what's pre-indexed into alerts
process where process.name == "rundll32.exe" and
  process.command_line : "*javascript:*"
```

Jupyter notebooks with direct data-source access (rather than only a
SIEM UI) let hunters apply statistical/ML techniques — clustering,
anomaly scoring — that a standard SIEM search language can't express.

## 7. Documenting and operationalizing hunts

A hunt that isn't documented is a hunt that gets re-run from scratch
next quarter with no institutional memory:

```
Hunt ID: H-2026-014
Hypothesis: Living-off-the-land lateral movement via WMI
Data sources: Windows Event Logs (Sysmon), 90-day window
Result: No malicious activity found; identified 2 legitimate admin
  scripts using WMI that should be added to a baseline allowlist
Outcome: New Sigma rule created for anomalous WMI use outside allowlist;
  ATT&CK T1047 coverage moved from "none" to "detected"
```

## 8. Checklist

- [ ] Hunts are hypothesis-driven and mapped to specific ATT&CK techniques
- [ ] Both hypothesis-driven and baseline/statistical hunting used
- [ ] A "no findings" hunt still documented and counted as coverage gained
- [ ] Confirmed findings converted into permanent detections
- [ ] Hunt results logged for institutional memory, not re-derived each time

## What's next

Module 3 looks at zero trust architecture, a design philosophy that
shrinks the space an attacker can move through undetected in the first place.
