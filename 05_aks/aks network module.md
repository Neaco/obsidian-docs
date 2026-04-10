
Responsibilities inside the module
- Create a VNet with a defined address space
- Create one or more subnets intended for AKS node pools

The module :
- owns the VNet and the AKS node subnet(s)
- Has no environment policy
- always guarantees that an explicit subnet exists for AKS node pools
- always guarantees clear ownership of the VNet and subnets it creates or consumes