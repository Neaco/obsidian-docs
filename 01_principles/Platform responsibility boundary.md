
This artifact defines **what the platform owns**, **what it exposes**, and **what it explicitly refuses**.  
Anything not listed here is **out of scope by default**.

---

## Platform Owns (Authoritative)

The platform is the **source of truth and enforcement** for:

### 1. Policy enforcement
- Security baselines
- Compliance requirements
- Mandatory quality thresholds
- Mandatory observability signals

These are **not configurable by default**.

### 2. Golden paths to production
- Standardized workflows to build, deploy, and operate services
- Opinionated defaults that encode company standards
- Enforcement of required steps and gates

Deviation requires explicit approval.

### 3. Interfaces and abstractions
- The platform defines and versions:
  - service interfaces
  - configuration contracts
  - escape-hatch mechanisms

Breaking changes are owned by the platform.

### 4. Developer experience (end-to-end)
- Time-to-first-production
- Clarity of failure signals
- Reduction of cognitive load on the happy path

DX is a **platform responsibility**, not a side effect.

---

## Platform Exposes (Self-Service)

The platform provides **controlled self-service** for:

### 1. Service lifecycle actions
- Service creation
- Deployment
- Rollback
- Decommissioning

All actions go through platform-defined paths.

### 2. Bounded configuration
- Runtime configuration
- Resource sizing within limits
- Deployment parameters allowed by policy

Freedom exists **only within predefined envelopes**.


### 3. Visibility
- Access to logs, metrics, alerts
- Clear quality and compliance status
- Cost attribution signals

Visibility is mandatory, not optional.

---
## Platform Explicitly Refuses

The platform **does not**:

### 1. Support arbitrary architectures
- No bespoke infrastructure per team
- No custom deployment pipelines outside the golden path

### 2. Act as an application operations team
- No debugging of application logic
- No on-call for application-level failures

Ownership remains with product teams.

### 3. Guarantee feature parity for edge cases
- Non-standard needs do not redefine defaults
- Escape hatches do not become first-class paths

### 4. Hide trade-offs
- Costs are visible
- Constraints are explicit
- “Magic” abstractions are avoided

---
## Boundary Rule (Non-Negotiable)

If a responsibility:
- requires deep infrastructure knowledge, or
- introduces silent policy deviation

then it **must be owned by the platform or not exist at all**.
