# 10 · Capstone — Enterprise Security Program Design

This capstone brings together every module from Level 1 through Level 4
into a single deliverable: a complete enterprise security program
design for a fictional (or your own) organization — the kind of document
a CISO presents to a board.

## 1. Project scope

Choose a realistic organization profile to design for — pick one, or
use your own workplace as inspiration (genericized, no real confidential
details):

```
Example profile: "MidTier Health" -- a 2,000-employee healthcare
  technology company handling patient data (HIPAA), processing card
  payments (PCI DSS), and selling SaaS to hospital systems (customers
  demanding SOC 2 reports).
```

## 2. Section 1 — Business and risk context

Start from the business, not the technology, exactly as Module 9's risk
management principles require:

```
- What does the organization do, and what data/systems matter most?
- What regulatory frameworks apply? (Level 3 Module 9)
- What's the threat landscape for this sector? (Level 3 Module 4)
- Top 5 business risks, ranked (Module 9's risk register format)
```

## 3. Section 2 — Security architecture

Apply Module 1's architecture principles to design the target-state
environment:

```
- Defense-in-depth layers for this organization's actual systems
- Zero trust roadmap (Module 3) -- current state vs. target, phased
- Reference architecture diagram: how data flows from patient/customer
  intake through to storage, with every trust boundary labeled
- Cryptography/PKI approach for data at rest and in transit (Module 4)
```

## 4. Section 3 — Detection and response capability

```
- SOC operating model chosen and why (Module 8) -- in-house/MSSP/hybrid
- SIEM and detection strategy, mapped to ATT&CK coverage (Level 3 Mod 5)
- Threat hunting cadence and hypothesis sources (Module 2)
- IR plan including incident command structure for a SEV1 (Module 7)
- SOAR automation targets for the highest-volume alert types (Module 5)
```

## 5. Section 4 — Cloud and DevSecOps posture

```
- Multi-account/cloud architecture (Level 3 Module 6)
- CI/CD security gates: SAST, dependency/secrets scanning, IaC scanning
  (Module 6)
- Container/Kubernetes hardening baseline if applicable (Level 3 Mod 7)
```

## 6. Section 5 — Compliance and third-party risk

```
- Control mapping across applicable frameworks (Level 3 Module 9's
  "map once, satisfy many" approach)
- Vendor risk assessment process (Module 9)
- Continuous compliance evidence collection approach
```

## 7. Section 6 — Roadmap and budget

The section that makes this a leadership document rather than a
technical wish list — phase the work realistically:

```
Year 1 Q1-Q2: Close critical gaps (e.g. MFA everywhere, eliminate
  standing admin access, patch the highest-risk legacy system)
Year 1 Q3-Q4: SOC coverage gap closed, SIEM detection coverage baseline
  established against ATT&CK
Year 2: Zero trust phase 2-3, DevSecOps pipeline maturity, SOAR
  automation of top playbooks
Year 3: Advanced threat hunting program, post-quantum crypto readiness
  assessment, continuous compliance automation

Estimated budget by year, roughly allocated across people/tools/services,
with each major line item traceable back to a specific risk it reduces
(Module 9's justification principle).
```

## 8. Section 7 — Executive summary

Written last, read first — one page, framed entirely in business risk
and value terms, following Level 3 Module 10's executive summary style:

```
"This program reduces our top identified business risk — [specific
risk] — from Critical to Medium within 12 months, brings us into
compliance with [framework] ahead of our Q3 audit, and is projected to
cut mean time to detect security incidents by [X]% through the proposed
SOC and detection investments."
```

## 9. Deliverable checklist

- [ ] Program grounded in the organization's actual business risk, not generic best practice
- [ ] Every major technical decision traces back to a module concept from this program
- [ ] Zero trust and cloud/DevSecOps sections include a phased, realistic roadmap
- [ ] Compliance section maps controls once across all applicable frameworks
- [ ] Budget/roadmap section ties spending to specific risk reduction
- [ ] Executive summary is one page, business-framed, written last

## Program complete

This capstone marks the end of the Cybersecurity Mastery Path — from
"what is cybersecurity" in Level 1, through hands-on technical depth in
Levels 2-3, to designing and leading a full enterprise program here in
Level 4. The skills compound: architecture decisions inform detection
strategy, detection findings inform threat intelligence, threat
intelligence informs risk prioritization, and risk prioritization
justifies the budget that funds the whole cycle again.
