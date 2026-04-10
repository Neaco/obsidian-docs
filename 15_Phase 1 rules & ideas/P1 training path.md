# Phase 1 — Training path (6–8 weeks)

**Goal**  
Become credible on Terraform-as-product, Kubernetes multi-tenancy, and GitOps — with concrete outputs you can keep for your portfolio.

---

## Week 1 — Platform foundations + Linux/networking refresh  
_(Fast but non-optional)_

### You train
- **Networking you’ll actually be grilled on**
  - CIDR / subnets
  - NAT
  - DNS
  - L4 vs L7
  - TLS basics
  - Ingress vs gateway
- **Linux ops basics**
  - systemd
  - journald
  - process & memory inspection
  - sockets
  - permissions
  - troubleshooting mindset

### You produce
- **Platform assumptions doc**
  - environments (dev / stg / prd)
  - tenancy model
  - naming conventions
  - repo boundaries
- **Debug playbook** (1–2 pages)
  - how you troubleshoot a broken deployment

### Passing bar
- You can explain a full request path:
  - client → DNS → LB → ingress/gateway → service → pod  
  - and clearly state where and why it can break.

---

## Week 2 — Terraform fundamentals that matter for platform work

### You train
- **Module design**
  - interface design
  - inputs / outputs
  - composition
  - dependency boundaries
- **State & lifecycle**
  - remote state
  - locking
  - drift
  - import strategy
  - taint vs recreate tradeoffs
- **Testing**
  - `terraform test` (minimum)
  - or lightweight integration testing
  - linting and formatting

### You produce
- Repo: `terraform-modules`
  - `modules/`
  - `examples/`
  - strict variable validation
  - meaningful outputs
  - versioning + changelog

### Passing bar
- Your modules look like something another team can use  
  **without asking you questions every day**.

---

## Week 3 — Terraform “platform-grade” patterns

### You train
- **Environment promotion**
  - dev / stg / prd
  - minimal duplication
- **Composition patterns**
  - root modules
  - module stacks
  - dependency injection via outputs
- **Policy & guardrails**
  - required tags
  - naming rules
  - allowed CIDRs
  - basic policy enforcement at the Terraform layer

### You produce
- A **root stack** that creates:
  - VPC / VNet + subnets
  - managed Kubernetes cluster skeleton (EKS / AKS / GKE)
  - one managed database (optional but recommended)
- An **upgrade story**
  - how module versions are bumped safely
  - how changes propagate

### Passing bar
- You can explain:
  - how you prevent breaking changes
  - how upgrades roll across environments.

---

## Week 4 — Kubernetes multi-tenancy essentials

### You train
- **Namespaces as tenancy**
  - quotas
  - limits
  - basic priority classes
- **RBAC**
  - roles
  - rolebindings
  - service accounts
  - least-privilege design
- **Isolation**
  - NetworkPolicies
  - Pod Security (or equivalent)
  - admission basics

### You produce
- Repo: `platform-cluster`
  - namespace templates
  - RBAC per team
  - quotas + limits
  - baseline network policy:
    - deny-by-default
    - explicit allows
- **Tenant onboarding doc**
  - how a new team is onboarded in ~10 minutes

### Passing bar
- You can demonstrate that:
  - Team A cannot break Team B
  - across compute, network, and permissions.

---

## Week 5 — GitOps foundations  
_(Pick Argo CD or Flux — commit to one)_

### You train
- Desired state & reconciliation
- App-of-apps / Kustomize layering
- Secrets strategy (conceptual is enough)
- Repo separation:
  - cluster config
  - platform services
  - applications

### You produce
- Repo: `gitops`
  - `clusters/dev`
  - `clusters/stg`
  - `clusters/prd`
  - `platform/`
    - ingress
    - cert-manager
    - external-dns
    - basic monitoring
  - one sample app promoted via Git

### Passing bar
- You can:
  - recreate the cluster from Git
  - show that a PR directly changes what runs.

---

## Week 6 — Glue it all together: one coherent platform slice

### You train
- **Thin platform thinking**
  - smallest useful set of platform capabilities
- **Release discipline**
  - change review
  - rollback strategy
  - drift detection

### You produce
- **End-to-end demo**
  - Terraform provisions infra (network + cluster)
  - GitOps bootstraps platform services
  - a team namespace appears with guardrails
  - a sample service deploys with a standard ingress

### Passing bar
- You can screen-record a 5–10 minute walkthrough  
  and it **does not look like a toy**.


---

## Non-negotiables (what makes this platform-grade)

- Everything reproducible from scratch.
- Clear repo boundaries and ownership.
- Docs written for a **developer user**, not for you.
- Every default has an explicit reason.