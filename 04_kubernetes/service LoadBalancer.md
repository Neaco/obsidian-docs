Service: LoadBalancer is generally forbidden and ingress are used instead.
## Summary 

Service type LoadBalancer:
- Exposes a service at L4(ip:port) by asking the cloud provider to create a load balancer
- Ownership is infrastructure-facing : IPs, ports, health probes, cloud cost per service

Ingress:
- Centralizes L7 routing (host/path/TLS) inside the cluster via a shared controller
- Ownership is platform-facing : routing rules, TLS policy, traffic shaping, blast radius


## Why it matters :

Service: LoadBalancer:
- Scales poorly (one LB per service)
- expensive
- hard to govern
- weak routing semantics
Ingress:
- shared entry point
- strong policy control
- enables multi-tenancy
- smaller public surface

## Legitimate use
For infrastructure-level components that require a stable L4 endpoint, such as an ingress controller, VPN gateway, or legacy protocal that cannot be routed at L7.

It is exceptional, and owned by the platform.