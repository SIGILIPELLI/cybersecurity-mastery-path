# 08 · Cloud Security Basics

Cloud platforms (AWS, Azure, GCP) shift *some* security responsibility to
the provider but not all of it — and misunderstanding exactly where the
line falls is the single most common cause of cloud breaches. This module
covers the shared responsibility model and hands-on hardening using AWS's
always-free tier.

## 1. The Shared Responsibility Model

```
Provider is responsible for:      Customer is always responsible for:
  Physical data center security     Identity & access management (IAM)
  Hypervisor/host OS patching       Data encryption configuration
  Network infrastructure            Security group / firewall rules
  Global infrastructure resilience  Application-layer security
                                     What you put in public storage buckets
```

**"Security OF the cloud vs. security IN the cloud."** The provider
secures the infrastructure; you are always responsible for how you
configure and use it. The overwhelming majority of real cloud breaches
are customer misconfigurations, not provider failures — a public S3
bucket, an overly permissive IAM policy, an unencrypted database.

## 2. IAM: least privilege in practice

```json
// WRONG: full admin access granted to an application's service role
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}

// RIGHT: scoped to exactly what the app needs
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::my-app-uploads/*"
}
```

```bash
# Enable MFA on every human IAM user, especially the root account
aws iam enable-mfa-device --user-name alice --serial-number arn:aws:iam::...

# Audit who has admin-equivalent access
aws iam get-account-authorization-details | jq '.UserDetailList[] | select(.GroupList[] | contains("Admins"))'

# Rotate access keys, never commit them to source control
aws iam list-access-keys --user-name alice
```

**Root account rule:** the AWS root account should never be used day-to-
day — lock it behind MFA and a password stored in a vetted secrets
manager, create IAM users/roles for all actual work.

## 3. The public storage bucket problem

Publicly-readable/writable cloud storage buckets are one of the most
common, highest-impact real-world breach causes.

```bash
# Check an S3 bucket's public access settings
aws s3api get-public-access-block --bucket my-bucket

# Enforce account-wide blocking of public access as a baseline default
aws s3control put-public-access-block \
  --account-id 123456789012 \
  --public-access-block-configuration \
    BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

Treat "public by default" as a decision that must be explicit and
reviewed, never an accident — most cloud providers now default new
storage to private specifically because of how often this went wrong.

## 4. Network security groups

```bash
# WRONG: SSH open to the entire internet
aws ec2 authorize-security-group-ingress --group-id sg-123 \
  --protocol tcp --port 22 --cidr 0.0.0.0/0

# RIGHT: restrict to a known IP range (office VPN, bastion host)
aws ec2 authorize-security-group-ingress --group-id sg-123 \
  --protocol tcp --port 22 --cidr 203.0.113.0/24
```

Use a **bastion host** or cloud-native session manager (AWS Systems
Manager Session Manager) instead of exposing SSH/RDP to the internet at
all — this eliminates an entire class of brute-force and credential-
stuffing attacks against management ports.

## 5. Encryption at rest and in transit

```bash
# Enable default encryption on new S3 buckets
aws s3api put-bucket-encryption --bucket my-bucket \
  --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms"}}]}'

# Enable encryption on an RDS database
aws rds create-db-instance --storage-encrypted --kms-key-id alias/my-key ...
```

Encryption in transit: enforce TLS for every load balancer/API gateway,
disable legacy TLS versions (< 1.2), and never terminate TLS onto an
unencrypted internal hop for sensitive data.

## 6. Logging and monitoring: CloudTrail

```bash
# Enable account-wide API activity logging -- who did what, when
aws cloudtrail create-trail --name org-trail --s3-bucket-name my-log-bucket
aws cloudtrail start-logging --name org-trail
```

CloudTrail records every API call made against your account — the cloud
equivalent of the auth logs in Level 2 Module 6, and the primary forensic
source when investigating a cloud breach: it answers "which credential did
this, from where, and what did it touch."

## 7. Automated cloud posture checks

```bash
# ScoutSuite -- free, open-source multi-cloud security auditing tool
pip install scoutsuite
scout aws --profile my-lab-profile
```

ScoutSuite produces an HTML report flagging misconfigurations across IAM,
storage, network, and logging — the cloud equivalent of the Lynis/CIS
Benchmark approach from Level 2 Module 3, applied to cloud resource
configuration instead of OS settings.

## 8. Cloud security checklist

- [ ] MFA enforced on root and all human IAM users
- [ ] No IAM policy grants `"Action": "*", "Resource": "*"` without justification
- [ ] Public access blocked by default on all storage; explicit exceptions reviewed
- [ ] No management port (SSH/RDP/database) open to `0.0.0.0/0`
- [ ] Encryption at rest enabled on every data store
- [ ] CloudTrail (or equivalent) enabled account-wide, logs shipped off-account
- [ ] Access keys rotated regularly; long-lived keys avoided in favor of roles

## Key terms

| Term | Meaning |
|---|---|
| **Shared Responsibility Model** | Division of security duties between cloud provider and customer |
| **IAM** | Identity and Access Management — who/what can do what in a cloud account |
| **Security group** | Cloud-native virtual firewall for an instance/network interface |
| **Bastion host** | A hardened, tightly-controlled single entry point for administrative access |
| **CloudTrail** | AWS's API activity logging service (Azure Activity Log, GCP Audit Logs are equivalents) |
| **Public bucket exposure** | Cloud storage misconfigured to allow unauthenticated public access |

## Exercise

1. Create an AWS free-tier account (or use an existing one) and enable MFA
   on the root account first, before anything else.
2. Create a scoped IAM policy (not admin) for a hypothetical app needing
   only read/write to one S3 bucket, following section 2.
3. Create an S3 bucket, confirm public access is blocked by default, and
   verify with `get-public-access-block`.
4. Enable CloudTrail account-wide and confirm at least one logged event
   from your own actions above.
5. Run ScoutSuite against your account and document the top three findings
   with a proposed fix for each.
