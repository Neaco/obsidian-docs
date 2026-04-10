#phase-0 #phase-1 #phase-2 #phase-3 #phase-4 #phase-5 #phase-6
# Project Phases — Reference

This document defines the full progression of the platform engineering project.

---

## Phase 0 — Foundations & framing
**Objective**
Establish mental models, vocabulary, and constraints.

**Key outputs**
- Platform principles
- High-level architecture
- Cost and scope constraints
- Project operating rules

**Exit criteria**
- You can explain what a platform team does without using buzzwords.

---

## Phase 1 — Platform core
**Objective**
Build a credible platform baseline.

**Scope**
- Terraform as a product (modules + versioning)
- AKS single cluster
- Kubernetes multi-tenancy
- GitOps bootstrap
- Cloudflare DNS + TLS integration

**Key outputs**
- Reproducible AKS cluster
- GitOps-managed platform services
- Automated DNS and certificates
- ADRs for all major decisions
- Interview-ready explanations

**Non-goals**
- Multi-cluster
- Service mesh
- Production-grade HA

**Exit criteria**
- End-to-end platform flow works from Git.
- You can demo it in \<10 minutes.
- You can defend every major choice.

---

## Phase 2 — Internal Developer Platform (IDP)
**Objective**
Make the platform usable by other engineers without hand-holding.

**Scope**
- Golden paths
- Self-service workflows
- Standard CI/CD templates
- Default observability

**Key outputs**
- Service scaffolding
- One-command onboarding
- Opinionated pipelines
- Platform documentation

**Exit criteria**
- A developer can ship a service without talking to you.

---

## Phase 3 — Platform maturity
**Objective**
Add guardrails, not friction.

**Scope**
- Policy enforcement
- Security boundaries
- Cost allocation
- Operational playbooks

**Key outputs**
- Policy-as-code
- Cost visibility per team
- Incident playbooks
- Upgrade strategies

**Exit criteria**
- Platform scales to multiple teams without chaos.

---

## Phase 4 — Specialization & differentiation
**Objective**
Increase seniority signal and salary leverage.

**Possible directions**
- Platform + SRE
- Platform + Security
- Platform + DX

**Key outputs**
- Deep-dive ADRs
- Advanced design trade-offs
- Specialized demos

**Exit criteria**
- You are credible for senior/staff-level discussions.

---

## Phase 5 — Portfolio & interview packaging
**Objective**
Convert work into hiring signal.

**Scope**
- Repo cleanup
- Documentation polishing
- Interview narratives
- Salary positioning

**Key outputs**
- Public-facing repos (curated)
- Strong README stories
- Clear career positioning

**Exit criteria**
- Platform work clearly differentiates you from DevOps profiles.

---

## Rule
Do not partially execute future phases.
Depth beats breadth.