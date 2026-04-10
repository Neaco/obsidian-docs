Changing an output is more dangerous than changing an input.
Inputs are chosen by consumers, outputs are depended on by consumers.

#### Outputs:
1. **Design outputs as stable contracts**
	Once an output exists, never remove it or change its type ; deprecate and replace instead
2. **Prefer extensible output shapes**
	Use objects so new fields can be added without breaking existing consumers


#### Environment
Environment-specific decisions belong in the root stack, because reusable modules should encode capabilities, not policy.
Making modules environment-aware couples them to deployment context, reduces reuse, and increases the risk of hidden behavior differences between environments.