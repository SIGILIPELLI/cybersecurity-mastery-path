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

## How It Actually Works: how SAST/dependency scanning integrate into CI as build-graph analysis, and how policy gates enforce without slowing the pipeline

**SAST in CI** runs the same taint-analysis engine described in Level 2
Module 7, but the integration detail that makes it usable at pipeline speed
is **incremental analysis**: rather than re-analyzing the entire codebase's
control-flow/data-flow graph on every commit (expensive — full-repo taint
analysis can take longer than the rest of the build), modern SAST tools
cache the graph from the last analyzed commit and recompute only the
subgraph reachable from changed functions, using the same dependency-graph
diffing idea a build system like Bazel uses for incremental compilation.
This is why SAST scan time in CI scales roughly with the size of a *diff*
rather than the size of the repository, which is what makes running it on
every pull request economically viable instead of only nightly.

**Dependency/supply-chain scanning** works by resolving your project's full
transitive dependency tree, computing a cryptographic hash of each resolved
package version, and checking those hashes/version-CPEs against the same
NVD-backed CVE database mechanism from Level 2 Module 5's vulnerability
scanning — applied to your build manifest instead of a live network host.
The **SBOM** (Software Bill of Materials) this produces is the artifact that
makes this checkable *after* the fact too: because it's a hash-identified,
versioned list, a newly disclosed CVE can be matched against every SBOM an
organization has ever generated to instantly answer "which of our shipped
builds contain the vulnerable version," without re-scanning any running
system.

**Policy-as-code gating** in CD is a synchronous call, at the deployment
step, to the identical Rego/OPA evaluation engine from Level 3 Module 6's
CSPM — the gate blocks the pipeline's next step until the policy evaluation
returns `allow`. This scales to hundreds of deployments per day precisely
because the policy check is a small, fast, local (no external network round
trip in most implementations) function evaluation against the manifest
about to be deployed, not a human review step — the same allow/deny
determinism from IAM evaluation, just invoked automatically at a pipeline
checkpoint instead of at request time. **Runtime drift detection** closes the
loop by continuously re-running the same CSPM diff (Level 3 Module 6) after
deployment, catching the specific case policy-as-code gating cannot: a
resource changed directly in production after the gate already passed it.

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
