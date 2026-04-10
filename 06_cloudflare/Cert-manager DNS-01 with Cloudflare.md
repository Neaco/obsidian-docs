### What is DNS-01 ?
The CA checks the DNS TXT Record (instead of a HTTP call for HTTP-01)


### Why DNS-01 ?

- Cloudflare my block or proxy HTTP-01 requests
- TLS termination may already happen at Cloudflare
- Ingress may not exist yet
- HTTP routing may be changing during deploys

![[P1_ Why prefer DNS-01 over HTTP-01 for certificate validation ?]]


## Setup

cert-manager owns the ACME flow. It has Cloudflare API access to create DNS TXT records.
Applications never touch DNS or certificates directly.


ExternalDNS : permissions to create/update/delete DNS records it owns.
cert-manager(DNS-01): permissions to create/update/delete TXT records under  _\_acme\_challenge.*_




## Lifecycle

Platform team owns the certificate lifecycle process:
- cert-manager installation and configuration
- issuer configuration (ACME account, DNS-01 solver via Cloudflare)
- renewal policies and alerting
- where certs are stored (K8S secrets) and who can read them (RBAC)
Application teams own usage within guardrails: 
- requesting a certificate for their hostname (via an approved Certificate / Ingress pattern)
- attaching it to their ingress (or letting ingress-shim do it)