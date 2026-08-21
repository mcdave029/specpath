# Rule: Generating an Engineering Spec

## Goal

Produce the **engineering contract** for a ticket or feature: what changes, what must stay true, where the change stops, and how anyone will know it worked.

**Delta-oriented.** For a change to existing behavior, state the exact change *against current behavior*. Never regenerate a description of the whole system: a spec that re-describes what already works buries the one paragraph that matters and invites the implementing agent to rewrite things nobody asked it to touch.

**A dated decision record.** Like an ADR, the spec captures what was decided, when, and on what evidence. It goes stale by design and does not claim to mirror the code forever. A later reader treats it as "this is what we decided on that date," not as live documentation.

**Constraints matter more than requirements.** "Do NOT pre-fetch more than 3 items" eliminates wrong implementations. "Make it fast" does not. Every constraint written down is one fewer assumption the implementing agent has to invent.

## Inputs

| Input | Role |
|---|---|
| The ticket | **Primary intent source** — user story, acceptance criteria, success measure |
| Risk tier from `intake.md` | LOW / STANDARD / HIGH; drives the status gate below |
| PRD | Only if one was generated (`create-prd.md` runs only when ticket intent was insufficient) |
| Research artifacts | If any were produced; `research.md` is on-demand, not a mandatory phase |
| Design / brainstorm conclusions | The decisions already reached with the humans |

The ticket is the intent of record. Where the ticket and a stale PRD disagree, the ticket wins.

## Process

1. **Check steering documents.** Read the project's constitution files (`AGENTS.md`, `product.md`, `tech.md`, `structure.md`, or whatever root file your tool reads) before writing anything. They define architecture, conventions, and prohibitions that would otherwise be guessed. If none exist, say so in the spec: every architectural claim then rests on the codebase and research alone, and must be tagged accordingly in §4.
2. **Read the ticket, its risk tier, and any research.** Confirm you can state the intent in one sentence before writing the spec.
3. **Write the spec** using the eleven-section template below.
4. **Self-critique.** Ask internally: "What would a critical reviewer flag as ambiguous, unverifiable, or missing?" Fix every blocking ambiguity before handing off.
5. **Do NOT interview the human.** Route each gap by its kind:
   - **Material product, architecture, or security judgment** — ask the human directly, as a small number of targeted questions. These are the calls you are not entitled to make.
   - **Repo or library facts** — do not ask; invoke `research.md` and find out.
   - **Everything else** — decide it, record the decision and its reasoning in the spec, and move on. An unrecorded decision is the same as an unanswered question.
6. **Hand off to `interrogate-spec.md`.** The interrogation runs in a fresh context and is what moves the spec toward `Ready for Implementation`.

**Recovery point.** The spec is the session-restart artifact. If a session derails, context fills, or implementation drifts from intent — open a new session, pin the spec, resume from the last committed task. The spec carries everything the next session needs; no context rebuilding required.

## Output: `spec-[feature].md`

