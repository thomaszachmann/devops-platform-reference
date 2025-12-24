# Harbor – Security Model

This document describes the **security model** for Harbor as a platform
container registry.  
The focus is on **supply-chain security, access control, and policy enforcement**
in a Kubernetes-based platform.

Harbor is treated as a **security control point**, not just a storage service.

---

## Security Objectives

The primary security goals for Harbor are:

- Prevent vulnerable or untrusted images from reaching production
- Enforce least-privilege access to container images
- Provide visibility into image provenance and vulnerabilities
- Support auditability and compliance requirements

---

## Threat Model Overview

Key threat categories addressed:

| Threat | Description |
|------|-------------|
| Compromised CI pipeline | Malicious images pushed to registry |
| Vulnerable base images | Known CVEs propagated to workloads |
| Unauthorized access | Image exfiltration or tampering |
| Supply-chain attacks | Injection of untrusted artifacts |
| Credential leakage | Registry credentials abused |

Harbor is positioned to **break the attack chain early**, before workloads are deployed.

---

## Access Control & Identity

### Authentication
- Local users for platform administrators
- Optional external identity provider via OIDC
- CI/CD systems authenticate using robot accounts

### Authorization
- Projects used to separate teams and environments
- Role-based access control applied per project
- No shared credentials across environments

Design principle:
> **Human users and automation are strictly separated.**

---

## Robot Accounts

Robot accounts are used exclusively by CI/CD pipelines.

Security characteristics:
- Scoped to a single project
- Minimal required permissions
- Revocable without impacting human users
- Credentials rotated periodically

This reduces blast radius in case of credential compromise.

---

## Image Vulnerability Scanning

### Scanning Strategy
- Automatic scanning on image push
- Scans executed before image promotion
- Scan results stored centrally

### Policy Enforcement
- Severity thresholds defined per environment
- Critical vulnerabilities can block promotion
- Exceptions require explicit approval

This ensures that **security feedback is immediate and enforceable**.

---

## Image Trust & Signing

Harbor supports image signing to establish trust.

Security benefits:
- Ensures image integrity
- Prevents unauthorized image modification
- Enables admission control policies in Kubernetes

Signing is optional and can be introduced gradually to balance security and complexity.

---

## Registry-to-Cluster Security

### Image Pull Policies
- Kubernetes pulls images only from approved registries
- Credentials scoped per namespace or workload
- ImagePullSecrets managed centrally

### Admission Controls (Optional)
- Enforcement of signed images
- Blocking of images with critical vulnerabilities
- Validation of image provenance

This shifts security enforcement **left**, before runtime.

---

## Secrets Management

Security principles:
- No secrets stored in Git
- No long-lived static credentials
- CI secrets injected at runtime
- Registry credentials scoped and rotated

Harbor does not act as a secret store and integrates with the platform’s
central secrets management strategy.

---

## Network Security

### Network Isolation
- Harbor deployed in a dedicated namespace
- Network policies restrict ingress and egress
- Only required services can access the registry

### Transport Security
- TLS enforced for all registry communication
- Strong cipher suites
- No plaintext access permitted

---

## Auditing & Traceability

Harbor provides:
- Audit logs for authentication and image actions
- Visibility into image lifecycle events
- Traceability from image build to deployment

Audit data can be integrated into centralized logging systems for long-term retention.

---

## Incident Response Considerations

In case of a security incident:
- Compromised robot accounts can be revoked immediately
- Vulnerable images can be quarantined
- Image promotion pipelines can be paused
- Audit logs support root cause analysis

Clear separation of duties enables **fast and controlled response**.

---

## Security Trade-offs

| Decision | Rationale |
|--------|-----------|
| Robot accounts | Reduced blast radius |
| Automatic scanning | Early vulnerability detection |
| Optional signing | Gradual security adoption |
| Central registry | Consistent policy enforcement |

---

## When Harbor Is a Security Requirement

Harbor becomes essential when:
- Multiple teams share a Kubernetes platform
- Compliance or audit requirements exist
- Supply-chain security is a concern
- Production environments must be protected from untrusted images

---

## Summary

Harbor is positioned as a **security enforcement layer** in the container
supply chain.

It provides:
- Early vulnerability detection
- Strong access control
- Image trust mechanisms
- Clear auditability

This approach aligns with **real-world enterprise security expectations** and
supports scalable Kubernetes platforms.

