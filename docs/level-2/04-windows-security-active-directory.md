# 04 · Windows Security & Active Directory Basics

Most enterprise networks run on **Active Directory (AD)** — Microsoft's
directory service for managing users, computers, and permissions at scale.
AD is also the single most targeted piece of infrastructure in enterprise
attacks: compromise the domain controller and you effectively own every
Windows machine on the network. This module covers AD's core concepts and
the defensive controls that matter most, using a free lab environment.

!!! note "Lab setup"
    Use a local lab: a Windows Server evaluation VM (free 180-day trial
    from Microsoft) promoted to a domain controller, plus a Windows 10/11
    VM joined to that domain. Never practice against a production or
    third-party AD environment.

## 1. Active Directory core concepts

| Term | Meaning |
|---|---|
| **Domain** | A logical grouping of users/computers under one authority |
| **Domain Controller (DC)** | Server holding the AD database, authenticates every login |
| **Organizational Unit (OU)** | Container for organizing objects and applying Group Policy |
| **Group Policy Object (GPO)** | Centrally-managed configuration pushed to users/computers |
| **Kerberos** | The authentication protocol AD uses (ticket-based, no passwords sent over the wire after initial login) |
| **NTLM** | Older, weaker Windows authentication protocol, still enabled by default in many environments for compatibility |
| **Forest / Trust** | Multiple domains grouped together, optionally with trust relationships between them |

## 2. Why AD is the top attacker target

```
Workstation compromise (phishing) → local admin →
  credential dumping (LSASS memory) → lateral movement →
  Domain Admin compromise → full domain control
```

This chain — known as the standard AD attack path — is why domain
controllers get disproportionate defensive attention: a single
misconfiguration several hops away from the DC can still lead to full
compromise.

## 3. Defensive baseline: Group Policy hardening

Key GPO settings every AD environment should enforce (Group Policy
Management Console → create/edit a GPO linked to your domain):

```
Computer Configuration > Policies > Windows Settings > Security Settings
  Account Policies > Password Policy
    Minimum password length: 14+
    Password history: 24 remembered
  Account Lockout Policy
    Lockout threshold: 5 attempts
    Lockout duration: 15+ minutes
  Local Policies > Audit Policy
    Audit logon events: Success, Failure
    Audit account management: Success, Failure
```

## 4. Least privilege in AD

The most common real-world AD misconfiguration is **privilege sprawl** —
too many accounts in Domain Admins, or standard users with local admin on
their own workstation.

```powershell
# List members of Domain Admins -- this group should be small and audited
Get-ADGroupMember -Identity "Domain Admins" -Recursive

# Find accounts that never expire (a red flag for stale/forgotten accounts)
Get-ADUser -Filter {PasswordNeverExpires -eq $true} -Properties PasswordNeverExpires

# Find accounts with no logon in 90+ days -- candidates for disabling
Get-ADUser -Filter * -Properties LastLogonDate |
  Where-Object {$_.LastLogonDate -lt (Get-Date).AddDays(-90)}
```

**Tiered administration model** — the defensive standard: Tier 0 (domain
controllers, AD itself), Tier 1 (servers), Tier 2 (workstations) each get
*separate* admin accounts that never cross tiers, so a compromised
workstation admin credential can't be used to touch the DC.

## 5. Credential protection

```powershell
# Enable Credential Guard (Windows 10/11 Enterprise) -- isolates LSASS
# secrets in a virtualized container, defeating common credential-dumping
# tools even with local admin/SYSTEM access
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V

# Restrict Group Policy: block interactive logon by service accounts
# Deny log on locally / Deny log on through RDS
```

**LAPS (Local Administrator Password Solution)** — randomizes and
rotates the local admin password on every domain-joined machine, storing
it in AD with access-controlled read permission, so a leaked local admin
password on one machine doesn't grant admin on every machine (a classic
lateral-movement technique called "pass the hash" relies on identical
local admin passwords across the fleet).

## 6. Auditing and detection basics

```powershell
# Key event IDs to monitor (feeds Level 2 Module 6 / Level 3 Module 5)
# 4624 - successful logon
# 4625 - failed logon
# 4672 - special privileges assigned (admin-equivalent logon)
# 4720 - user account created
# 4732 - member added to a security-enabled local group
# 4768/4769 - Kerberos ticket requests (relevant to Kerberoasting detection)

Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} -MaxEvents 20
```

A sudden spike in event 4769 (Kerberos service ticket requests) across many
accounts in a short window is a classic **Kerberoasting** indicator — an
attacker requesting tickets for service accounts in bulk to crack their
password hashes offline.

## 7. Attack surface reduction checklist

- Disable SMBv1 domain-wide (`Set-SmbServerConfiguration -EnableSMB1Protocol $false`) — legacy protocol with known critical vulnerabilities (EternalBlue).
- Disable NTLM where Kerberos can be used instead; audit NTLM usage first with `Set-ADForestMode` restrictions.
- Enforce LDAP signing and channel binding to prevent relay attacks.
- Restrict who can query AD for service account details (mitigates Kerberoasting reconnaissance).
- Patch domain controllers promptly — many AD-specific critical CVEs (e.g. Zerologon) exist specifically in DC-facing protocols.

## Key terms

| Term | Meaning |
|---|---|
| **Kerberoasting** | Requesting service tickets to crack service account passwords offline |
| **Pass the hash** | Using a stolen password hash directly for authentication instead of the plaintext password |
| **Tiered administration** | Separating admin privilege by asset criticality so a low-tier compromise can't reach high-tier systems |
| **LAPS** | Local Administrator Password Solution — randomizes/rotates local admin passwords per machine |
| **Credential Guard** | Windows feature isolating credential material from normal OS memory access |
| **Zerologon** | Critical 2020 vulnerability (CVE-2020-1472) allowing domain controller compromise via a Netlogon flaw |

## Exercise

1. Build the lab: one Windows Server DC, one joined Windows 10/11 client.
2. Create three OUs (Finance, IT, Sales) and apply a password-policy GPO
   to one of them; verify with `gpresult /r` on a joined client.
3. Run the `Get-ADGroupMember` and stale-account PowerShell queries above
   against your lab domain and record the results.
4. Configure LAPS in your lab following Microsoft's official LAPS setup
   guide, and verify a randomized password appears in AD for your client.
5. Written answer: explain, in your own words, why Domain Admins should
   never be used as a daily-driver account for routine IT work.
