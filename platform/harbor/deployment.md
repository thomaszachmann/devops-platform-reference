# Harbor – Deployment

This document describes the **deployment strategy** for Harbor as a
platform service on Kubernetes.  
The focus is on **repeatability, security, and operational clarity** rather
than a copy-paste installation guide.

---

## Deployment Scope

Harbor is deployed as a **shared platform component** and provides:
- Central container registry
- Vulnerability scanning
- Image governance and policy enforcement

Harbor is **not** treated as an application workload but as **stateful infrastructure**.

---

## Deployment Model

- Deployment method: **Helm**
- Target platform: Kubernetes (AWS EKS)
- Namespace: `harbor`
- Exposure: Ingress with TLS
- Storage: Persistent Volumes
- Authentication: Local users + optional external IdP

---

## Prerequisites

Before deploying Harbor, the following requirements must be met:

### Kubernetes
- Kubernetes version compatible with the selected Harbor release
- Cluster supports persistent volumes
- Ingress controller available (e.g. NGINX)

### Platform Services
- DNS record for Harbor endpoint
- TLS certificates (managed externally or via cert-manager)
- Storage class suitable for registry workloads

### Access
- Cluster-admin permissions for initial installation
- CI/CD credentials prepared separately

---

## Namespace Layout

Harbor is deployed into a **dedicated namespace**:


namespace: harbor


Rationale:
- Clear separation from application workloads
- Independent lifecycle and access control
- Simplified monitoring and backup configuration

---

## Helm-Based Deployment

Harbor is installed using the official Helm chart.

Design principles:
- Values files are environment-specific
- Secrets are not committed to Git
- Defaults are reviewed and hardened

Example structure:


```
platform/harbor/
├── values/
│ ├── dev.yaml
│ └── prod.yaml
└── deployment.md
```


> The actual Helm values depend on the environment and are intentionally
> not fully embedded in this repository.

---

## Core Configuration Areas

### Ingress & TLS
- HTTPS-only access
- TLS termination at the ingress
- Strong cipher configuration
- Internal access preferred (no public exposure unless required)

### Storage
- Persistent volumes for:
  - Registry data
  - Database
  - Job service
- Storage class selected based on durability and performance
- Capacity planning is done explicitly

### Resource Management
- CPU and memory limits defined for all components
- Registry and scanner resources are monitored closely
- Horizontal scaling considered for stateless components

---

## Security Hardening

### Network Isolation
- Harbor namespace isolated via network policies
- Only required ingress traffic allowed
- Egress restricted where possible

### Credentials
- Admin password generated securely
- CI access via robot accounts
- No static credentials stored in Git

### Scanning Configuration
- Vulnerability scanning enabled by default
- Severity thresholds configured per environment
- Scan concurrency adjusted to avoid cluster overload

---

## Deployment Workflow

Typical deployment flow:

1. Prepare namespace and prerequisites
2. Review and adjust Helm values
3. Install or upgrade Harbor via Helm
4. Verify:
   - Pod health
   - Ingress accessibility
   - Registry push/pull
   - Scan execution
5. Integrate with CI/CD pipelines

---

## Upgrade Strategy

Upgrades are treated as **planned maintenance events**.

Guidelines:
- Always review Harbor release notes
- Test upgrades in non-production environments
- Backup registry data before upgrading
- Use Helm upgrade with rollback capability

Downtime expectations are documented and communicated in advance.

---

## Backup & Recovery

Harbor requires **regular backups** of:
- Registry storage
- Database
- Configuration

Backup strategy depends on:
- Underlying storage provider
- RPO/RTO requirements
- Compliance constraints

Recovery procedures are documented separately in the operations section.

---

## Observability

Minimum monitoring requirements:
- Pod availability
- Storage utilization
- Scan queue length
- Error rates during push/pull

Metrics are integrated into the platform monitoring stack and alerting
is configured for critical conditions.

---

## Deployment Trade-offs

| Decision | Rationale |
|--------|-----------|
| Helm-based deployment | Repeatable and upgrade-safe |
| Dedicated namespace | Clear ownership and isolation |
| External TLS management | Consistent certificate strategy |
| Stateful treatment | Registry is business-critical |

---

## Summary

Harbor is deployed as a **first-class platform service** with:
- Explicit lifecycle management
- Clear security boundaries
- Operational readiness

The deployment approach mirrors **real-world production platforms** and
avoids hidden complexity or implicit assumptions.




```
