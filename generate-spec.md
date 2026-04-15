# Rule: Generating a Behavioral Spec from a PRD

## Goal

Translate a PRD (and research summary, if available) into a precise behavioral specification an AI agent can implement without guessing.

**Constraints matter more than requirements.** "Do NOT pre-fetch more than 3 items" eliminates wrong implementations. "Make it fast" does not. Every constraint in the spec is one fewer assumption the implementing agent has to make.

This is the technical layer between the product layer (PRD) and the execution layer (tasks). The spec is written for the AI, not for stakeholders.

## When to Use This

Run this step after `create-prd.md` (and `research.md` if Phase 1 ran) and **before** `generate-tasks.md`. The task list will reference the spec and frame all implementation work as "satisfy these constraints and success criteria."

## Process

1. **Receive inputs:** The user provides `tasks/prd-[feature].md` and, if it exists, `tasks/research-[feature].md`.
2. **Ask clarifying questions:** 2–3 questions maximum — only for gaps that block spec precision. See guidelines below.
3. **Generate `spec-[feature].md`:** Use the 5-section template below.
4. **Self-critique:** Before presenting, ask internally: "What would a critical reviewer flag as ambiguous or missing?" Address any P1 issues (blocking ambiguities) before presenting.
5. **Present to user:** Show the draft spec. Wait for review and corrections.
6. **Save:** Write to `tasks/spec-[feature].md`.

## Clarifying Question Guidelines

Ask only when the answer is not clear from the PRD or research. Focus on:

- Which existing components or systems does this touch?
- Who is allowed to perform this action?
- What happens in the failure case?
- Are there performance or scale constraints?

**Limit to 3 questions.** If more are needed, the PRD needs more work first.

Number questions and provide lettered options where possible:

```
1. Which existing system handles [X]?
   A. [Option]
   B. [Option]
   C. I'll decide during implementation
```

## Output: `spec-[feature].md`

```markdown
# Spec: [Feature Name]

**PRD:** tasks/prd-[feature].md
**Research:** tasks/research-[feature].md (omit if no research was run)
**Status:** Draft

---

## 1. Reference Architecture

How is this type of problem solved in this codebase or ecosystem?
What existing pattern should be followed?

[Describe the pattern, not the implementation. Reference existing code
locations if known from research. If no clear precedent exists, name
the general pattern and why it applies here.]

---

## 2. Current Architecture

What exists today that this feature touches or extends?

[List affected modules, services, data stores, and interfaces.
Describe current state only — not desired state.
If the feature is entirely new with no touchpoints, say so.]

---

## 3. Constraints

What must NOT happen. This is the most important section.

- Do NOT [specific action to avoid]
- Do NOT [another constraint]
- NEVER [hard invariant that cannot be violated]
- ALWAYS [non-negotiable requirement]

[Make each constraint specific enough that an AI agent cannot misinterpret it.

Vague: "Handle errors properly."
Specific: "On network failure, retry 3 times with exponential backoff
starting at 100ms, then surface the error to the caller — do NOT
swallow silently or return a partial result."]

---

## 4. Implementation Plan

[Plain language description of the approach. Phased if the feature is large.
Reference the pattern from Section 1. Reference the constraints from Section 3.
This is not pseudocode — it is a narrative the AI uses to orient itself.]

**Phase 1:** [Walking skeleton — stubs, interfaces, type definitions, empty
modules with correct signatures. No behavior.]

**Phase 2:** [Core behavior and logic]

**Phase 3:** [Edge cases, error handling, and polish]

---

## 5. Success Criteria

How we know this is done. Must be measurable and observable.
Use Given/When/Then format.

- [ ] Given [initial context], when [action is taken], then [observable outcome]
- [ ] Given [initial context], when [action is taken], then [observable outcome]
- [ ] [Performance criterion: operation completes within X under Y conditions]
- [ ] [Security or invariant criterion: X is never exposed / always enforced]

---

## Open Questions

[Anything unresolved that the interview phase should surface.
If nothing is unresolved, write "None — spec is ready for interview."]
```

## Spec Status Lifecycle

The `Status` field in the spec header moves through three states. Never skip a state.

| Status | Set by | Meaning |
|---|---|---|
| `Draft` | `generate-spec.md` on creation | Spec written, not yet reviewed |
| `Ready for Interview` | User after reviewing the draft | Spec is accurate, ready to be interrogated |
| `Ready for Implementation` | `interview-spec.md` after all questions resolved | All ambiguities resolved, safe to generate tasks |

The spec should never reach `generate-tasks.md` while still in `Draft` status.

## What This Spec Does NOT Contain

- API endpoint request/response shapes — too prescriptive, constrains the AI unnecessarily
- Test skeletons — tests are the backpressure mechanism, separate from the spec
- Database schema definitions — these belong in implementation, derived from constraints
- State machine diagrams — unless the feature is fundamentally state-driven and the diagram is the clearest way to express it

## Lightweight Spec Variant

For features that do not warrant a full spec but are too risky to skip entirely:

```markdown
# Lightweight Spec: [Feature Name]

**PRD:** tasks/prd-[feature].md
**Status:** Ready for Implementation

## Constraints
- Do NOT [constraint]
- NEVER [constraint]
- ALWAYS [constraint]

## Success Criteria
- [ ] Given [context], when [action], then [outcome]
- [ ] Given [context], when [action], then [outcome]
```

80% of the value. 20% of the overhead. Use when:
- Files affected = 2–5
- Requirements are clear but the change is non-trivial
- A full spec would be disproportionate to the scope

## Final Instructions

1. **Do NOT write implementation code.** Spec only.
2. **Do NOT generate test skeletons.** Tests are a separate concern.
3. **Constraints section is mandatory.** Never leave it empty. If you cannot identify constraints, the PRD is not clear enough.
4. **Self-critique before presenting:** "What ambiguities remain that could cause a wrong implementation?"
5. **Present the spec to the user before saving.** Wait for approval or corrections.
6. After saving: "Spec saved to `tasks/spec-[feature].md`. Next step: run `interview-spec.md` to surface failure-causing questions before tasks are created."
