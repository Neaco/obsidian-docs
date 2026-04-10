Root stacks hold environment and deployment-specific decisions: composition (which module), policy choices, integration points, and concrete parameters for a given environment.

Ex: A terraform configuration that instantiates modules to build one environment

Purpose:
- Turn decisions into concrete infrastructure
- Provide environment-specific values
- Own Terraform state for that environment.

A root stack:
- calls reusable modules
- passes real values (CIDRs, IDs, flags)
- wires outputs to inputs
- is what you actually tf apply