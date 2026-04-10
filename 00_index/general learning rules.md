# General Rules — Learning & Progression


## 1. Learning is output-driven
If no artifact is produced, no learning happened.

Valid outputs:
- ADRs
- Architecture diagrams (ASCII is enough)
- Terraform modules
- Kubernetes manifests
- Playbooks
- Interview-ready explanations

Reading or watching content without output does not count.

---

## 2. Notes precede implementation
You must be able to explain a system before building it.

Rule:
- If you cannot explain it in \<60 seconds, do not implement it yet.

This prevents cargo-cult engineering.

---

## 3. One abstraction level at a time
Never mix:
- concepts
- design decisions
- implementation details

Each belongs in a different note type.

Violation leads to confusion and hallucinated understanding.

---

## 4. Decisions are explicit and written
Every meaningful choice must result in an ADR.

If a decision is not written:
- it is not stable
- it will be contradicted later

ADRs are the authoritative memory, not chat history.

---

## 5. Constraints are non-negotiable
Constraints are part of the problem, not obstacles.

Typical constraints:
- cost ceiling
- time box
- single cluster
- portfolio-grade only

Any solution that ignores constraints is invalid.

---

## 6. Minimal viable realism
Everything built must answer:
"Is this plausible in a real company?"

Not required:
- production HA
- full compliance
- enterprise contracts

Required:
- correct mental models


---

## 7. No passive consumption loops
Allowed:
- read → write → build
- ask → decide → document

Not allowed:
- endless reading
- tool hopping
- premature optimization

If stuck, force a decision and document uncertainty.

---

## 8. One active phase at a time
Only one phase may be active.

If something belongs to another phase:
- park it
- do not partially implement it

Context switching destroys depth.

---

## 9. Interview test rule
Anything learned must pass this test:

"Could I defend this decision to a senior platform engineer in Zurich?"

If not, refine or discard.

---

## 10. Periodic reality checks
At regular intervals, ask:
- What did I actually build?
- What decisions are now locked?
- What is the next concrete step?

If answers are fuzzy, reset.