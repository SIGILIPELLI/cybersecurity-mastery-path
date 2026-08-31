# 09 · Compliance & Frameworks

Compliance frameworks translate "we should be secure" into specific,
auditable requirements. Understanding them matters even for hands-on
technical roles, because they define which controls get budget,
priority, and executive attention.

## 1. Why frameworks exist

Frameworks give organizations a shared, testable vocabulary for security
maturity, let auditors and regulators verify claims objectively, and
give security teams leverage: "we need to fix this" competes for budget
against every other business priority, but "we are out of compliance
with a control required for PCI DSS certification, which we need to
keep accepting card payments" gets funded.

## 2. Major frameworks at a glance

| Framework | Focus | Typical audience |
|---|---|---|
| **NIST CSF 2.0** | Risk-based, voluntary, broadly applicable | Any organization |
| **ISO/IEC 27001** | Certifiable Information Security Management System | Global, especially enterprise/vendor requirements |
| **PCI DSS** | Cardholder data protection | Anyone storing/processing/transmitting card data |
| **HIPAA** | Protected health information | US healthcare and business associates |
| **SOC 2** | Trust service criteria (security, availability, etc.) | SaaS/service providers, driven by customer demand |
| **GDPR** | Personal data of EU residents | Any org handling EU resident data, regardless of location |

## 3. NIST Cybersecurity Framework structure

```
Govern -> Identify -> Protect -> Detect -> Respond -> Recover
```

Each function breaks into categories and subcategories mapped to
concrete controls. For example, under **Protect**:

```
PR.AA (Identity Management, Authentication, and Access Control)
  PR.AA-01: Identities and credentials are managed
  PR.AA-05: Access permissions incorporate least privilege
```

This maps directly to work from earlier levels — Level 1 Module 5's
access control principles, Level 2's IAM hardening — showing that
frameworks formalize practices you're already learning, not a separate
skill set.

## 4. ISO 27001: the ISMS and Annex A

ISO 27001 certifies an **Information Security Management System** — a
management process (risk assessment, treatment, continuous improvement),
not just a checklist of technical controls. Annex A lists 93 controls
organized into themes (Organizational, People, Physical, Technological)
that the ISMS process selects from based on the organization's own risk
assessment — an org justifies *excluding* a control as much as including
one, in a Statement of Applicability.

## 5. PCI DSS: the 12 requirements

```
1.  Install and maintain network security controls
2.  Apply secure configurations to all system components
3.  Protect stored account data
4.  Protect cardholder data with strong cryptography during transmission
5.  Protect all systems from malware
6.  Develop and maintain secure systems and software
7.  Restrict access by business need to know
8.  Identify users and authenticate access
9.  Restrict physical access to cardholder data
10. Log and monitor all access to system components and cardholder data
11. Test security of systems and networks regularly
12. Support information security with organizational policies
```

Scope matters enormously here: reducing the number of systems that touch
card data (network segmentation, tokenization) shrinks the compliance
burden — a lesson that generalizes to every framework: **minimizing
scope of sensitive data exposure reduces both risk and audit cost.**

## 6. SOC 2 and the trust service criteria

SOC 2 reports (Type I: point-in-time; Type II: over a period, typically
6-12 months) attest to controls across up to five criteria: Security
(required), Availability, Processing Integrity, Confidentiality, and
Privacy. Unlike ISO 27001 (a certification) or PCI DSS (validated
compliance), SOC 2 is an *attestation report* produced by an independent
auditor — commonly requested by enterprise customers as part of vendor
due diligence before signing a contract.

## 7. Mapping one control across multiple frameworks

Mature programs build a **unified control framework** that maps once to
many regulatory requirements, avoiding duplicated audit effort:

```
Control: "MFA required for all remote access to systems processing
          sensitive data"
  Maps to:
    NIST CSF   PR.AA-03
    ISO 27001  Annex A 8.5
    PCI DSS    Requirement 8.4.2
    SOC 2      CC6.1
```

Tools like GRC platforms (Vanta, Drata, or spreadsheet-based mappings
for smaller orgs) automate this cross-mapping so one piece of evidence
satisfies multiple audits.

## 8. Continuous compliance vs. point-in-time audits

Historically, compliance meant scrambling before an annual audit.
Modern practice favors **continuous compliance**: automated evidence
collection (screenshots, config exports, log samples) gathered
continuously so an audit is a formality confirming what's already known,
not a fire drill — directly leveraging the CSPM (Module 6) and SIEM
(Module 5) tooling already built.

## 9. Checklist

- [ ] Applicable frameworks identified based on industry, data handled, customers
- [ ] Controls mapped once across all applicable frameworks, not duplicated per audit
- [ ] Scope of sensitive data (esp. PCI) minimized through segmentation/tokenization
- [ ] Evidence collection automated/continuous rather than pre-audit scramble
- [ ] Gaps tracked as remediation items with owners and deadlines

## What's next

Module 10's capstone project produces a full penetration test report —
the kind of documented evidence that feeds directly into requirements
like PCI DSS Requirement 11 and ISO 27001's continuous testing controls.
