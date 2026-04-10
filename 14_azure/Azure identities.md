
# User
A human identity.

For:
- logging on the portal
- running az login
- Emergency / break-glass access


# Group
A collection of identities (users, service principals, managed identities)
Has no credentials.

For:
- RBAC at scale
- Policy targeting
- Simplifying permission management


# Service Principal
An application identity. Represents software, not a human.
Credentials : client secret, certificate, or federated identity (OIDC)

For:
- CI/CD
- External systems (Github actions, terraform,..)
- Non-azure workloads needing azure access

NB: must manage credentials rotation


# Managed identity
A special kind of service principal created and managed by Azure (no credentials to manage)
Either:
- system-assigned: tied to one azure resource, lifecycle
- user-assigned : standalone, reusable across resources


For:
- Azure resources calling other Azure resources securely (ex: AKS -> Key vault)
- Can only be used inside Azure

NB: No credentials management