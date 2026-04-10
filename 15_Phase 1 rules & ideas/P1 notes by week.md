# Phase 1 — What notes to write (week by week)

---

## Week 1 — Platform fundamentals

### Notes to write
- `01_principles/What is a platform team.md`
- `01_principles/Paved roads vs freedom.md`
- `02_architecture/High-level platform architecture.md`
- `11_interview/Explain platform engineering in 60 seconds.md`

### Passing bar
- You can explain why platform teams exist without saying “DevOps”.

---

## Week 2 — Terraform as a platform product

### Notes to write
- `03_terraform/Terraform module design.md`
- `03_terraform/State management and drift.md`
- `03_terraform/Environment promotion strategy.md`
- `09_decisions/ADR-001-terraform-module-structure.md`
- `11_interview/How my platform provisions infrastructure.md`

### Passing bar
- You can justify module interfaces and versioning decisions.

---

## Week 3 — AKS & networking basics

### Notes to write
- `05_aks/AKS baseline architecture.md`
- `05_aks/AKS networking model.md`
- `09_decisions/ADR-002-public-ingress-design.md`
- `11_interview/How traffic reaches a service in my platform.md`

### Passing bar
- You can trace a request end-to-end without hand-waving.

---

## Week 4 — Kubernetes multi-tenancy

### Notes to write
- `04_kubernetes/Kubernetes multi-tenancy.md`
- `04_kubernetes/RBAC design for teams.md`
- `04_kubernetes/Network policies for isolation.md`
- `09_decisions/ADR-003-namespace-boundaries.md`
- `11_interview/How my platform isolates teams.md`

### Passing bar
- You can explain blast-radius reduction clearly.

---

## Week 5 — Cloudflare integration

### Notes to write
- `06_cloudflare/Cloudflare role in the platform.md`
- `06_cloudflare/ExternalDNS with Cloudflare.md`
- `06_cloudflare/Cert-manager DNS-01 with Cloudflare.md`
- `09_decisions/ADR-004-cloudflare-vs-direct-ingress.md`
- `11_interview/How my platform handles DNS and TLS.md`

### Passing bar
- You can explain why DNS-01 beats HTTP-01 in a platform.

---

## Week 6 — GitOps foundations

### Notes to write
- `07_gitops/Why GitOps for platform teams.md`
- `07_gitops/Argo CD app-of-apps pattern.md`
- `09_decisions/ADR-005-gitops-repo-structure.md`
- `10_playbooks/Cluster bootstrap via GitOps.md`
- `11_interview/How changes reach production.md`

### Passing bar
- You can explain reconciliation and rollback without buzzwords.

---

## Week 7 — Glue & validation

### Notes to write
- `02_architecture/End-to-end platform flow.md`
- `10_playbooks/New team onboarding.md`
- `10_playbooks/New service onboarding.md`
- `11_interview/How I prevent the platform from becoming a bottleneck.md`

### Passing bar
- You sound like an owner, not a tool operator.