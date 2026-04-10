#phase-1
# Project Status — Phase 1

This document is the authoritative snapshot of the project.
If something is not written here (or in ADRs), it is not considered decided.

---

## Current phase
Phase: 1 — Platform Core

Objective:
Build a credible internal platform baseline suitable for Swiss enterprise platform-engineer interviews.

---

## Fixed constraints (non-negotiable)
- Budget cap: \<90 CHF / month
- Scope: portfolio-grade realism, not production HA
- Cluster model: single AKS cluster
- Environments: namespaces (not multiple clusters)
- One active phase only

---

## Target role
Platform Engineer (internal developer platform focus)

---

## Technology stack (locked unless an ADR says otherwise)

### Cloud & infrastructure
- Cloud provider: Azure
- Kubernetes: AKS
- IaC: Terraform
  - Modules treated as products
  - Versioned, validated, documented

### Delivery model
- GitOps: Argo CD (app-of-apps pattern)
- Cluster state derived from Git only

### Edge / DNS / TLS
- Edge provider: Cloudflare
- DNS management: ExternalDNS (Cloudflare provider)
- TLS: cert-manager with DNS-01 (Cloudflare)
- Ingress: NGINX Ingress Controller
- Traffic model: Internet → Cloudflare → AKS ingress

### Security baseline (Phase 1 level)
- Namespace isolation
- RBAC per team
- NetworkPolicies (default deny + explicit allow)
- Least-privilege Cloudflare API tokens

---

## Phase 1 scope

### In scope
- Terraform modules for:
  - network
  - AKS cluster
  - supporting infra
- Kubernetes multi-tenancy
- GitOps bootstrap of platform services
- Automated DNS and certificates via Cloudflare
- Clear ADRs for all major decisions
- Interview-ready explanations

### Out of scope
- Multi-cluster setups
- Service mesh
- Backstage or full IDP UI
- Managed observability platforms
- Compliance certifications
- Production-grade HA

---

## Repository layout (intended)

platform-engineering/
├── terraform-modules/
├── platform-cluster/
├── gitops/
├── cloudflare/
├── docs/
└── adr/


Each repository must include:
- a clear README
- problem → solution → trade-offs
- reproducibility instructions

---

## Decisions taken (summary)
- Single AKS cluster with namespace-based environments
- Cloudflare used as mandatory edge
- DNS-01 preferred over HTTP-01 for TLS
- GitOps as the only deployment mechanism
- Terraform responsible only for infrastructure, not app state

Authoritative details live in ADRs.

---

## ADRs (to be created / tracked)
- ADR-001: Terraform module structure
- ADR-002: AKS ingress design
- ADR-003: Namespace and tenancy boundaries
- ADR-004: Cloudflare vs direct ingress exposure
- ADR-005: GitOps repository structure

Status: drafts or pending unless marked otherwise.

---

## What is implemented
- Nothing yet (Phase 1 execution not started)

This is intentional.
Implementation follows documentation and decisions.

---

## What is documented
- Learning rules
- Phase reference
- LLM interaction rules
- Vault structure and note templates

---

## Current focus
- Finalize Phase 1 ADRs
- Design Terraform module boundaries
- Design AKS networking and ingress model

No implementation until these are stable.

---

## Next single concrete step
Draft **ADR-001: Terraform module structure**.

Success criteria:
- Clear module boundaries
- Justified interfaces
- Upgrade and versioning strategy explained

---

## Alignment check
If any advice or implementation suggestion:
- contradicts this document
- ignores constraints
- assumes future phases

It must be rejected or corrected before proceeding.