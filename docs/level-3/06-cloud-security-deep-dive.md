# 06 · Cloud Security Deep Dive

Level 2 Module 8 covered cloud security fundamentals: shared
responsibility, IAM basics, storage misconfigurations. This module goes
deeper into the tooling and architecture patterns that keep large,
multi-account cloud environments secure.

## 1. Multi-account/multi-project architecture

Large organizations don't run everything in one account — a compromised
dev environment should never be able to reach production data:

```
AWS: Organizations + Control Tower
  Management account (billing, org policy -- nothing else runs here)
    -> Security account (centralized logging, GuardDuty, Security Hub)
    -> Log Archive account (immutable, write-once log storage)
    -> Production OU (workload accounts, strict guardrails)
    -> Dev/Sandbox OU (looser guardrails, isolated from prod)
```

Service Control Policies (SCPs) enforce guardrails that even an account
admin cannot override:

```json
{
  "Effect": "Deny",
  "Action": "s3:PutBucketAcl",
  "Resource": "*",
  "Condition": {
    "StringEquals": { "s3:x-amz-acl": "public-read" }
  }
}
```

## 2. IAM deep dive: least privilege at scale

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::app-data-bucket/reports/*",
    "Condition": {
      "IpAddress": { "aws:SourceIp": "10.0.0.0/16" },
      "Bool": { "aws:MultiFactorAuthPresent": "true" }
    }
  }]
}
```

Key practices at scale: use roles, not long-lived IAM user keys;
require MFA for anything sensitive; use IAM Access Analyzer to find
unused permissions and unintended external access; rotate and
eventually eliminate static credentials in favor of workload identity
federation (OIDC) for CI/CD pipelines.

```bash
# Find permissions granted but never actually used, over 90 days
aws accessanalyzer list-findings --analyzer-arn <arn>
```

## 3. Cloud Security Posture Management (CSPM)

Manual configuration review doesn't scale across hundreds of accounts
and thousands of resources. CSPM tools continuously check configuration
against best-practice benchmarks (CIS Benchmarks):

```bash
# Open-source example: Prowler runs CIS AWS Foundations checks
prowler aws --compliance cis_2.0_aws

# Output flags things like:
# [FAIL] 2.1.1 Ensure all S3 buckets employ encryption-at-rest
# [FAIL] 1.12  Ensure credentials unused for 90 days are disabled
```

## 4. Cloud-native detection and logging

```bash
# AWS: enable GuardDuty (managed threat detection) org-wide
aws guardduty create-detector --enable

# CloudTrail: the audit log of every API call -- must be enabled,
# multi-region, and delivered to an immutable, access-restricted bucket
aws cloudtrail create-trail \
  --name org-trail \
  --s3-bucket-name log-archive-bucket \
  --is-multi-region-trail \
  --enable-log-file-validation
```

A classic real-world incident pattern: an attacker who gains one set of
leaked credentials immediately calls `GetCallerIdentity` and lists IAM
permissions to figure out what they can do — GuardDuty and CloudTrail
alerting on unusual API call patterns is often the only detection
opportunity before damage is done.

## 5. Network security in the cloud

```
VPC design:
  Public subnet   -> only load balancers/bastion, nothing with direct data access
  Private subnet  -> application servers, no direct internet route
  Data subnet     -> databases, reachable only from the private subnet
```

```bash
# Security groups: default-deny, explicit allow only
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp --port 443 \
  --source-group sg-loadbalancer   # from the LB's SG, not 0.0.0.0/0
```

VPC Flow Logs capture network-level metadata for detection and forensics,
the cloud equivalent of NetFlow.

## 6. Secrets management

Hardcoded credentials in code or environment variables are one of the
most common cloud breach root causes. Use a managed secrets service with
automatic rotation instead:

```bash
aws secretsmanager create-secret --name prod/db/password \
  --secret-string '{"username":"app","password":"GENERATED"}'

aws secretsmanager rotate-secret --secret-id prod/db/password \
  --rotation-lambda-arn <rotation-fn-arn> \
  --rotation-rules AutomaticallyAfterDays=30
```

## 7. Infrastructure as Code security

Scan Terraform/CloudFormation *before* deployment, not after:

```bash
# tfsec / checkov scan IaC for misconfigurations pre-deploy
checkov -d ./terraform/

# Example finding:
# CKV_AWS_20: "S3 bucket has an ACL defined which allows public access"
```

Catching a public-bucket misconfiguration in a pull request review is
free; catching it after data has already been exfiltrated is a breach
notification.

## 8. Checklist

- [ ] Multi-account structure separates prod, dev, and security/logging
- [ ] SCPs/org policies enforce guardrails no single account can override
- [ ] IAM least privilege verified continuously (Access Analyzer or equivalent)
- [ ] CSPM tool running against CIS Benchmarks on a schedule
- [ ] CloudTrail/audit logging enabled, multi-region, immutable storage
- [ ] Network segmented (public/private/data tiers), default-deny security groups
- [ ] Secrets in a managed vault with rotation, never hardcoded
- [ ] IaC scanned for misconfigurations before deployment

## What's next

Module 7 extends this into container and Kubernetes environments, which
introduce their own orchestration-layer attack surface.
