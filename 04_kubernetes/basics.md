# Namespaces 
isolate API objects and access, not runtime behavior.

Isolate well:
- Object names & scoping
   - Pods, Services, ConfigMaps, Secrets, Deployments
   - Same name can exist in different namespaces without conflict
- RBAC boundaries
   - You can grant access per namespace
   - Teams can be isolated at the API level


Does not isolate(by default):
- Networking
   - Pods in different namespaces can talk freely unless NetPol are added
- Node-level resources
   - CPU, memory, disk, kernel are shared
   - One noisy namespace can affect others without quota / limits


## StatefulSets
To deploy a stateful service. Each pod will have its own mounted volume.
**Needs** a headless service (`spec.clusterIp: None`)