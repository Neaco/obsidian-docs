
## Mandatory controls

- **RBAC** to control who can see and modify resources per namespace
- **NetworkPolicies** to restrict pod-to-pod traffic across namespaces
- **ResourceQuotas and LimitRanges** to prevent one team from exhausting cluster resources

Later:
- Pod Security / admission
- Node pool separation
- Runtime security


Even with those : [[P1_One realistic scenario where Team A can impact team B]] 


### Minimum Team artefacts

1. Namespace manifest
2. RBAC manifests (defines what the team can do in their namespace)
3. ResourceQuota (prevent cluster exhaustion)
4. LimitRange (prevent noisy-neighbor behavior)
5. Baseline NetworkPolicy (default deny ingress)