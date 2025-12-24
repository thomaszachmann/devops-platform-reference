# Harbor – Container Registry & Image Security

## Purpose
Harbor is used as a **central container registry and security control point**
for the Kubernetes platform.  
It complements cloud-native registries by adding **enterprise-grade security,
governance, and compliance features**.

This setup reflects a **realistic production scenario** where security,
traceability, and policy enforcement are first-class concerns.

---

## Why Harbor?

While AWS ECR is sufficient for many use cases, Harbor is introduced here to
demonstrate **platform-level container security and governance**.

Key reasons:
- Centralized image scanning and vulnerability visibility
- Fine-grained RBAC and project separation
- Policy-based image promotion (dev → prod)
- Image signing and trust
- Cloud-agnostic registry strategy

> Design principle: **Security and governance are enforced before workloads reach the cluster.**

---

## Architecture Overview

Harbor is deployed as a **platform service** and integrated with:
- GitLab CI/CD (image push, scan, promotion)
- Kubernetes (image pull, policy enforcement)
- Optional external identity provider (OIDC)

High-level flow:
1. CI pipeline builds container image
2. Image is pushed to Harbor
3. Vulnerability scanning is triggered automatically
4. Policies determine whether the image can be promoted
5. Kubernetes pulls only approved images

---

## Deployment Strategy

Harbor is deployed using **Helm** on Kubernetes.

Deployment characteristics:
- Dedicated namespace (`harbor`)
- TLS-enabled ingress
- Persistent storage for registry data
- Resource limits defined for all components

Key considerations:
- Harbor is treated as **stateful infrastructure**
- Backups and upgrades are planned explicitly
- Registry availability is critical for the platform

> This repository focuses on **architecture and integration**.
> Exact Helm values are environment-specific and intentionally not hardcoded.

---

## Security Model

### Access Control
- Projects are used to separate environments and teams
- RBAC is applied per project
- CI systems use robot accounts with minimal permissions

### Image Scanning
- Automatic vulnerability scanning on image push
- Severity thresholds can block image promotion
- Scan results are visible to both CI and platform teams

### Image Trust & Signing
- Optional image signing is supported
- Kubernetes can be configured to pull only signed images
- This enables supply-chain security controls

### Secrets Management
- No credentials are stored in Git
- CI credentials are injected via secret management
- Kubernetes pulls images using scoped registry credentials

---

## Integration with GitLab CI/CD

Typical CI responsibilities:
- Build container image
- Push image to Harbor
- Fail pipeline on critical vulnerabilities
- Promote images via tagging strategy

Benefits:
- Security feedback is available early
- CI pipelines become enforceable quality gates
- Platform teams retain control over production images

---

## Operational Considerations

### Backup & Recovery
- Registry storage must be backed up regularly
- Backup strategy depends on the underlying storage system
- Disaster recovery procedures are documented separately

### Upgrades
- Harbor upgrades are planned and tested
- Helm-based deployment simplifies rollback
- Compatibility with Kubernetes versions is validated before upgrades

### Monitoring
- Key metrics:
  - Registry availability
  - Storage utilization
  - Scan queue length
- Alerts are integrated into the platform monitoring stack

---

## Cost Considerations

Main cost drivers:
- Persistent storage for images
- Compute resources for scanning
- Network traffic during image pulls

Optimization strategies:
- Image retention policies
- Cleanup of unused tags
- Limiting scan scope where appropriate

---

## Design Trade-offs

| Decision | Reasoning |
|--------|-----------|
| Harbor vs. ECR-only | Security and governance visibility |
| Helm-based deployment | Standardized lifecycle management |
| Central registry | Consistent policies across teams |
| Optional signing | Balance between security and complexity |

---

## When Harbor Is (and Is Not) Recommended

### Recommended when:
- Multiple teams share a platform
- Security and compliance are required
- Image provenance matters
- Cloud-agnostic strategy is desired

### Not required when:
- Small teams with minimal security needs
- Short-lived or experimental environments
- Registry governance is handled elsewhere

---

## Summary

Harbor is positioned as a **platform security component**, not just a registry.

It demonstrates:
- Security-first platform design
- Enterprise-ready container workflows
- Clear separation between infrastructure, CI/CD, and workloads

This mirrors how Harbor is used in **real-world Kubernetes platforms**, not as a demo installation.

