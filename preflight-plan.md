# Rule: Plan Preflight

## Goal

A fresh-context review of the implementation plan before any task executes. The plan is the last cheap place to catch a structural mistake: after this, errors surface as failed implementation rounds. Preflight is a feasibility check of the plan against the spec — it does not re-review the spec (that was interrogation's job) and it does not implement.

## When to Use This

After `generate-tasks.md`, before implementation begins. Like interrogation, the reviewer must be a fresh context: a new session or isolated sub-agent given the plan, the spec, the ticket, and repository access — never the planning conversation.

## What Preflight Checks

1. **Dependencies** — every task's `Depends:` is complete and acyclic; no task silently relies on an outcome no earlier task produces.
2. **Contradictions** — no two tasks undo or fight each other; no task contradicts a spec constraint or invariant.
3. **Task coherence** — each task is one verifiable outcome; oversized or incoherent tasks (multiple outcomes, vague titles, unbounded footprint) are flagged for splitting.
4. **Verification coverage** — every task has a runnable `Verify:` command; every spec success criterion (SC-n) is addressed by at least one task; the commands actually exist in the repository (check its scripts/config — do not trust the plan's word).
5. **Protected boundaries** — the combined footprint is checked against the workspace's protected-path lists. A task that would touch a protected path in a plan classified below HIGH is a classification failure, not a plan detail.
6. **Scope vs spec** — the plan's total footprint matches the spec's implementation boundaries; unexplained expansion (files or systems the spec never mentions) is flagged.
7. **Risk re-evaluation** — with the actual plan and footprint now visible, re-apply the risk rubric from `intake.md`. The tier may stay or escalate — never silently drop. An escalation to HIGH re-triggers the human spec-approval gate before implementation.

## Routing Findings

| Finding | Route |
|---|---|
| Mechanical plan problem (missing dependency edge, oversized task, missing/wrong verify command, ordering) | Correct the plan directly; record each correction |
| Footprint/scope expansion with an evident cause | Correct the plan to match the spec, or flag for split if the spec genuinely needs the larger footprint |
| Material conflict with the spec or architecture | Escalate to the human; the resolution amends the spec (new spec version), and affected plan sections are regenerated |
| Already settled by a documented standing project decision (a decisions file, ADR, or preloaded skill the project provides) | Apply the decision, cite it, correct the plan if it contradicts the decision; never escalate it as an open question |
| Protected-boundary hit or tier escalation | Reclassify; apply the HIGH-tier gate before proceeding |

## Output Contract

Preflight records (in your workspace's workflow state, alongside the plan) — deliberately distinct from the interrogation record, so spec findings and plan-feasibility findings never blur:

- the plan revision reviewed;
- pass/fail per check (dependencies, contradictions, coherence, verification coverage, protected boundaries, scope-vs-spec);
- each correction made, and each escalation raised;
- the risk re-evaluation result (tier, changed or not, reason);
- the outcome: clean, corrected, or escalated.

## Final Instructions

1. Fresh context only — the reviewer never inherits the planning conversation.
2. Correct mechanical problems yourself; never "note" a fixable defect and proceed with it in place.
3. A clean or corrected preflight is the entry signal for the implementation loop. An escalated preflight blocks implementation until the human decides.
