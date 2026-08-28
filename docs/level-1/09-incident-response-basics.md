# 09 · Incident Response Basics

Every control in this course can fail. Incident response (IR) is the
discipline of what happens *after* it does — detecting a problem, containing
the damage, removing the cause, recovering normal operation, and learning
from it. A team with weak preventive controls but a strong IR process
recovers from breaches; a team with strong preventive controls and no IR
plan is unprepared for the one that gets through anyway.

## 1. The incident response lifecycle

The widely-used NIST framework breaks IR into phases. This module covers the
first four in detail; Level 4 Module 7 (Incident Response at Scale) covers
running this process across a large organization.

| Phase | Goal | Typical activities |
|---|---|---|
| **1. Preparation** | Be ready before anything happens | IR plan, contact lists, tooling, backups, tabletop exercises |
| **2. Detection & Analysis** | Notice something happened and understand it | Alert triage, log review, confirming a true positive |
| **3. Containment** | Stop it from getting worse | Isolate affected systems, disable compromised accounts, block malicious IPs |
| **4. Eradication** | Remove the actual cause | Delete malware, close the vulnerability that was exploited, remove attacker footholds |
| **5. Recovery** | Return to normal operation | Restore from clean backups, rebuild systems, verify integrity before reconnecting |
| **6. Lessons Learned** | Improve for next time | Post-incident report, root cause, process/control changes |

```
Preparation --> Detection & Analysis --> Containment --> Eradication --> Recovery --> Lessons Learned
      ^                                                                                      │
      └──────────────────────────────────────────────────────────────────────────────────────┘
                              (feeds back into improved preparation)
```

!!! note "This is a cycle, not a line"
    The lessons-learned phase feeds directly back into preparation — a
    patched vulnerability, an updated playbook, a new detection rule. An
    organization that treats each incident as an isolated event, rather than
    input to the next cycle, will keep having the same incident.

## 2. Detection — the phase most organizations underinvest in

You cannot respond to what you don't know happened. Detection sources
include:

| Source | What it catches |
|---|---|
| **Antivirus/EDR alerts** | Known malware signatures, suspicious process behavior |
| **Firewall/IDS logs** | Unusual network traffic, known attack signatures |
| **Authentication logs** | Failed login spikes, logins from unusual locations/times |
| **File integrity monitoring** | Unexpected changes to critical system files |
| **User reports** | "This email looked suspicious," "my computer is acting strange" |
| **SIEM correlation** | Combining signals from multiple sources into one alert (Level 2 Module 6) |

A critical, sobering statistic pattern across real breach reports: the median
time between initial compromise and detection has historically been measured
in **weeks to months**, not minutes — attackers who gain a foothold often
spend a long time moving quietly (Level 3 Module 1's "lateral movement")
before doing anything that trips an alert. This is precisely why Module 9's
logging and Level 2/3's SIEM modules matter as much as prevention: a fast
detection cuts that dwell time dramatically, which directly limits the
damage an attacker can do.

## 3. Containment strategies

Once something is confirmed, the immediate priority shifts from
understanding everything to **stopping the bleeding** — full analysis can
continue afterward on a preserved, isolated copy.

| Strategy | When to use | Trade-off |
|---|---|---|
| **Isolate the host** (disconnect network, not power) | Active malware/attacker on one machine | Preserves evidence in memory; stops the machine communicating outward |
| **Disable the account** | Compromised credentials | Immediate; may alert the attacker that they've been noticed |
| **Block the IP/domain at the firewall** | Known malicious C2 (command-and-control) infrastructure | Fast, but attacker can rotate infrastructure |
| **Segment the network further** | Spreading incident (e.g., worm behavior) | Buys time; disruptive to legitimate users too |

!!! warning "Don't power off a compromised machine"
    Powering off destroys volatile evidence in RAM — running processes,
    network connections, encryption keys that might only exist in memory —
    that forensic analysis (Level 3 Module 2) may need. **Disconnect the
    network cable/Wi-Fi instead**, leaving the machine running and isolated,
    unless there's an active, ongoing destructive action (e.g., ransomware
    actively encrypting files) that outweighs evidence preservation.

