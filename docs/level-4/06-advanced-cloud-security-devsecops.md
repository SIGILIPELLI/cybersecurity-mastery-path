# 06 · Advanced Cloud Security & DevSecOps

Level 3 Module 6 covered cloud security architecture. This module
focuses on embedding security directly into the software delivery
pipeline — DevSecOps — so security scales with deployment velocity
instead of becoming a bottleneck fought at every release.

## 1. Shifting left: where security fits in the pipeline

```
Design -> Code -> Build -> Test -> Deploy -> Operate
  |         |        |        |        |         |
Threat    SAST/    Dependency  DAST/   IaC scan  Runtime
model     secrets   scanning   pentest  (Lvl3-6)  monitoring
          scanning
```

Every stage catches classes of issues the later stages either can't see
or catch far more expensively — a hardcoded secret caught by a pre-commit
hook costs seconds; the same secret caught after it's live in production
and possibly already indexed by a scanner bot costs an incident response.

## 2. Static Application Security Testing (SAST) in CI

```yaml
# GitHub Actions example: fail the build on high-severity SAST findings
name: security-scan
on: [pull_request]
jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Semgrep
        run: |
          pip install semgrep
          semgrep --config=auto --error --severity=ERROR .
```

SAST catches patterns like SQL injection via string concatenation
(Level 2 Module 7) directly in source code, before it's ever deployed.

## 3. Dependency and supply chain scanning

Modern applications are mostly third-party code — the software supply
chain is now a primary attack surface:

```bash
# Scan dependencies for known CVEs
npm audit --audit-level=high
pip-audit

# Generate a Software Bill of Materials (SBOM) -- an inventory of every
# component and its provenance, required by many enterprise/government
# procurement standards now
syft packages dir:. -o cyclonedx-json > sbom.json
```

```bash
# Verify artifact integrity/provenance (SLSA-style) before deployment --
# was this exact artifact actually built by our trusted CI pipeline?
cosign verify --key cosign.pub myregistry/app:1.4.2
```

Real-world supply chain compromises (a popular open-source package
taken over and laced with malware, then pulled in automatically by
thousands of downstream builds) make dependency provenance verification
as important as scanning for known CVEs.

## 4. Secrets scanning and prevention

```bash
# Pre-commit hook: block a commit containing an obvious secret pattern
detect-secrets scan --baseline .secrets.baseline

# CI-side scanning of the full history, not just the diff, in case a
# secret leaked in an earlier commit
gitleaks detect --source . --verbose
```

If a secret does leak into version control, rotating it is not optional
even after removal from the codebase — git history retains it
indefinitely unless the repository history itself is rewritten, and by
then it may already be compromised.

## 5. Dynamic testing and IaC scanning integrated into CD

```yaml
# Pipeline stage: scan Terraform before it's ever applied
- name: IaC Security Scan
  run: checkov -d ./infrastructure/ --compact

# Pipeline stage: run a lightweight DAST scan against a staging deploy
- name: DAST
  run: |
    docker run owasp/zap2docker-stable zap-baseline.py \
      -t https://staging.internal.example
```

## 6. Policy as code gating deployment

```yaml
# OPA/Conftest: reject a deployment manifest that violates policy,
# enforced automatically in the pipeline, not by manual review
package main

deny[msg] {
  input.kind == "Deployment"
  not input.spec.template.spec.securityContext.runAsNonRoot
  msg := "Deployments must set runAsNonRoot"
}
```

## 7. Runtime protection and drift detection

Security doesn't stop at deploy — runtime tools detect when a running
system diverges from its known-good, scanned state:

```bash
# Detect a container that started with different capabilities/mounts
# than what was scanned and approved pre-deploy
falco -r /etc/falco/falco_rules.yaml
```

```bash
# Detect infrastructure drift -- someone manually changed something
# outside of the IaC pipeline, bypassing all prior scanning
terraform plan -detailed-exitcode
```

## 8. Culture: security as a shared responsibility, not a gate

The most effective DevSecOps programs treat security tooling as fast,
actionable feedback for developers (inline PR comments, clear
remediation guidance) rather than a slow, opaque gate that blocks
releases without explanation — the latter breeds workarounds and
resentment; the former gets fixes merged the same day they're found.

## 9. Checklist

- [ ] SAST, dependency, and secrets scanning run automatically in CI
- [ ] SBOM generated per build; artifact provenance verified before deploy
- [ ] IaC scanned pre-apply; policy-as-code gates non-compliant deployments
- [ ] Leaked secrets are rotated immediately, not just removed from code
- [ ] Runtime drift and anomaly detection cover the post-deploy gap
- [ ] Findings delivered to developers as fast, actionable feedback

## What's next

Module 7 scales incident response processes to match organizations
operating at this level of deployment velocity and cloud complexity.
