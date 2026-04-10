#phase-1
# Phase 1 — Concepts and Documentation Map

This document defines:
- the concepts that must be understood for Phase 1
- the documentation that is worth reading
- the expected depth of understanding

If a concept is not listed here, it is optional.
If documentation is not listed here, it is not required.

---

## How to use this document

For each concept:
1. Read the listed documentation (primary sources only).
2. Write at least one concept note in your vault.
3. Be able to explain the concept in \<60 seconds.
4. Link the concept to a concrete platform decision.

Reading without producing notes does not count.

---

## 1. Platform engineering fundamentals

### Concepts to understand
- What an internal developer platform is
- Platform vs DevOps vs SRE responsibilities
- Paved roads vs self-service vs autonomy
- Platform as a product (users = engineers)

### Required documentation
- CNCF — Platform Engineering whitepapers / blog posts
- Internal Developer Platform (IDP) concept articles (CNCF / Thoughtworks)
- “Platform teams” sections from Team Topologies

### Expected depth
- You can explain why platforms exist
- You can explain why they fail
- You can justify limiting developer freedom

---

## 2. Terraform as a platform product

### Concepts to understand
- Module boundaries and ownership
- Stable interfaces vs internal implementation
- Versioning and backward compatibility
- State management and drift
- Environment promotion patterns

### Required documentation
- Terraform official documentation:
  - Modules
  - State
  - Backends
  - Version constraints
- HashiCorp best practices for module design

### Expected depth
- You can design modules other teams consume
- You can explain breaking vs non-breaking changes
- You can explain how upgrades are rolled out safely

---

## 3. Azure & AKS fundamentals (platform view)

### Concepts to understand
- Azure networking primitives (VNet, subnet, NSG)
- AKS control plane vs node pools
- Managed identities and access patterns
- Load balancer behavior in AKS
- Cost drivers in AKS

### Required documentation
- Azure Architecture Center — AKS baseline architecture
- AKS official documentation:
  - Networking
  - Identity
  - Node pools
  - Pricing

### Expected depth
- You can explain where traffic flows
- You know which Azure components you actually pay for
- You can explain why a design is cost-aware

---

## 4. Kubernetes multi-tenancy

### Concepts to understand
- Namespaces as tenancy boundaries
- RBAC design for teams
- Resource quotas and limits
- NetworkPolicies and isolation
- Blast radius reduction

### Required documentation
- Kubernetes official docs:
  - Namespaces
  - RBAC
  - ResourceQuota / LimitRange
  - NetworkPolicy
- CNCF / Kubernetes multi-tenancy guides

### Expected depth
- You can explain how teams are isolated
- You can explain what is *not* isolated
- You can explain failure scenarios

---

## 5. Ingress, traffic, and edge architecture

### Concepts to understand
- L4 vs L7 load balancing
- Ingress controller responsibilities
- TLS termination patterns
- Source IP handling behind proxies
- Edge vs cluster responsibilities

### Required documentation
- NGINX Ingress Controller documentation
- AKS ingress and load balancer documentation
- General HTTP/TLS fundamentals (no vendor fluff)

### Expected depth
- You can trace a request end-to-end
- You know where TLS is terminated and why
- You can explain trade-offs clearly

---

## 6. Cloudflare integration (platform edge)

### Concepts to understand
- DNS vs proxying in Cloudflare
- DNS-01 vs HTTP-01 challenges
- API tokens and least privilege
- WAF and rate limiting (conceptual only)

### Required documentation
- Cloudflare official docs:
  - DNS
  - API tokens
  - DNS-01 challenges
- ExternalDNS Cloudflare provider documentation
- cert-manager Cloudflare DNS-01 documentation

### Expected depth
- You can explain why DNS-01 is preferred
- You can explain how DNS is automated safely
- You can explain how Cloudflare fits into platform boundaries

---

## 7. GitOps fundamentals (platform usage)

### Concepts to understand
- Desired state vs imperative actions
- Reconciliation loops
- App-of-apps pattern
- Separation of concerns (infra vs platform vs apps)
- Rollback and drift correction

### Required documentation
- Argo CD official documentation:
  - Core concepts
  - App-of-apps
  - Kustomize support
- GitOps principles (Weaveworks)

### Expected depth
- You can explain how a change reaches the cluster
- You can explain rollback behavior
- You can explain why Git is the source of truth

---

## 8. Security baseline (Phase 1 level)

### Concepts to understand
- Least privilege
- Shared responsibility model
- Secret handling (conceptual)
- Guardrails vs blocking controls

### Required documentation
- Kubernetes RBAC docs
- Cloudflare API token docs
- General cloud shared responsibility models

### Expected depth
- You can justify what is enforced vs trusted
- You can explain security boundaries without buzzwords

---

## What you must explicitly ignore in Phase 1

Do NOT deep-dive into:
- Service meshes
- Backstage
- Zero trust architectures
- Managed observability platforms
- Compliance frameworks

These belong to later phases.

---

## Validation checklist

You are ready to exit Phase 1 knowledge-wise if:
- You can explain every concept listed above in your own words
- You can link each concept to a platform decision or ADR
- You are not relying on memorized docs, but on mental models
