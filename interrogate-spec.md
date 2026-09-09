# Rule: Fresh-Context Spec Interrogation

## Goal

Break the spec before implementation does. A fresh-context interrogator — one that did NOT author the spec and does not carry the authoring conversation — reads the spec cold and hunts for the ways it could produce a wrong implementation. The authoring context is blind to its own assumptions; fresh context is the point, not a nicety.

This replaces the mandatory human interview of earlier specpath versions. The human is consulted only where genuine judgment is required — never one question per category for ceremony.

## When to Use This

After `generate-spec.md`, before `generate-tasks.md`. The interrogator must be a fresh context: a new session or an isolated sub-agent given ONLY the spec, the ticket, and repository access — never the conversation that wrote the spec.

## What the Interrogator Checks

Work through all of these; report only real findings:

1. **Data decisions** — missing/null/malformed inputs, boundary values, validation placement.
2. **Conflict resolution** — which rule wins when two valid rules apply; priority order under simultaneous conditions.
3. **Pattern selection** — cases where the chosen reference pattern does not apply; existing code contradicting it.
4. **Failure recovery** — retries, exhaustion, partial success, error surfacing, rollback/compensation.
5. **Boundary conditions** — scale, concurrency, scope edges, adjacent-system interference.
6. **Invariants** — are the stated invariants actually preserved by the specified change? Are any load-bearing invariants missing?
7. **Verification** — is every success criterion verifiable as written? Does the verification intent actually cover it?
8. **Ambiguity and contradiction** — statements a reasonable implementer could read two ways; sections that contradict each other or the ticket.
9. **Missing failure cases** — failure paths the spec never mentions.
10. **Wrong-satisfaction paths** — ways an implementation could satisfy the letter of every criterion while defeating the intent.
11. **Criteria-vs-criteria consistency** — do the success criteria cohere as a set? Two criteria no single implementation can satisfy simultaneously, overlapping criteria that disagree about the same behavior, or one criterion silently narrowing another. A spec-internal contradiction caught here is the cheapest catch it will ever get; missed, it resurfaces as a plan-review escalation.

## Routing Every Finding

Each finding takes exactly one route:

| Finding type | Route |
|---|---|
| Resolvable by evidence (repo fact, library behavior, doc lookup) | Research it now (`research.md`, bounded pass) and fix the spec with tagged evidence |
| Mechanically fixable ambiguity (wording, missing case with an obvious answer consistent with intent) | Fix the spec directly; record what changed |
| Material product / architecture / security-policy judgment | Ask the human — the specific question, the options, a recommendation |
| Already settled by a documented standing project decision (a decisions file, ADR, or preloaded skill the project provides) | Apply the decision and cite it in the resolution; never route it to the human as an open question. A spec that contradicts the decision is a spec fix, not a question |
| Not a real problem on inspection | Discard; do not pad the report |

Never ask the human a question that evidence could answer, or that a standing project decision already answers. Never silently guess on a material judgment.

## Outcome by Risk Tier

When no unresolved material issue remains, proceed per the spec's risk tier (from `intake.md`):

| Tier | Action |
|---|---|
| **LOW** | Set spec `Status: Ready for Implementation`; continue. |
| **STANDARD** | Post/surface the spec where the humans will see it (ticket link, session summary), set Status Ready, continue — the human may object at any time. |
| **HIGH** | STOP. Explicit human approval of the spec is required before Status becomes Ready. |

## Output Contract

The interrogation records (in your workspace's workflow state, alongside the spec):

- the spec version and revision interrogated;
- each finding: category, severity (material / mechanical / informational), the route taken, and its resolution;
- the outcome: clean, fixed-and-clean, or escalated;
- the tier disposition (continued / posted / awaiting approval) and any post/approval reference.

Spec edits made during interrogation are applied to the spec file directly. If interrogation materially changes the spec's architecture or intent, that is not a fix — it is a redesign; route it to the human.

## Final Instructions

1. The interrogator receives the spec, the ticket, and the repository — never the authoring conversation.
2. Report findings, not a category-by-category essay. A clean category is silence.
3. Do NOT generate tasks here. Next step after Ready: `generate-tasks.md`.
4. A spec that reaches `generate-tasks.md` without interrogation recorded is an incomplete pipeline.
