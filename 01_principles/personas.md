# Phase 0 — Artifact 2: Internal User Personas

These personas define **who the platform is built for** and **who it is not optimized for**.  
They are authoritative inputs for all future design decisions.

---

## Persona A — Product Developer (Primary)

### Profile
- Owns one or more business services
- Delivers features under time pressure
- Not an infrastructure specialist

### Goals
- Ship changes to production quickly and safely
- Avoid security or compliance surprises late in the process
- Minimize context switching

### Must *not* think about
- Infrastructure topology
- Security implementation details
- Observability plumbing
- Policy interpretation

### Allowed to customize
- Service-level configuration (environment variables, feature flags)
- Resource sizing within predefined bounds
- Deployment cadence within the golden path

### Explicit constraints
- Must use the golden path to deploy
- Cannot disable mandatory quality, security, or observability gates

---

## Persona B — Power / Edge Team (Secondary)

### Profile
- Builds non-standard workloads (batch, data, legacy, regulated)
- Has higher infrastructure maturity than the median team

### Goals
- Meet specialized requirements without breaking organizational policies
- Retain limited architectural freedom where explicitly justified

### Must *not* think about
- Company-wide security baselines
- Core compliance controls
- Platform internals

### Allowed to customize
- Extended configurations via approved escape hatches
- Non-default runtimes or deployment modes, when justified

### Explicit constraints
- Opt-outs are explicit, documented, and visible
- No silent deviations from platform-enforced policies

---

## Boundary Signal (Intentional)

- The platform optimizes for **Persona A**
- Persona B exists to **define limits**, not to drive defaults

This distinction is critical to preserve a single, opinionated golden path.