## 4. Chain of custody and evidence handling

If an incident might lead to legal action, law enforcement involvement, or
regulatory reporting, evidence handling has to be defensible. **Chain of
custody** is the documented, unbroken record of who collected evidence, when,
how, and who has had access to it since — the same standard used in physical
crime scenes, applied to digital evidence.

A minimal chain-of-custody log entry includes: what was collected (e.g., "disk
image of workstation WK-042"), who collected it, the exact timestamp, the
method used, and a cryptographic hash (Module 4) of the collected data taken
immediately — so that any later dispute about whether the evidence was
altered can be settled by re-hashing and comparing.

## 5. Logs — the raw material of detection and investigation

Nearly every phase above depends on **logs actually existing and being kept
long enough to matter**. A baseline logging checklist:

- [ ] Authentication logs (successful and failed logins) are retained, not
      just generated
- [ ] Logs are sent to a separate system, not just stored locally on the
      machine that might get compromised (an attacker who controls a host can
      delete its local logs)
- [ ] Clocks are synchronized (NTP) across systems — correlating events
      across machines with different clocks is unreliable otherwise
- [ ] Retention period is long enough to cover realistic detection delays
      (recall section 2's dwell-time statistic — 30 days of logs doesn't help
      if detection takes 90)

```bash
# Linux -- a quick look at real authentication log activity
sudo grep "Failed password" /var/log/auth.log | tail -20      # Debian/Ubuntu
sudo journalctl -u sshd --since "1 hour ago"                   # systemd-based
```

Level 2 Module 6 (SIEM & Log Analysis Basics) builds directly on this —
correlating exactly this kind of log data across many systems is what a SIEM
does at scale.

## 6. A minimal incident response plan

Every organization — even a team of one — benefits from writing this down
*before* an incident, not during one, when adrenaline and time pressure make
good decisions harder:

1. **Contacts** — who do you call (internal IT/security lead, external
   incident response firm if applicable, legal counsel, law enforcement if
   warranted)?
2. **Severity classification** — what makes something a minor issue vs. a
   major incident requiring the full process?
3. **Immediate containment steps** for the most likely scenarios (a phished
   account, a ransomware alert, a lost laptop).
4. **Communication plan** — who needs to be told, and when (including any
   legal/regulatory breach-notification requirements for the data involved).
5. **Backup restoration procedure**, tested in advance — a backup you've
   never restored from is a hypothesis, not a plan.

## Key terms

| Term | Meaning |
|---|---|
| **Incident** | A confirmed or suspected security event requiring response |
| **Dwell time** | The time between initial compromise and detection |
| **Containment** | Actions to stop an incident from spreading or worsening |
| **Eradication** | Removing the root cause of the incident |
| **Chain of custody** | Documented record of who handled evidence, when, and how |
| **C2 (command-and-control)** | Infrastructure an attacker uses to control compromised systems remotely |
| **IOC (Indicator of Compromise)** | A specific artifact (file hash, IP, domain) suggesting a system is compromised |

## Exercise

No special tools required for most of this — a text editor and, optionally,
your own machine's local auth logs.

1. **Check your own authentication logs.** On macOS, run
   `log show --predicate 'eventMessage contains "authentication"' --last 1h`;
   on Linux, use the commands from section 5. Note whether you can identify
   any failed login attempts and, if so, whether they look like your own
   mistyped password or something else.

2. **Write a minimal incident response plan** (section 6) for your own
   personal digital life — treat "your accounts and devices" as the
   organization. Name at least one specific contact/resource for each of the
   five sections (e.g., your bank's fraud line for a compromised financial
   account).

3. **Walk through a scenario.** You receive an alert that your personal email
   account had a successful login from a country you've never visited. Write
   out, phase by phase (Preparation through Lessons Learned), exactly what
   you would do — be specific (e.g., "Containment: change the password
   immediately, then check and revoke active sessions/connected apps in
   account settings").

4. **Written answer.** Explain, in your own words, why powering off a
   suspected-compromised machine immediately is usually the wrong first move,
   and describe what you should do instead — reference section 3's guidance
   specifically.
