# 03 · Zero Trust Architecture

"Never trust, always verify" — zero trust replaces the old model of a
hardened perimeter with a soft, trusted interior with continuous
verification of every access request, regardless of network location.

## 1. Why the perimeter model failed

The traditional model assumed anything inside the corporate network was
trustworthy. Two realities broke this assumption for good: remote work
means there often *is* no single perimeter, and once an attacker gets a
foothold anywhere inside (one phished laptop), the old model gave them
broad implicit trust to move laterally undetected — exactly the pattern
Level 3 Module 8's red team exercises are built to expose.

## 2. Core zero trust principles (NIST SP 800-207)

```
1. All data sources and computing services are treated as resources
2. All communication is secured regardless of network location
3. Access to individual resources is granted per-session, not per-network
4. Access is determined by dynamic policy (identity, device health,
   behavior) -- not a static network location
5. The organization monitors and measures the integrity/security posture
   of all owned/associated assets continuously
6. All resource authentication/authorization is dynamic and strictly
   enforced before access is allowed
7. The organization collects as much information as possible about
   current state of assets, network, and communications to improve
   posture over time
```

## 3. The policy decision point / policy enforcement point model

```
User/Device -> Policy Enforcement Point (PEP) -> Resource
                       |
              Policy Decision Point (PDP)
              (evaluates: identity, device posture,
               location, behavior, resource sensitivity)
```

Every single access request — not just the initial login — passes
through this evaluation. A user authenticated an hour ago whose device
posture just changed (e.g., disk encryption disabled, EDR agent
stopped) can be denied on the *next* request without waiting for
re-authentication.

## 4. Identity as the new perimeter

```
Strong identity foundation required for zero trust:
  - Phishing-resistant MFA (FIDO2/WebAuthn, not just SMS OTP)
  - Continuous risk-based authentication (impossible travel, device change)
  - Just-in-time, time-bound privileged access instead of standing admin rights
```

```bash
# Conditional access policy example (Azure AD / Entra ID style)
# Require compliant device AND MFA for access to sensitive apps,
# regardless of whether the request comes from the office network
```

## 5. Micro-segmentation

Network segmentation (Level 2-3) becomes far more granular under zero
trust — not just "prod vs. dev" but per-workload policy:

```yaml
# Kubernetes NetworkPolicy from Level 3 Module 7 is a zero trust building
# block: default-deny, explicit per-service allow
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: zero-trust-default-deny
spec:
  podSelector: {}
  policyTypes: ["Ingress", "Egress"]
```

Software-defined perimeters (SDP) extend this beyond containers — a user
or service only sees the network path to resources it's explicitly
authorized for, everything else is invisible, not merely blocked.

## 6. Device trust and posture assessment

```
Device posture signals evaluated continuously:
  - OS patch level current?
  - Disk encryption enabled?
  - EDR agent running and healthy?
  - Not jailbroken/rooted?
  - Managed by MDM vs. unmanaged personal device?
```

A device failing posture checks can be granted reduced access (e.g.,
web-only email, no local downloads) rather than a binary allow/deny —
supporting BYOD without granting BYOD devices full trust.

## 7. Migration reality: zero trust is a journey, not a product

No organization flips a switch to "zero trust" overnight. A practical
roadmap:

```
Phase 1: Strong identity (MFA everywhere, eliminate standing admin access)
Phase 2: Device posture integrated into access decisions
Phase 3: Micro-segmentation of highest-value resources first
Phase 4: Continuous, risk-based policy evaluation across all resources
Phase 5: Full dynamic PDP/PEP model with real-time telemetry feedback
```

Vendors selling "zero trust in a box" are selling one component — real
zero trust is an architectural commitment (Module 1) spanning identity,
network, device, application, and data controls together.

## 8. Checklist

- [ ] MFA is phishing-resistant (FIDO2/WebAuthn) for privileged access
- [ ] Access decisions incorporate device posture, not identity alone
- [ ] Standing privileged access replaced with just-in-time elevation
- [ ] Micro-segmentation applied to highest-value resources first
- [ ] Access re-evaluated continuously, not only at initial login
- [ ] A phased roadmap exists rather than treating zero trust as one project

## What's next

Module 4 covers the cryptography and PKI foundations that make strong
identity and encrypted-everywhere communication actually possible.
