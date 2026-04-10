## Context

When building a kubernetes platform, a decision had to be made on the default networkPolicies security rules, between ingress only and ingress + egress.

## Decision

Only ingress will be configured.
Egress is required for many legitimate dependencies, and enforcing a deny-all egress creates high operational friction without proportional risk reduction.

## Consequences

Low risk of data exfiltration.
Lower design overhead