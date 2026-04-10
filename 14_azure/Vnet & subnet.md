
**A VNet defines who can talk to whom by default; subnets define where policies and services are applied.**

A virtual network is:
- a private IP address space
- a routing boundary where resources can talk by default
We *apply* traffic management (filtering, routing rules) *onto it* via other constructs.


a subnet defines:
- a smaller IP range
- a policy and attachment boundary
- where ytou attach rules (NSGs, routes)
- and where managed services are placed


Attaching a managed service to a subnet means: 
- the service consumes IPs from that subnet
- the traffic to and from the service follows the subnet's routing and security rules (NSGs, UDRs)
Important, because:
- subnet policies become the only lever to control traffic to managed services
- it defines blast radius, egress paths, who can reach what
- mistakes here affect all workloads using that service