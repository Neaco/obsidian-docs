# Prompt — Project Recap and Alignment

You are joining an ongoing platform engineering project mid-stream.
Your first task is to **reconstruct the current state accurately** and confirm alignment before giving advice.

Below is the authoritative recap.

## Current phase
Phase: \<N> — \<Phase name>

## Decisions already taken (do not challenge unless inconsistent)
- Cloud provider: Azure (AKS)
- Edge / DNS / TLS: Cloudflare + DNS-01
- IaC: Terraform modules with versioning
- Environments: Namespaces (single cluster)
- GitOps tool: <Argo CD / Flux>
- Ingress controller: <NGINX / other>

## What is already implemented
- \<bullet list of completed items>
- <repos created, clusters up, tooling installed>

## What is documented
- ADRs written:
  - ADR-XXX: \<title>
  - ADR-YYY: \<title>
- Playbooks:
  - \<name>
- Interview notes:
  - \<name>

## Constraints
- Budget cap: \<X> CHF / month
- Time horizon: \<weeks>
- Non-goals:
  - \<explicit non-goals>

## What I want from you now
Before proposing anything:
- Restate the current state in your own words.
- Identify the next single most important step.
- Point out any inconsistency or missing decision.
- If assumptions are missing, ask explicitly

Do not proceed until alignment is confirmed.