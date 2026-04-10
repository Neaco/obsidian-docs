In a cluster where:
- namespaces
- RBAC
- NetworkPolicies
- quotas
are used.


Team A can impact Team B by creating excessive churn (many pods/config updates) that overloads the Kubernetes API server and controllers; since the control plane is shared, reconciliation slows for everyone even if namespaces, RBAC, NetworkPolicies, and quotas are in place.

Also, there mayu be no limitRange on team A's config. At runtime, they may create CPU contention. Team B's pods on the same nodes will become slow.