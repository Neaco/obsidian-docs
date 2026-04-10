
Must trace:
Client → DNS → Cloudflare → AKS LB → Ingress → Service → Pod

Include:

Where TLS terminates

Where IPs change

Where policies apply


1. **Client Device Boundary**
- Routing : L3+L4 (socket creation)
- Responsibility: OS/app, local DNS cache, local TLS store
- Typical Failures: local DNS cache, corp proxy, captive portal
- Source IP: client pub IP
- Destination IP : Cloudflare IP ( or origin if CF dns-only)

1. **Public Internet boundary**
- Routing: L3
- Responbility: ISP routing, BGP path, general connectivity
- Typical failures : packet loss, routing issues, MTU issues
- Source IP : as above
- Destination IP : as above

1. **Cloudflare Edge Boundary**
- Routing: L3+L4 & L7 (hostname, URL, WAF rules)
- Responsibility: DNS hosting (if used), CDN/Proxying, WAF/rate limits, TLS policy client<->cf / sometimes TLS cf<>origin
- Typical failures: wrong DNS record, WAF block, rate limiting, TLS mode mismatch, origin unreachable from CF
- Source IP: cf egress IP
- Dest IP : Azure p IP (LB frontend)

1. **Azure public entry boundary** (Public IP + Az LB dataplane)
- Routing: L4 & health probes
- Responsibility public-facing L4 entry-point for the cluster (or ingress service) and its health enforcement
- Typical failures : wrong IP/port, failed health probes, SNAT/port exhaustion, mis-synced rule/service
- Source IP: cf egress IP
- Dest IP : Az p IP
- notes : only forwards to healthy backends
- Symptom class : timeout / connection reset, sometimes TLS not completing

1. **Customer network boundary** (VNET / subnet, NSGs, UDRs)
- Routing: L3 routing tables, NSG allow/deny rules (5-tuple)
- Responsibility: network controls : subnets, NSGs, UDRs, routing, peering, firewall
- Typical failures : NSG denies, UDR routes to nowhere/NVA, asymmetric routing
- Source IP: cf egress IP
- Dest IP : node private IP
- Symptom class : looks like timeout, but LB shows healthy

1. **Cluster ingress boundary**
- Routing: L7 (hostname, path, headers)
- Responsibility: Host/path routing, TLS termination(if in cluster), redirects, request size / timeouts
- Typical failures : wrong ingress rules, wrong service target, bad TLS secret, app returning 403/503 from ingress
- Source IP : node IP
- Dest IP : Service ClusterIP
- Symptom class : HTTP status codes(404/503/502), not raw TCP timeouts

1. **Kubernetes service discovery** (services => endpoints)
- Routing: L4 (clusterIP + Port) ; endpoint list maintained by CP
- Responsbility: mapping service name/port to actual backends, kube-proxy/IPVS rules, endpoint readiness gating
- Typical failures : selectors don't match, endpoints empty, readiness failing, wrong targetPort
- Source IP: node IP or pod IP
- Dest IP : pod IP
- Symptom class : 503 from ingress, connection errors *inside* cluster


1. **Pod network boundary** (CNI/pod IP reachability)
- Routing: Pod [[CIDR]], networkPolicy rules (L3/L4)
- Responsibility: pot-to-pod routing, network policy enforcement, SNAT rules, node/pod CIDRs
- Typical failures: NetPol blocks, CNI misconfig, IP exhaustion, node route issues
- Source IP: pod IP (caller)
- Dest IP : pod IP (callee)
- Symptom class : timeouts between services, intermittent connectivity

1. **Workload boundary** (pod -> container process)
- Routing: local port binding
- Responsibility: application process, prots
- Typical failures: app not listening, crash, CPU throttling, OOM
- Source IP : pod IP
- Dest IP : pod IP
- Symptom Class : 5xx, slow responses, connection refused (if nothing listening)




