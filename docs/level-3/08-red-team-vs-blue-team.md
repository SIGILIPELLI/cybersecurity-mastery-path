# 08 · Red Team vs Blue Team Concepts

Every skill built so far — offense (pentesting) and defense (SIEM,
forensics, IR) — comes together in structured exercises that test
detection and response against realistic, controlled adversary behavior.

!!! warning "Authorized exercises only"
    Red/purple team exercises are run against your own infrastructure or
    lab, with written rules of engagement, defined scope, and management
    sign-off. Never simulate an attack against systems you don't have
    explicit authorization to test.

## 1. Red, blue, and purple teams

- **Red team** — simulates a real adversary: reconnaissance, initial
  access, persistence, lateral movement, objective achievement — testing
  whether defenses *detect and stop* realistic attack behavior, not just
  whether a vulnerability exists.
- **Blue team** — the defenders: SOC analysts, IR, detection engineers,
  actively monitoring and responding during the exercise.
- **Purple team** — a collaborative model where red and blue work
  together in real time, sharing what worked and what didn't as the
  exercise happens, maximizing learning over "gotcha" competition.

## 2. Rules of engagement (RoE)

Before any exercise starts, a written RoE must define:

```
Scope: which systems/networks are in-scope, which are explicitly excluded
Timing: exercise window, blackout periods (e.g. no testing during month-end close)
Techniques allowed/disallowed: e.g. no destructive actions, no real phishing
  of executives without separate sign-off, no testing production payment systems
Emergency stop: a "safe word" / kill switch and an escalation contact
  reachable at all times during the exercise
Rules for handling real (non-simulated) incidents discovered mid-exercise
```

## 3. Structuring a red team exercise around ATT&CK

Instead of "just hack it," mature red team exercises are scenario-based:
emulate a specific, realistic threat actor's known TTPs (from Module 4's
threat intelligence), giving the blue team a meaningful measurement of
readiness against threats relevant to the organization:

```
Scenario: emulate a ransomware affiliate's known chain
  1. Initial access:  T1566.001 - Phishing attachment (simulated, benign payload)
  2. Execution:       T1059.001 - PowerShell
  3. Persistence:      T1053.005 - Scheduled Task
  4. Discovery:        T1082 - System Information Discovery
  5. Lateral movement:  T1021.002 - SMB/Windows Admin Shares
  6. Impact (simulated only, never real): T1486 - Data Encrypted for Impact
```

## 4. Blue team detection and response during the exercise

The blue team's job is to detect each stage using the tools built in
prior modules, and to practice the full IR lifecycle without knowing
in advance which steps will fire:

```spl
# Example detection the blue team should already have from Module 5:
# T1059.001 - encoded PowerShell command
index=wineventlog EventCode=4688 
| search CommandLine="*-EncodedCommand*" OR CommandLine="*-enc*"
```

After each stage, the blue team logs: was it detected, how long did
detection take (mean time to detect), and was the response appropriate
and timely (mean time to respond)?

## 5. Purple team debrief structure

The most valuable output of any exercise is the after-action debrief,
mapped stage by stage:

| ATT&CK Technique | Detected? | Time to Detect | Gap / Fix |
|---|---|---|---|
| T1566.001 Phishing | Yes | 4 min (email gateway) | none |
| T1059.001 PowerShell | No | — | Add Sigma rule for EncodedCommand |
| T1053.005 Scheduled Task | Yes | 22 min (manual review) | Automate detection, currently manual only |
| T1021.002 Lateral movement | No | — | No SMB logon monitoring — add rule |

Each undetected step becomes a tracked action item — a new detection
rule, a control gap to close, or a process fix — feeding directly back
into Module 5's SIEM work and Module 4's threat intelligence coverage
tracking.

## 6. Adversary emulation tooling

```bash
# MITRE Caldera or Atomic Red Team execute known ATT&CK techniques
# in a controlled, reversible way for exercise purposes
Invoke-AtomicTest T1059.001 -TestNumbers 1
```

These frameworks intentionally simulate technique *signatures* without
causing real damage, letting the same test be re-run safely to validate
a fix.

## 7. Measuring program maturity over time

Track metrics across successive exercises, not just one point in time:

```
Exercise 1 (Q1): 3/6 techniques detected, avg detect time 45 min
Exercise 2 (Q3): 5/6 techniques detected, avg detect time 12 min
```

Improving detection coverage and shrinking detection/response time
across repeated exercises is the actual measure of a maturing security
program — a single exercise is a snapshot, the trend is the signal.

## How It Actually Works: how adversary emulation tooling actually maps to detections, and why maturity is measured as coverage over a matrix

Adversary emulation platforms (Atomic Red Team, Caldera) work by
implementing each MITRE ATT&CK technique as a small, self-contained,
reversible **test procedure** that performs the *exact minimal OS-level
action* the technique describes — for T1003 (credential dumping), the test
might invoke a specific documented Windows API sequence for reading process
memory, not a real attacker's full toolchain, and immediately cleans up
after itself. Running these tests one technique at a time, with blue team
telemetry capture on, produces a **direct empirical answer** to "does our
detection stack actually alert on this," rather than a theoretical one —
because the test executes the identical syscalls/API calls a real technique
would (the same sequence-based detection logic engineered in Level 3's SIEM
module), the resulting alert (or silence) is a faithful signal, not a
simulation of one.

**Purple teaming** structurally accelerates this over a standard red-team
engagement because it collapses the feedback loop: in a traditional
sequential red/report/remediate cycle, the blue team only learns what was
missed weeks later, after the engagement report — by which time the red
team has moved to the next objective and the specific telemetry gap is
harder to reconstruct. Running detection engineering *during* the exercise,
technique by technique, means a missed detection can be diagnosed
immediately against the still-fresh raw logs (was the telemetry source
missing entirely, or present but not correlated) and often fixed and
re-tested against the *same* technique within the same engagement — turning
one exercise into many rapid detect/tune iterations instead of one.

**Program maturity** measurement is mechanically a **coverage matrix**: an
organization maps which of ATT&CK's ~200 techniques it has (a) genuine log
telemetry for, (b) an active correlation rule for, and (c) empirically
validated via emulation testing — three progressively stronger claims, since
having a log source doesn't guarantee a rule consumes it, and having a rule
doesn't guarantee it actually fires on the real technique until tested. This
is why "maturity" is reported as a percentage of the matrix filled at each
tier rather than a single score: it's a literal count of squares in a grid
that either have or lack demonstrated, tested coverage, which is exactly
what lets a security program track measurable progress release over release
rather than relying on subjective confidence.

## 8. Checklist

- [ ] Written rules of engagement approved before any exercise begins
- [ ] Scenario built around realistic, relevant ATT&CK techniques
- [ ] Blue team detection/response timed and logged per stage
- [ ] Purple team debrief maps every stage to detected/undetected + fix
- [ ] Gaps become tracked action items, not just discussion points
- [ ] Metrics tracked across exercises to show trend, not just one snapshot

## What's next

Module 9 shifts from technical exercises to the compliance frameworks
that require and formalize this kind of testing.
