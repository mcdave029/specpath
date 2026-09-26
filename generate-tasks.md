# Rule: Generating a One-Pass Implementation Plan

## Goal

To guide an AI assistant in turning a Ready spec into an ordered implementation plan — one flat sequence of tasks — in a single pass. Every task carries the outcome it delivers, the spec criteria it satisfies, its dependencies, its expected footprint, the existing code it leverages, and the exact command that proves it done. The plan is directly consumable by execution harnesses that extract per-task briefs by heading (superpowers subagent-driven-development's `task-brief` and `review-package` scripts read this format as-is).

The plan is generated once, without pausing for confirmation. Review happens afterwards in `preflight-plan.md`, in a fresh context.

## Output

- **Format:** Markdown (`.md`)
- **Location:** `tasks/` by default — a workspace adapter may override the directory and the filename convention
- **Filename:** `tasks-[feature].md`

## Precondition

The spec must carry `Status: Ready for Implementation`, set by `interrogate-spec.md` after it clears the risk-tier gate from `intake.md`.

- **Spec is Ready:** proceed. Read the spec fully, plus the PRD and research summary if they exist.
- **Spec exists but is still `Draft`:** STOP. Do not plan against a draft. Route back: "The spec is not Ready for Implementation. Run `interrogate-spec.md` first. Planning against a draft spec means the implementer inherits every unresolved ambiguity."
- **No spec at all:** allowed only for work that `intake.md` classified LOW tier with deterministic verification: a single-file fix, a config change, a copy update, anything whose correctness a command can settle. Plan directly from the ticket's acceptance criteria, treating each criterion the way success-criterion IDs are treated below. A plan artifact is still produced. No path through this phase ends with zero artifact.
- **Anything above LOW tier without a Ready spec:** STOP and route back to `generate-spec.md`.

## Process

1. **Verify the precondition** above and resolve it before writing anything.
2. **Read the inputs.** The spec (success criteria and their IDs, invariants, verification intent), the PRD for intent, the research summary for prior art, and the repo's steering documents.
3. **Discover the verification commands.** Find the project's real test runner, linter, type checker, and build from its scripts and config (`package.json`, `Makefile`, `pyproject.toml`, `Gemfile`, CI workflow). Every task needs a concrete runnable command, so discover them now rather than while writing tasks.
4. **Map criteria to work.** Every `SC-n` and every invariant must be claimed by at least one task. A criterion nothing claims is a gap in the plan, not a criterion to drop.
5. **Decide whether a walking skeleton is needed** (see below), then slice the rest into vertical slices.
6. **Anchor the plan.** Record the Base SHA — the branch head this plan is written against. The header carries it, and every line number the plan cites is relative to it, which is what makes the staleness caveat meaningful rather than decorative.
7. **Copy the binding constraints.** Lift the spec's invariants and constraints into `## Global Constraints` VERBATIM, with their exact values. Execution harnesses hand that block to reviewers as their attention lens, so a constraint left out of it is a constraint nobody checks. Do not paraphrase and do not summarize — a rounded threshold is a changed requirement.
8. **Write the whole plan in one pass** — every task, first to last, no confirmation pause. Spec-verification task last. Branch/worktree setup is the execution harness's job, never a plan task.
9. **Check the footprint against PR size** (see below) and flag it if the work should be split into separate tickets.
10. **List relevant files** — spec, PRD, and research first — and **save** to `tasks/tasks-[feature].md`.

## Per-Task Contract

Every task carries these fields. `Escalate if:` appears only where a real stop condition exists; the rest are required for any task that touches code.

| Field | What it holds |
|---|---|
| Outcome | One sentence: the coherent, verifiable thing this task makes true. |
| `Spec:` | The success-criterion IDs (`SC-1`, `SC-4`) and/or named invariants this task addresses. |
| `Depends:` | Upstream task numbers, or `none`. |
| `Footprint:` | Files expected to be created or modified. |
| `Leverage:` | Existing module, pattern, or utility to reuse instead of reinventing. |
| `Interfaces:` | *(optional)* What this task consumes from earlier tasks and produces for later ones — exact names and signatures. An implementer sees only its own task; this field is how cross-task contracts travel. |
| `Deliverable:` | *(no-diff tasks only — required there)* The exact artifact the task produces when its outcome is not a code change: an audit report, a published comment, an evidence file. Names the thing the reviewer reviews (see No-Diff Tasks). |
| `Verify:` | The exact command(s) that prove the task done — one assertion per line (see below). |
| `Escalate if:` | The condition under which the implementer stops and surfaces to the human or amends the spec. |

**Grounding rule for `Footprint:`** Only include exact file paths confirmed from the repo, through the spec, the research summary, or a direct codebase read. If a path is unknown, write `[path TBD — confirm before implementing]`. Do not invent paths with false precision. Fabricated file paths are the most common cause of tasks that fail on the first attempt.

**`Verify:` must be concrete.** The project's own test runner, linter, or build, scoped to the code the task touches. Not "run the tests". Use a placeholder only when the command is genuinely undiscoverable from the repo, and mark it as one.

**One assertion per line.** When proving a task takes more than two independent assertions, write one command per line — each line must pass on its own — or point at a committed verification script. Never stack assertions into a long `&&` chain: a chain fails as a unit without saying which clause failed, and quoting that survives one shell breaks in another, so a chain whose clauses all pass individually can still exit non-zero as a whole. One line per assertion is also what lets a re-run isolate the failing clause immediately.

**Executable harnesses are committed files, referenced by path — never inlined.** When verification needs more than the project's own commands (a purpose-built assertion script, a render-and-diff harness), the harness is committed beside the plan — in the plan's own directory by default; a workspace adapter may name another location — and that committed file is the **executable of record**. The `Verify:` line references it by repo-relative path. Do not inline the harness body in the plan, and do not reference a path outside the repository — a session-local scratch directory does not survive the session, so its Verify line dangles for whoever resumes. An inline copy diverges from the executable the first time a fix round has to change the harness, and a plan frozen at preflight cannot follow — a later session re-materializing the harness from plan text would run the stale version. Changing a committed harness is a normal code change, reviewed like one.

**`Verify:` commands never contain secret values.** When a verification step needs a credential, key, or token, the command derives it at run time — from the environment, a secrets manager, or version history (`git show <sha>:<path>` for a value that legitimately lives in a historical file) — and never quotes the value itself. A committed plan is a durable artifact: a pasted secret in a Verify line is a leak, whether or not any scanner's rules would match it.

**`Escalate if:` exists so the implementer does not improvise past a boundary.** Typical conditions: a protected module or interface would have to be touched; the actual footprint materially exceeds what is listed; a spec invariant cannot be satisfied as written. The implementer stops there. It does not renegotiate the spec on its own.

**Line references go stale.** Task briefs are read after earlier tasks have already changed the code, so anchor references by content — function or symbol names, unique strings — rather than bare line numbers. Where a `file:line` reference is genuinely useful, the task block MUST carry the caveat beside it: *line numbers are as of the plan's Base SHA; re-locate by content before editing.* The caveat lives inside the task block because brief extraction hands the implementer only that block. The same rule binds the code the plan produces: an implementer never writes a `file:line` into a source comment; it anchors by name. And the plan never prescribes a code comment: where a fact must be recorded, prescribe the example that pins it.

Example task block:

```markdown
### Task 3: Reject withdrawal requests above the daily limit

Spec: SC-3, SC-4, invariant "no balance goes negative"
Depends: 2
Footprint: src/domain/withdrawal.ext, test/domain/withdrawal_test.ext
Leverage: src/domain/limits.ext (existing threshold lookup)
Verify: <test-runner> test/domain/withdrawal_test.ext && <linter> src/domain/withdrawal.ext
Escalate if: enforcing the limit requires changing the shared ledger interface
```

## Task Design Rules

### Task Sizing

A task is sized by coherent, verifiable scope: **one outcome, one verification, a footprint the implementer can hold in mind at once.** Not by a clock and not by a file count. A three-line change across five call sites can be one task; two unrelated behaviors in one file are two. The test is whether a single `Verify:` line actually settles it. If proving the task done takes two unrelated commands checking two unrelated things, it is two tasks.

**Title discipline:** avoid vague words in task titles. If the title contains "system", "integration", "complete", or "setup", the task is almost certainly oversized, so split it. An implementer handed an oversized task makes architectural decisions mid-implementation, which is the most expensive time to make them.

### Walking Skeleton (Conditional)

Include a walking-skeleton task **only when the change creates new cross-layer structural boundaries**: a new module, service, or interface spanning layers, where the wiring itself is an unproven assumption. Then the skeleton comes first, as stubs, interfaces, type definitions, and empty modules with correct signatures. No behavior, no logic. It forces the architectural decisions into the open before any logic is written.

For changes that live inside existing structure, **skip it** and start directly with the first vertical slice. A skeleton over structure that already exists is ceremony, and it delays the first real verification.

When the skeleton is used, stub with the lightest pattern that proves the wiring:

| Pattern | When to use |
|---|---|
| Return empty or hardcoded value | Data layer not yet implemented |
| Comment placeholder | Wiring exists, logic deferred to a named later task |
| Throw "not implemented" | Interface method that must exist now |
| Skipped or pending test | Test wiring needed, behavior deferred |

Every stub must be replaceable by a later task without touching surrounding code.

### Vertical Slices

Slice by user-visible outcome, not by layer. Each task delivers something observable end-to-end.

**Wrong (layer slicing):**
- Task 2: Build all data access
- Task 3: Build all business logic
- Task 4: Build all UI

**Right (vertical slicing):**
- Task 2: User can [do X] (data access + logic + UI for that slice)
- Task 3: User can [do Y] (data access + logic + UI for that slice)

### No-Diff Tasks (audits, evidence, published reports)

Some tasks legitimately produce no code diff: a repo audit whose deliverable is a findings report, an evidence-collection task, a report published to the tracker. These carry the same contract with two adjustments:

- **`Deliverable:` is required** — the exact artifact the task produces (a path or a destination). A no-diff task without a named deliverable is unverifiable and unreviewable.
- **The review gate is an artifact review, not a diff review.** Execution harnesses that review by code diff have nothing to extract for these tasks; the harness must route them to a reviewer that receives the deliverable itself, the task's own section, and the spec criteria it claims to satisfy. Skipping review because there is no diff is not an option: a published claim that is wrong costs a public correction afterward, which is exactly the failure a review seat exists to catch beforehand.

`Verify:` still applies wherever part of the deliverable is mechanically checkable (the artifact exists, its counts match a pinned extraction command); the artifact review covers what commands cannot.

### PR-Size Discipline

The plan targets **one vertical slice per pull request**. Before finalizing, add up the expected footprint across all tasks.

If the total exceeds what one human reviewer can hold in a single sitting, a soft guideline of roughly 400 changed lines excluding lockfiles, generated schema files, and snapshots, the work should be split into separate tickets **before implementation starts**, not planned as one oversized PR. Count behavior units and changed contracts more heavily than raw lines: a 600-line mechanical rename reviews easily; a 200-line change across four contracts does not.

The planner's job is to **flag** this, with the estimate and the suggested split points. Splitting the work is a human and tracker action, not something this phase performs.

### Format Contract (machine extraction)

Execution harnesses slice the plan mechanically: per-task briefs are extracted by heading, and an implementer receives ONLY its own task's section. The grammar is rigid:

1. **One heading per task:** `### Task N: [title]` — flat integers starting at 1, in file order. Never hierarchical numbering (`Task 2.1` defeats number-boundary matching). Amendments continue this numbering with the next integer (see Amendments Append).
2. **A task's section is self-sufficient.** Everything between its heading and the next task heading travels with the task — and nothing else does. Exact values, code, and context the implementer needs go inside the section.
3. **Nothing after the last task.** Trailing sections would be swept into the final task's brief. Every plan-level section precedes `## Tasks`.
4. **No other heading may look like a task.** No heading anywhere else in the file may match the shape `Task <number>`. (Extractors skip fenced code blocks, so examples inside fences are safe.)
5. **The header names the Spec.** Harness controllers treat the spec as the binding authority the plan argues from; the plan must say where it is.

### The Plan Freezes at Preflight

Once the plan passes its preflight review, the file is immutable for the duration of execution: no checkbox ticks, no status notes, no inline edits. Progress tracking belongs to the execution harness's ledger — one owner per datum — which is why this format has no checkboxes at all. Mid-execution bookkeeping edits pollute review diffs or force noise commits; both failure modes disappear when the plan is read-only. A plan defect discovered mid-execution is ruled on in the harness's ledger; a material conflict with the spec amends the spec (versioned) and **appends** tasks per the amendment contract below — an amendment event, never bookkeeping.

### Amendments Append — Completed Work Is Never Regenerated

A frozen plan admits exactly one kind of write. After an approved spec amendment, the new work is APPENDED after the last existing task, continuing the flat numbering (the next integer onward — never a hierarchical insertion like "1.b", which defeats number-boundary extraction). Nothing already in the file is edited:

- **Each appended task is a full, self-sufficient brief** per the Per-Task Contract, and opens with its amendment provenance inside the section — the spec version that sanctioned it and the approval reference — because brief extraction hands the implementer only that block.
- **Existing sections are never regenerated.** A completed task's section describes work already executed; "regenerating the affected section" produces a block that mixes done-work with new work, and a brief extracted from it hands the implementer already-executed instructions. If an amendment supersedes a task that has not yet run, the supersession is recorded in the harness's ledger (the task is skipped there) — the plan text stands.
- **The appended set ends with a verification task — full or scoped, decided by timing.** If the original "Verify all spec success criteria" task has NOT yet run when the amendment lands, the harness's ledger rules it superseded (like any superseded task — the text stands) and the appended set closes with the FULL spec-verification task, run against the amended spec; otherwise the not-yet-run original would false-fail on amended criteria whose implementing tasks sit after it in file order, or a scoped sweep would leave the plan with no full sweep at all. Only an amendment landing AFTER the original verification task already ran closes with a scoped re-verification: the amended criteria, plus any criterion whose earlier evidence the appended work invalidates. Either way the amendment never reaches back to edit the original task's text.
- **The append is committed as a dedicated amendment-event commit**, so the plan's history shows exactly which tasks each spec version added.

The Format Contract survives amendments by construction: appended tasks are new last tasks (nothing follows them), the numbering stays flat and increasing, and every block is self-sufficient.

### Division of Labor

This phase owns plan CONTENT: what to build, in what order, against which criteria, proven by which commands. The execution harness owns EXECUTION: orchestration roles, context isolation, dispatch, commit discipline, per-task review, quality gates, and fix loops. (superpowers subagent-driven-development is one such harness; its `task-brief` and `review-package` scripts consume this format directly.) Plans do not instruct the executor on how to run itself.

### Final Task: Spec Verification

The last task is always "Verify all spec success criteria." It does three things: confirms that **every `SC-n` has passing verification evidence**, meaning a command that ran and passed, named against the criterion; traces every constraint and invariant through the code rather than only through tests; and runs the relevant test scope to confirm no regressions.

When an amendment appends tasks, the appended set closes with its own verification task — the full sweep if the original has not yet run (the ledger rules the original superseded), a scoped re-verification otherwise (see Amendments Append); "last" means last in file order after any amendments.

## Output Format

```markdown
# Implementation Plan: [Feature Name]

**Ticket:** [reference]
**Spec:** `tasks/spec-[feature].md` (Status: Ready for Implementation)
**Base:** [git SHA of the branch head this plan was written against]
**Plan status:** frozen at preflight — progress lives in the execution harness's ledger, never in this file

## Global Constraints

<!-- Binding requirements copied VERBATIM from the spec: the invariants and
     constraints every task must respect, with exact values. Execution harnesses
     hand this block to reviewers as their attention lens — a constraint missing
     here is a constraint nobody checks. Close with the two standing rules below. -->
- [invariant or constraint, verbatim from the spec, one per line]
- Verify commands never contain secret values — derive at run time (environment, secrets manager, or `git show <sha>:<path>`).
- Line numbers cited in tasks are as of Base above — re-locate by content before editing.

## Relevant Files

<!-- Spec, PRD, and research first -->
- `tasks/spec-[feature].md` - Behavioral spec (Status: Ready for Implementation). Read before writing any code.
- `tasks/prd-[feature].md` - Product requirements (only if one exists).
- `tasks/research-[feature].md` - Research summary (if it exists).
- `path/to/file.ext` - Why this file is relevant.

## Footprint & PR Size

- Estimated changed lines: [n] (excluding lockfiles, generated schema, snapshots)
- Behavior units / contracts changed: [n]
- Verdict: [fits one PR] | [FLAG: exceeds one review. Suggested split: ...]

## Tasks

<!-- Walking-skeleton task ONLY if the change creates new cross-layer structural
     boundaries (see Task Design Rules). -->

### Task 1: [Outcome in one sentence]

Spec: SC-1, SC-2
Depends: none
Footprint: [path/to/file.ext], [path TBD — confirm before implementing]
Leverage: [path/to/existing/module]
Interfaces: [optional — what this produces that later tasks consume, exact names/signatures]
Deliverable: [no-diff tasks only — the exact artifact produced (path or destination)]
Verify: [one assertion per line, or a committed verification script referenced by path]
Escalate if: [condition requiring a stop and a human decision]

[Everything else the implementer needs — exact values, code, edge cases — goes
here, inside the task's own section.]

### Task 2: [Outcome in one sentence]

Spec: SC-3, invariant "[name]"
Depends: 1
Footprint: [path/to/file.ext]
Leverage: [path/to/existing/pattern]
Verify: [test-runner scoped to the touched code]

### Task N: Verify all spec success criteria

Spec: all SC-n, all invariants
Depends: all prior tasks
Verify: [the relevant test scope]

- Confirm every SC-n has passing verification evidence, named against the criterion.
- Trace every constraint and invariant through the code, not only through tests.
- Run the relevant test scope and confirm no regressions.
```

## Final Instructions

1. Do NOT implement anything in this phase. This phase produces a plan.
2. Generate the full plan in one pass, with no confirmation pause between tasks.
3. Save to `tasks/tasks-[feature].md` unless the workspace adapter specifies another location.
4. After saving, the next step is: **run `preflight-plan.md` in a FRESH context** before implementation begins. Do not roll into execution from this session.
5. The saved plan must satisfy the Format Contract — a plan that cannot be sliced by task heading fails its consumers.

## Target Audience

Assume the reader is a developer implementing the feature with AI assistance. Individual tasks are handed to implementer subagents as extracted briefs containing ONLY that task's section, plus whatever header and constraints the harness controller supplies, so each task block must be self-sufficient: an implementer reading one task block should know what to build, what it must satisfy, where it lives, what to reuse, how to prove it, and when to stop and ask.
