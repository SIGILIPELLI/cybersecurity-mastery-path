# 05 · Authentication & Access Control

Modules 1–4 covered *what* you're protecting and the cryptography that
enforces confidentiality and integrity. This module covers the layer that
decides **who gets in and what they're allowed to do once they're in** — the
mechanism behind nearly every real-world breach headline that starts with
"attackers gained access using..."

## 1. Authentication vs. authorization — the distinction that matters

These two words get conflated constantly, and the difference is the basis of
every access-control system you'll ever configure:

| | Authentication | Authorization |
|---|---|---|
| Question answered | "Who are you?" | "What are you allowed to do?" |
| Happens | First, at login | After authentication, on every action |
| Example | Entering a correct password | Being allowed to view *this specific* invoice |
| Failure mode | Someone impersonates another identity | An authenticated user does something they shouldn't be able to |

A system can authenticate you perfectly (you really are who you say you are)
and still authorize you incorrectly (you can now edit invoices you have no
business touching) — this exact failure, called **broken access control**, is
the single most common category in the OWASP Top 10 (Module 6).

## 2. Authentication factors

Authentication proves identity using one or more of three factor categories:

| Factor type | Examples | Weakness |
|---|---|---|
| **Something you know** | Password, PIN, security question | Can be guessed, phished, or leaked in a breach |
| **Something you have** | Phone (SMS/authenticator app), hardware key, smart card | Can be lost, stolen, or (for SMS) intercepted via SIM-swap |
| **Something you are** | Fingerprint, face, iris (biometrics) | Cannot be changed if compromised; false-accept/reject trade-offs |

**Multi-factor authentication (MFA)** combines two or more *different*
categories — a password plus an authenticator app code is MFA; a password
plus a security question is not, because both are "something you know" and
both can be phished the same way.

| MFA method | Security level | Notes |
|---|---|---|
| SMS code | Weakest MFA | Vulnerable to SIM-swapping and SS7 interception; better than nothing |
| Authenticator app (TOTP) | Strong | Time-based one-time codes, generated offline, not interceptable in transit |
| Push notification | Strong, but watch for "MFA fatigue" | Attackers spam approval requests hoping for an accidental tap |
| Hardware security key (FIDO2/U2F) | Strongest | Phishing-resistant — bound cryptographically to the real site's domain |

!!! tip "MFA is the single highest-leverage control an individual can enable"
    A very large share of account-takeover attacks are stopped cold by MFA,
    because the attacker who has your leaked password still lacks the second
    factor. If you enable exactly one thing from this entire course on your
    own accounts today, make it MFA — starting with email and financial
    accounts, since a compromised email is usually the key to resetting
    everything else.

## 3. Password policy, done well and done badly

Password policy has a well-documented history of good intentions producing
bad outcomes. Modern guidance (NIST SP 800-63B) reversed several long-standing
"best practices":

| Old advice | Why it backfired | Current guidance |
|---|---|---|
| Force a password change every 90 days | Users respond with `Password1`, `Password2`, `Password3`... | Only force a change on evidence of compromise |
| Require complex special-character rules | Users respond with predictable substitutions (`P@ssw0rd!`) that automated tools already model | Prioritize **length** over complexity |
| Allow unlimited login attempts | Enables brute-force and credential-stuffing attacks | Rate-limit and lock out after repeated failures |
| No password reuse checking | Users recycle already-breached passwords | Check new passwords against known-breached lists |

The practical modern recommendation: a **long passphrase** (`correct horse
battery staple` — four random words, ~25+ characters) is both easier to
remember and dramatically harder to brute-force than a short complex password
(`Tr0ub4dor&3`), because brute-force difficulty scales with the *length* of
the search space far more than with character-set complexity.

**Password managers** solve the practical problem this creates — remembering
a unique long passphrase per site is unrealistic without one — by generating
and storing a unique strong password per account behind a single master
passphrase (protected with MFA of its own).

## 4. Authorization models

Once a user is authenticated, something has to decide what they can do.
Three common models, increasing in flexibility:

### Discretionary Access Control (DAC)

The resource *owner* decides who else gets access — this is how Linux file
permissions (Module 3) and most file-sharing systems work. Simple, but scales
poorly: permissions sprawl as owners make individual, inconsistent decisions.

### Role-Based Access Control (RBAC)

Permissions are attached to **roles**, and users are assigned to roles, rather
than granting permissions to individuals directly.

```
User "alice" --> assigned role "billing-admin" --> role grants:
                                                      - view invoices
                                                      - edit invoices
                                                      - export reports
```

When Alice changes teams, you change her role assignment — not dozens of
individual permission grants scattered across systems. This is the dominant
model in real organizations and is what Level 2 Module 4 (Active Directory)
builds on extensively.

### Attribute-Based Access Control (ABAC)

Access decisions consider multiple **attributes** at evaluation time — not
just "who," but "from where," "at what time," "on what device." Example
policy: *"Finance-department users may access the payroll system only from a
company-managed device, during business hours, from an approved country."*
ABAC is more flexible than RBAC but more complex to reason about and audit —
it underlies modern **Zero Trust** architectures (Level 4 Module 3).

| Model | Decision basis | Best for |
|---|---|---|
| DAC | Owner's discretion | Personal files, small teams |
| RBAC | Role membership | Most business applications |
| ABAC | Multiple contextual attributes | High-security, context-sensitive environments |

## 5. Least privilege in access control, concretely

