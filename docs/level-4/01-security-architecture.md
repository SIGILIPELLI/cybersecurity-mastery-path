# 01 · Security Architecture

Level 4 shifts perspective: from executing individual tests and building
individual detections, to designing the *system* those activities operate
within. Security architecture is the discipline of building security in
as a structural property, not bolting it on afterward.

## 1. Defense in depth, revisited at scale

Every prior level touched pieces of this; architecture makes it explicit
and deliberate, layer by layer, so that no single control failure is
catastrophic:

```
Perimeter    -> firewalls, WAF, DDoS protection
Network      -> segmentation, NAC, east-west controls
Host         -> hardening, EDR, patching
Identity     -> MFA, least privilege, PAM
Application  -> secure SDLC, input validation
Data         -> encryption at rest/in transit, DLP
Detection    -> SIEM, logging, threat hunting
```

The architect's question for every layer: "if this control fails or is
bypassed, what stops the attacker at the next layer?" A design where the
answer is "nothing" is not defense in depth, it's a single point of failure.

## 2. Security architecture frameworks

- **SABSA** — business-driven, traces every control back to a business
  requirement ("why does this control exist? What business risk does it
  address?").
- **TOGAF** with a security overlay — enterprise architecture with
  security woven through each domain (business, data, application,
  technology).
- **NIST SP 800-160** — systems security engineering, treats security as
  an engineering discipline with the same rigor as reliability or
  performance engineering.

## 3. Threat modeling as an architectural input

Threat modeling happens *before* a system is built, so security shapes
the design rather than being retrofitted:

```
STRIDE categories applied to a new payment API design:
S - Spoofing:          Can an attacker impersonate a legitimate caller?
T - Tampering:          Can request/response data be modified in transit?
R - Repudiation:        Can a user deny performing a transaction?
I - Information disclosure: Does an error message leak internal details?
D - Denial of service:  Can one client exhaust shared resources?
E - Elevation of privilege: Can a standard user reach admin-only functions?
```

Each identified threat gets a mitigating architectural decision *before*
a line of code is written — e.g., mutual TLS for spoofing/tampering,
structured audit logging for repudiation, rate limiting per-client for DoS.

## 4. Zero trust as an architectural principle

Zero trust (detailed fully in Module 3) starts here as a design
philosophy: no implicit trust based on network location. Every access
request is authenticated, authorized, and encrypted, whether it
originates "inside" the corporate network or not — an architectural
shift away from the old "hard shell, soft center" perimeter model that
collapses catastrophically once an attacker gets past the shell.

## 5. Reference architecture example: a three-tier web application

```
Internet
  -> WAF/CDN                    (filters known attack patterns, absorbs DDoS)
  -> Load Balancer (TLS term.)   (public-facing, minimal attack surface)
  -> App tier (private subnet)   (no direct internet route, egress-filtered)
  -> Data tier (private subnet)   (reachable only from app tier, encrypted at rest)
  -> Identity provider (SSO/MFA) (all cross-tier calls authenticated)
  -> Centralized logging/SIEM    (every tier ships logs here)
```

Every arrow in this diagram is a trust boundary decision, and every trust
boundary should be traceable back to a threat model finding — this is
what separates a security architecture from a network diagram.

## 6. Security architecture review process

New systems and major changes should go through architecture review
*before* build, not as a pre-launch gate that's too late to act on
meaningfully:

```
1. Design submitted with data flow diagram + threat model
2. Security architect reviews trust boundaries, authN/authZ, data classification
3. Findings categorized: blocking (must fix pre-build) vs advisory
4. Sign-off recorded, re-reviewed if scope materially changes
```

## 7. Balancing security against usability and cost

An architecture that is theoretically perfect but unusable gets
bypassed by frustrated users (shadow IT, shared passwords, disabled
controls) — genuinely undermining security more than a pragmatic, adopted
design would. Good architecture explicitly documents trade-offs made and
why, so future reviewers understand the reasoning rather than
second-guessing a decision made with context they don't have.

## How It Actually Works: how a threat model actually derives architectural controls from a data-flow diagram

Architectural threat modeling produces controls, not just a document,
through a mechanical process rather than intuition. It starts by drawing a
**data flow diagram**: every process (a service), every data store
(a database, a queue), every external entity (a user, a third-party API),
and every flow between them, with **trust boundaries** drawn wherever a flow
crosses from one privilege/ownership context into another (the same
trust-boundary concept from Level 1 Module 1, now applied at system scale).
The mechanical step is then walking every element against STRIDE
systematically: for each *data flow crossing a trust boundary*, ask
Spoofing (can the receiver be sure who sent this?), Tampering (can the data
be modified in transit or at rest without detection?), and so on for all six
categories — and for each "yes," the diagram directly names the missing
control category (authentication for Spoofing, integrity checking/HMAC for
Tampering, encryption for Information Disclosure). This is why threat
modeling done properly on the three-tier example above is reproducible
across analysts: two people modeling the same diagram converge on largely
the same control set, because the method is a structured walk of a finite
graph, not free-form speculation.

**Balancing security against usability and cost** has a real quantitative
framing underneath the qualitative discussion: for any proposed control, the
architecture review should estimate **annualized loss expectancy (ALE)** —
`Single Loss Expectancy × Annual Rate of Occurrence` — for the risk the
control mitigates, and compare it against the control's own annualized cost
(license, engineering time, ongoing friction measured in lost productivity
or user attrition). A control whose cost exceeds the ALE it prevents is a
net negative investment even though it "improves security" in isolation —
this is the same Risk ≈ Threat × Vulnerability × Impact model from Level 1
Module 1, now applied on the *defender's* side of the ledger to decide which
of many technically valid controls actually belong in a given architecture,
which is the concrete reason "add every control" is not simply the
maximally secure choice organizations can afford to make.

## 8. Checklist

- [ ] Defense-in-depth layers explicitly documented for critical systems
- [ ] Threat modeling performed before major system design is finalized
- [ ] Trust boundaries in architecture diagrams traceable to specific controls
- [ ] Zero trust principles applied to new design work by default
- [ ] Architecture review is a pre-build gate, not a pre-launch afterthought
- [ ] Trade-off decisions documented, not just the final design

## What's next

Module 2 builds the proactive detection capability — threat hunting —
that a well-architected environment needs to actually exercise.