```markdown
# Spec: [Feature Name]

**Ticket:** [ticket reference]
**Date:** [YYYY-MM-DD]
**Author:** [author]
**Risk tier:** LOW | STANDARD | HIGH
**Status:** Draft
**Spec version:** 1

---

## 1. Source & Context

[Link the ticket. Link the PRD only if one exists, and research artifacts if any.
One paragraph: what this ticket is asking for, in your own words.]

---

## 2. Risk Classification

**Tier:** [LOW | STANDARD | HIGH]

[One paragraph of rationale against your team's rubric — the workspace adapter
supplies the criteria. Name the specific factors that set the tier.]

Uncertainty rounds **UP**. The tier is re-evaluated at plan preflight and again at
final-diff verification; it may escalate there, but never silently drops.

---

## 3. Change / Delta

The exact change against current behavior. This is the heart of the spec.

**Today:** [what the system does now]
**After this change:** [what it will do instead]

[Enumerate each behavioral difference. For genuinely new surface area with no prior
behavior, say so and describe only the new surface — not the surrounding system.]

---

## 4. Current / Reference Architecture

What exists today that this touches, and the pattern to follow.

[Affected modules, services, data stores, interfaces — then the existing pattern this
change should follow, with code locations if known. Describe the pattern, not the
implementation. If no precedent exists, name the general pattern and why it applies.]

Tag every claim with its evidence quality:
- **[Confirmed]** — directly observed in the codebase, research, or steering documents
- **[Inferred]** — a logical conclusion from observed evidence, not directly read
- **[Unresolved]** — could not be determined; carry it to Open Questions

Do not present Inferred architecture as Confirmed. An inferred module that does not
exist produces a task that fails on its first line.

---

## 5. Invariants

What MUST remain true after the change — system properties, data integrity, security
properties.

- [e.g. "every ledger entry still balances to zero"]
- [e.g. "a request without a valid session can never read another tenant's rows"]

Invariants are properties *preserved*; constraints (§6) are actions *forbidden or
required*. An invariant survives any implementation; a constraint governs this one.

---

## 6. Constraints

What must NOT happen, and what must ALWAYS happen.

- Do NOT [specific action to avoid]
- NEVER [hard rule that cannot be violated]
- ALWAYS [non-negotiable requirement]

Make each one specific enough that an agent cannot reinterpret it.

Vague: "Handle errors properly."
Specific: "On network failure, retry 3 times with exponential backoff starting at
100ms, then surface the error to the caller — do NOT swallow it silently or return a
partial result."

---

## 7. Contract Decisions

*(when applicable — write "None" if this change crosses no durable contract)*

Durable contracts belong here when they are part of the architectural decision: API
request/response shapes, event formats, message envelopes, schemas consumed by other
systems or teams.

The line: **durable contracts in, implementation prescription out.** How a function is
written is out of scope — unless the prescription is what protects a boundary or a
contract, in which case it belongs here and names the boundary it protects.

---

## 8. Data Scope

*(required — never omit; write "No data-lifecycle change" if that is the case)*

- **Data touched:** [entities, fields, stores]
- **Lifecycle changes:** [create / read / retain / delete / migrate]
- **Personal or otherwise sensitive data involved:** [yes/no, and which]

Changes to personal-data handling or to data lifecycle classify **HIGH** tier. Your
team's rubric, supplied by the workspace adapter, binds the specifics of what counts.

---

## 9. Implementation Boundaries

Where the change stops.

- **Out of bounds:** [files, modules, systems this change must not touch]
- **Deferred:** [adjacent work that is real but explicitly not in this ticket, with
  where it goes instead]

---

## 10. Success Criteria

Measurable and observable. Each criterion carries a stable ID so verification evidence
can bind to it later.

- **SC-1** — Given [context], when [action], then [observable outcome]
- **SC-2** — Given [context], when [action], then [observable outcome]
- **SC-3** — [performance, security, or invariant criterion: X completes within Y / is
  never exposed / is always enforced]

IDs are stable. If an amendment drops a criterion, mark it withdrawn rather than
renumbering the rest.

---

## 11. Verification Intent

How each success criterion will be verified — the *kind* of check, not the command.

| ID | Verification |
|---|---|
| SC-1 | [unit test / integration test / lint / build / end-to-end walk / manual or visual review] |
| SC-2 | [...] |

Intent only. The plan supplies exact commands per task.

---

## Open Questions

[Anything unresolved, each tagged with who or what resolves it — human judgment,
research, or the interrogation. If nothing is unresolved, write "None."]
```

## Spec Status Lifecycle

Two states, not three. There is no `Ready for Interview` state, and no unconditional human review gate.

| Status | Set by | Meaning |
|---|---|---|
| `Draft` | `generate-spec.md` on creation | Spec written; not yet interrogated |
| `Ready for Implementation` | after `interrogate-spec.md`, subject to the tier gate below | Safe to plan against |

The transition is gated by risk tier:

| Tier | Gate |
|---|---|
| LOW | Mark `Ready for Implementation` and continue |
| STANDARD | Post the spec where the humans will see it — they may object at any time — and continue |
| HIGH | **STOP.** Explicit human approval is required before `Ready for Implementation` |

**Amendments after Ready** increment `Spec version` and re-trigger the tier gate for the amended content. A HIGH-tier amendment needs approval again; it does not inherit the earlier one.

## What This Spec Does NOT Contain

- **Test skeletons.** Tests are the backpressure mechanism, generated with the plan — not here.
- **Implementation prescription that does not protect a boundary or contract.** If it does protect one, it goes in §7 and says which.
- **A full-system description where a delta suffices.** Regenerating what already works is noise, and noise is what implementing agents act on by mistake.

One format only. A LOW-tier spec is naturally short because the delta is small — sections may collapse to a single line ("Contract Decisions: none"), but every section is present. Absence is a signal; a missing section is indistinguishable from a forgotten one.

## Final Instructions

1. **Do NOT write implementation code.** Spec only.
2. **Do NOT generate test skeletons.**
3. **Constraints, Invariants, and Data Scope are mandatory.** If you cannot fill them, the ticket intent is not clear enough yet — go back to `intake.md`.
4. **Tag every architectural claim** [Confirmed] / [Inferred] / [Unresolved]. Never upgrade a guess.
5. **Self-critique before handing off:** "What ambiguity here could still cause a wrong implementation?"
6. **Save to `tasks/spec-[feature].md`** by default. Workspace adapters may override the location and naming convention.
7. After saving: "Spec saved to `tasks/spec-[feature].md`. Next step: run `interrogate-spec.md` in a **fresh context** — not this authoring conversation."
