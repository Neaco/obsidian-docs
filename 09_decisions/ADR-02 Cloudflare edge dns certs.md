# ADR-002: Cloudflare as Internet Edge + Automated DNS + DNS-01 Certificates

## Status
Applied

## Context
- We run a single AKS cluster (Phase 1 scope).
- We need a stable internet edge for application traffic.
- We want automated, reproducible DNS and certificate management.
- We want to minimize manual steps and reduce drift.

## Decision
- Use Cloudflare in proxied mode as the single internet-facing edge for application traffic
- Enforce L7 protections (WAF, rate limiting) at the cloudflare Edge
- Automate public DNS records using ExternalDNS with cloudflare as the authoritative DNS provider
- Issue and renew TLS certificates using cert-manager with DNS-01 challenges via CLoudflare
- Retrict hostname patterns to platform-approved formats to prevent route hijacking

## Options considered
1) Cloudflare proxied + ExternalDNS + cert-manager DNS-01
2) DNS-only + manual DNS + manual certificates
3) HTTP-01 validation via Ingress

## Decision drivers
- Reliability of certificate issuance
- Drift prevention and reproducibility
- Clear ownership boundaries (platform vs app teams)
- Reduced public exposure and consistent TLS policy

## Consequences
### Positive
- Centralized L7 security and policy at the edge, independant of cluster lifecycle
- Fully declarative DNS and certificate management, eliminating manual drift
- Reliable certificate issuance that does not depend on ingress availability or HTTP routing
- Reduced public attack surface by hiding origin IPs behind Cloudflare

### Negative / trade-offs
- Additional operational complexity in managing two TLS hops (client> Cloudflare, Cloudflare>origin)
- Platform must manage and secure Cloudflare API credentials for ExternalDNS and cert-manager
- Increased dependency on CLoudflare availability for application traffic

## Ownership / Responsibilities
- Cloudflare zone and edge policy: Platform team
- DNS record reconciliation: Platform team (ExternalDNS)
- Certificate issuance and renewal: Platform team (cert-manager)
- Allowed hostnames pattern for teams: Platform team enforced guardrail

## Related ADRs
- ADR-00Y: Default deny-ingress NetworkPolicy (egress allowed initially)