Module 3 introduced least privilege at the OS level; here it becomes a
concrete design rule for any access-control system you build or audit:

- Grant access to the **specific resource** needed, not the whole category
  ("this one invoice" not "all invoices").
- Grant the **specific action** needed, not full control (read-only where
  write isn't required).
- **Time-bound** elevated access where possible — a temporary admin grant
  that expires, rather than a permanent one.
- **Review regularly.** Access accumulates as people change roles and nobody
  removes the old grants — this "privilege creep" is one of the most common
  real-world audit findings.

!!! example "Privilege creep in practice"
    An employee moves from the finance team to marketing. Six months later,
    they still have write access to the payroll system from their old role —
    nobody revoked it. This isn't a hypothetical; access reviews consistently
    turn up exactly this pattern, and it's precisely the kind of finding
    you'll be looking for in Module 10's home lab assessment.

## How It Actually Works: how a password hash actually resists cracking, and how TOTP codes are generated

A password is never stored — what's stored is `hash(password + salt)`, and
each part of that formula defends against a specific attack. The **salt** (a
random value stored alongside the hash) exists because without it, two users
with the same password produce identical hashes, and an attacker can
precompute a **rainbow table** — a giant lookup of `hash → plaintext` — once
and reuse it against every account in every breached database it ever
targets. A unique salt per user forces the attacker to redo the work for
every single hash, turning one lookup into millions of individual cracking
attempts.

Modern password hashing also intentionally makes the hash *slow*, which is
the opposite of what you want from SHA-256 (built for speed). Algorithms
like **bcrypt** and **Argon2** run a deliberately expensive internal loop —
bcrypt performs a configurable number of Blowfish key-schedule rounds
(the "cost factor," typically 2^12), and Argon2 additionally forces the
computation to touch a large, configurable block of memory rather than pure
CPU cycles. That memory requirement specifically defeats GPUs and ASICs,
which are cracking-fast for cheap arithmetic but have comparatively little
fast memory per parallel core — so Argon2 narrows the gap between an
attacker's cracking rig and a legitimate login server's single hash
verification, rather than letting the attacker parallelize a billion cheap
hashes per second.

**TOTP** (Time-based One-Time Password — the six-digit code in an
authenticator app) is a second, independent factor built on HMAC:

```
counter      = floor(current_unix_time / 30)
HMAC_output  = HMAC-SHA1(shared_secret, counter)
code         = last 31 bits of HMAC_output, truncated & mod 10^6
```

Both the app and the server independently compute this from the same shared
secret (exchanged once, at enrollment, via the QR code) and the same current
time — no network round-trip is needed, which is why TOTP works offline.
This is also exactly why authenticator app clocks must stay roughly
synchronized with the server (most implementations tolerate ±1 time step):
if the clocks drift too far apart, the counter values diverge and every code
fails, and it's why "time drift" is a real support issue for TOTP
deployments.

**RBAC vs. ABAC under the hood**: a role-based system evaluates access as a
simple set-membership check (`user.roles ∩ resource.required_roles ≠ ∅`),
resolved once at login and cached in the session — fast, but coarse.
Attribute-based access control evaluates a policy expression against
live attributes at request time (`user.department == resource.owner_department
AND time.hour BETWEEN 9 AND 18`), which requires re-evaluating the policy on
every request rather than once per session — more precise, at the cost of
being slower and harder to audit, which is the real engineering trade-off
behind "why doesn't everyone just use ABAC everywhere."

## Key terms

| Term | Meaning |
|---|---|
| **Authentication** | Verifying identity — "who are you?" |
| **Authorization** | Determining permitted actions — "what can you do?" |
| **MFA** | Requiring two or more *different* factor categories to authenticate |
| **TOTP** | Time-based One-Time Password — the code an authenticator app generates |
| **RBAC** | Role-Based Access Control — permissions attached to roles, not individuals |
| **ABAC** | Attribute-Based Access Control — decisions based on contextual attributes |
| **Least privilege** | Grant the minimum access necessary, nothing more |
| **Privilege creep** | Accumulated, unrevoked access left over from past roles |
| **Broken access control** | An authenticated user able to act beyond their intended permissions |

## Exercise

No special tools required — an authenticator app (Google Authenticator, Authy,
or similar, free) if you don't already have one.

1. **Audit your own MFA coverage.** List your five most important online
   accounts (email, banking, primary work account, etc.). For each, check
   whether MFA is available and whether you have it enabled. Enable it on at
   least one account that currently lacks it, preferring an authenticator app
   over SMS where offered.

2. **Evaluate a password policy.** Find the password requirements page for
   three services you use (often visible at sign-up or in account settings).
   For each, note whether it follows current NIST-style guidance (prioritizing
   length, not forcing arbitrary complexity/rotation) or the outdated pattern
   from section 3 — and write one sentence on which is more secure and why.

3. **Design an RBAC scheme.** Imagine a small company with a shared document
   system holding contracts, invoices, and HR records. Design three roles
   (e.g., `finance`, `hr`, `general-staff`), and for each, specify which of the
   three document categories they can read and which they can edit. Then
   describe, in one sentence per role, what would go wrong if you granted DAC
   (individual, ad hoc) permissions instead.

4. **Spot the access-control failure.** Describe a realistic scenario (real or
   invented) where authentication succeeds correctly but authorization fails —
   i.e., a genuine broken-access-control case. Name which of DAC/RBAC/ABAC, if
   properly implemented, would have prevented it.
