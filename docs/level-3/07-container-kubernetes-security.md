# 07 · Container & Kubernetes Security

Containers and Kubernetes introduce a new set of layers to secure — the
image, the container runtime, and the orchestrator itself — on top of
everything Level 2-3 already covered for hosts and networks.

## 1. The layers of container security

```
1. Image        -- what's baked into the container
2. Registry     -- where images are stored and pulled from
3. Runtime      -- the container engine executing the image
4. Orchestrator -- Kubernetes control plane and scheduling
5. Cluster network / host OS -- everything underneath
```

A vulnerability at any layer compromises everything above it — a
misconfigured host kernel undermines container isolation regardless of
how well the image itself was built.

## 2. Image security

```dockerfile
# Bad: runs as root, uses a large attack-surface base image
FROM ubuntu:latest
COPY app /app
CMD ["/app"]

# Better: minimal base image, non-root user, pinned version
FROM gcr.io/distroless/static-debian12:nonroot
COPY --chown=nonroot:nonroot app /app
USER nonroot
CMD ["/app"]
```

```bash
# Scan images for known vulnerabilities before they ever reach a registry
trivy image myapp:1.4.2

# Example output:
# myapp:1.4.2 (debian 12)
# Total: 3 (HIGH: 2, CRITICAL: 1)
# libssl3  CVE-2026-XXXX  CRITICAL  fixed in 3.0.15
```

Scan in CI, fail the build on critical CVEs, and re-scan images already
in the registry regularly — a CVE published after your image was built
still needs to be caught.

## 3. Runtime hardening

```yaml
# Kubernetes Pod securityContext -- deny privilege escalation by default
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: myapp:1.4.2
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
```

A container running as root with `allowPrivilegeEscalation: true` and
no dropped capabilities is one kernel-level container-escape CVE away
from full host compromise — hardening the pod spec is cheap insurance
against vulnerabilities not yet discovered.

## 4. Kubernetes RBAC and API server security

```yaml
# Least-privilege Role: read-only access to Pods in one namespace only
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: app-team
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

```bash
# Audit who can do what -- a common finding is over-broad ClusterRoleBindings
kubectl get clusterrolebindings -o json \
  | jq '.items[] | select(.roleRef.name=="cluster-admin") | .subjects'
```

The Kubernetes API server itself must never be exposed to the public
internet without strong authentication (OIDC integration, not static
tokens) — an exposed, poorly-secured API server is equivalent to leaving
the keys to every workload in the cluster in a public place.

## 5. Network policy: default-deny between pods

By default, every pod in a cluster can talk to every other pod — the
Kubernetes equivalent of a flat network with no segmentation:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: app-team
spec:
  podSelector: {}
  policyTypes: ["Ingress", "Egress"]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: app-team
spec:
  podSelector:
    matchLabels: { app: backend }
  ingress:
    - from:
        - podSelector: { matchLabels: { app: frontend } }
      ports: [{ port: 8080 }]
```

## 6. Secrets in Kubernetes

Native Kubernetes Secrets are only base64-encoded, not encrypted, by
default — treat them as sensitive but not sufficient on their own:

```bash
# Enable encryption at rest for Secrets in etcd
# (EncryptionConfiguration applied to the API server)
kubectl get secrets --all-namespaces -o json | jq '.items[].metadata.name'
```

For production, integrate an external secrets manager (HashiCorp Vault,
cloud provider secret store) via a CSI driver or operator rather than
relying on raw Kubernetes Secrets for high-value credentials.

## 7. Admission control and policy as code

```yaml
# OPA Gatekeeper / Kyverno: reject deployments that violate policy
# before they ever reach the cluster
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-privileged-containers
spec:
  validationFailureAction: Enforce
  rules:
    - name: no-privileged
      match:
        resources: { kinds: ["Pod"] }
      validate:
        message: "Privileged containers are not allowed"
        pattern:
          spec:
            containers:
              - =(securityContext):
                  =(privileged): "false"
```

## How It Actually Works: what actually isolates a container, and how Kubernetes network policy is enforced in the kernel

A container is not a lightweight VM — it's a set of ordinary Linux processes
made to *look* isolated using three independent kernel features, and
understanding this is what makes "container escape" concrete rather than
mysterious. **Namespaces** give a process its own view of a global resource:
a PID namespace makes a container's process see itself as PID 1 with no
visibility into host processes; a mount namespace gives it its own
filesystem root; a network namespace gives it its own interfaces and
routing table. **Cgroups** (control groups) limit and account for resource
usage (CPU shares, memory ceilings) — the mechanism behind
`--memory=512m`. Neither of these is a security boundary in the way a
hypervisor is: both features change what a process *can see or is limited to
consuming*, not what it fundamentally has permission to do. That's the third
piece — **capabilities and seccomp**: by default a container process still
runs as a Linux process subject to the same DACL/capability checks from
Level 1 Module 3, and a container escape typically means the containerized
process retained a capability (like `CAP_SYS_ADMIN`, or running genuinely as
UID 0 with no user-namespace remapping) that lets it perform an operation —
mounting the host filesystem, loading a kernel module — that reaches outside
its namespace's intended view and touches the shared, single kernel every
container on that host actually runs on. Runtime hardening (seccomp
profiles, dropping capabilities, `readOnlyRootFilesystem`) works by removing
exactly the syscalls and capabilities a given workload doesn't need, shrinking
this shared-kernel attack surface to only what's actually required.

**Kubernetes NetworkPolicy** resources don't do the actual packet filtering
themselves — they're a declarative spec that the cluster's CNI plugin (Calico,
Cilium) compiles down into the same kernel-level packet filtering primitives
covered earlier: iptables rules or, in eBPF-based CNIs, small verified
programs attached at kernel hook points that run per-packet with near-native
speed. A "default-deny" NetworkPolicy is enforced by the CNI installing a
DROP rule matching all pod-to-pod traffic in that namespace, then adding
specific ALLOW rules only for the explicitly permitted pod-selector pairs
— structurally identical to the iptables ordered-chain evaluation from Level
1 Module 8, just auto-generated and continuously reconciled from the
Kubernetes API object instead of hand-written. RBAC on the API server itself
enforces a third, independent layer above the network: every API request
carries the caller's authenticated identity, and the API server evaluates it
against Role/RoleBinding objects using the identical allow-only,
default-deny evaluation model as cloud IAM (Level 2 Module 8) — meaning a
compromised pod that has no ServiceAccount token bound to any RoleBinding
simply cannot call the API server at all, regardless of network reachability.

## 8. Checklist

- [ ] Images built from minimal base images, scanned in CI, non-root by default
- [ ] Pod securityContext drops all capabilities, denies privilege escalation
- [ ] RBAC follows least privilege; cluster-admin bindings audited and rare
- [ ] API server not exposed publicly without strong auth
- [ ] NetworkPolicies default-deny, explicit allow between services
- [ ] Secrets backed by an external secrets manager for sensitive values
- [ ] Admission control (OPA/Kyverno) enforces policy before deploy

## What's next

Module 8 puts detection and hardening skills like these to the test
through structured red team vs. blue team exercises.
