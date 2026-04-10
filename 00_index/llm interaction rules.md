# How to Interact With the LLM to Make Progress

This document defines the correct way to use the LLM during this project.
If these rules are not followed, the LLM output must be considered unreliable.

---

## 1. The LLM is not memory
The LLM does not hold authoritative project state.

Rules:
- Never assume the LLM remembers past decisions.
- Always restate critical context explicitly.
- Treat previous answers as disposable unless written into notes.

Authoritative sources are:
- ADRs
- Project status note
- Architecture documents

---

## 2. The LLM is a multiplier, not a driver
The LLM accelerates thinking but does not set direction.

Rules:
- You define goals and constraints.
- The LLM proposes options or implementations.
- Final decisions are yours and must be written down.

If direction is unclear, stop and clarify before asking further questions.

---

## 3. Always start with constraints
Before any non-trivial question, specify:
- Goal
- Constraints (cost, time, scope)
- Non-goals

Any answer that ignores constraints is invalid.

---

## 4. Ask atomic questions
One question per message.
One expected output type per answer.

Good:
- "Design the Terraform module boundaries for AKS"
- "Review this ADR draft"

Bad:
- "Explain everything about AKS, Terraform, and Cloudflare"

---

## 5. Force explicit assumptions
If assumptions are missing, require them to be stated.

Prompt pattern:
"List your assumptions before answering."

If assumptions are wrong, correct them immediately.

---

## 6. Prefer review over generation
Generation creates hallucination risk.
Review detects it.

Use the LLM to:
- critique designs
- find inconsistencies
- stress-test decisions

Not to:
- invent architecture without constraints
- validate unclear thinking

---

## 7. Demand trade-offs
Every recommendation must include:
- why this option was chosen
- what was rejected
- downsides

If no downside is mentioned, the answer is incomplete.

---

## 8. Enforce phase boundaries
Every question must belong to the current phase.

If a question leaks into another phase:
- stop
- park it
- return to current objectives

Depth first, always.

---

## 9. Reality-check mode
Periodically force the LLM into critical mode.

Use prompts like:
- "Challenge this as if you were a senior platform engineer in Zurich"
- "What would break first in a real company?"

Answers must be blunt, not polite.

---

## 10. Alignment checkpoint
After complex exchanges, ask:
"Restate the current state and next step."

If the restatement is wrong:
- correct it
- do not proceed

---

## 11. Kill speculation early
If the LLM starts:
- suggesting tools not chosen
- expanding scope
- optimizing prematurely

Stop and reset with:
"This violates Phase \<N> constraints. Reset."

---

## 12. Promote outputs immediately
Any useful result must be:
- copied into Obsidian
- turned into an ADR, note, or checklist

If it stays only in chat, it does not exist.

---

## Summary rule
The LLM is a sharp tool.
Sharp tools require discipline.