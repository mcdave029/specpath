# Rule: Generating a One-Pass Implementation Plan

## Goal

To guide an AI assistant in turning a Ready spec into an ordered implementation plan, parent tasks and sub-tasks together, in a single pass. Every task carries the outcome it delivers, the spec criteria it satisfies, its dependencies, its expected footprint, the existing code it leverages, and the exact command that proves it done.

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
6. **Write the whole plan in one pass** — parent tasks and sub-tasks together, no confirmation pause. Branch-creation task first, spec-verification task last.
7. **Check the footprint against PR size** (see below) and flag it if the work should be split into separate tickets.
8. **List relevant files** — spec, PRD, and research first — and **save** to `tasks/tasks-[feature].md`.

## Per-Task Contract

Every task — parent or sub-task — carries these fields. `Escalate if:` appears only where a real stop condition exists; the rest are required for any task that touches code.

| Field | What it holds |
|---|---|
| Outcome | One sentence: the coherent, verifiable thing this task makes true. |
| `Spec:` | The success-criterion IDs (`SC-1`, `SC-4`) and/or named invariants this task addresses. |
| `Depends:` | Upstream task numbers, or `none`. |
| `Footprint:` | Files expected to be created or modified. |
| `Leverage:` | Existing module, pattern, or utility to reuse instead of reinventing. |
| `Verify:` | The exact command(s) that prove the task done. |
| `Escalate if:` | The condition under which the implementer stops and surfaces to the human or amends the spec. |

**Grounding rule for `Footprint:`** Only include exact file paths confirmed from the repo, through the spec, the research summary, or a direct codebase read. If a path is unknown, write `[path TBD — confirm before implementing]`. Do not invent paths with false precision. Fabricated file paths are the most common cause of tasks that fail on the first attempt.

**`Verify:` must be concrete.** The project's own test runner, linter, or build, scoped to the code the task touches. Not "run the tests". Use a placeholder only when the command is genuinely undiscoverable from the repo, and mark it as one.

**`Escalate if:` exists so the implementer does not improvise past a boundary.** Typical conditions: a protected module or interface would have to be touched; the actual footprint materially exceeds what is listed; a spec invariant cannot be satisfied as written. The implementer stops there. It does not renegotiate the spec on its own.

Example task block:

```markdown
- [ ] 2.1 Reject withdrawal requests above the daily limit
  Spec: SC-3, SC-4, invariant "no balance goes negative"
  Depends: 1.0
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
- Task 2.0: Build all data access
- Task 3.0: Build all business logic
- Task 4.0: Build all UI

**Right (vertical slicing):**
- Task 2.0: User can [do X] (data access + logic + UI for that slice)
- Task 3.0: User can [do Y] (data access + logic + UI for that slice)

### PR-Size Discipline

The plan targets **one vertical slice per pull request**. Before finalizing, add up the expected footprint across all tasks.

If the total exceeds what one human reviewer can hold in a single sitting, a soft guideline of roughly 400 changed lines excluding lockfiles, generated schema files, and snapshots, the work should be split into separate tickets **before implementation starts**, not planned as one oversized PR. Count behavior units and changed contracts more heavily than raw lines: a 600-line mechanical rename reviews easily; a 200-line change across four contracts does not.

The planner's job is to **flag** this, with the estimate and the suggested split points. Splitting the work is a human and tracker action, not something this phase performs.

### Atomic Commits

Every completed task is committed before the next one starts. This is a hard rule.

One task = one commit. If a task is partially done, do not commit. If a task fails, the rollback does not touch any previously committed task, so recovery is possible at any point without losing prior work.

### Role Assignment

Declare roles before generating tasks. The main agent is the orchestrator: it plans, delegates, sequences, and verifies. Subagents are the implementers: each receives one task, executes it, commits, and stops. **The orchestrator must not drift into direct implementation.** When you find yourself writing implementation logic in the main session, stop, create a task, and delegate it.

| Role | Responsibilities | Must NOT |
|---|---|---|
| Orchestrator (main agent) | Plan tasks, delegate, sequence, verify commits | Implement features directly |
| Implementer (subagent) | Implement one task, run its `Verify:`, commit, report | Take on adjacent tasks |

### Context Isolation for Task Execution

Each implementer subagent receives only:
1. The spec
2. The one task it is implementing, with its full field block
3. The `Leverage:` paths named in that task

Do NOT pass the full conversation history. Accumulated context from earlier tasks carries assumptions and patterns that contaminate downstream implementations. Fresh context per task is a hard rule, not a performance optimization.

### Quality Gates

Set up quality gates before the first implementation task: at minimum a type check, a linter, and the test runner. **Configure them as pre-commit hooks** so they fire on every commit attempt rather than living as reminders.

```
# pre-commit hook (tool-agnostic example)
your-typecheck-command && your-lint-command && your-test-command
```

When an implementer's commit is rejected by the hook, it sees the error immediately and self-corrects. Broken code is caught at the source instead of surfacing two tasks later. Each parent task leaves the codebase in a passing state before the next begins; if a parent task ends with a failing hook, fix it before proceeding.

### Final Task: Spec Verification

The last task is always "Verify all spec success criteria." It does three things: confirms that **every `SC-n` has passing verification evidence**, meaning a command that ran and passed, named against the criterion; traces every constraint and invariant through the code rather than only through tests; and runs the relevant test scope to confirm no regressions.

## Output Format

```markdown
# Implementation Plan: [Feature Name]

## Relevant Files

<!-- Spec, PRD, and research first -->
- `tasks/spec-[feature].md` - Behavioral spec (Status: Ready for Implementation). Read before writing any code.
- `tasks/prd-[feature].md` - Product requirements. Context for why this is being built.
- `tasks/research-[feature].md` - Research summary (if it exists).

- `path/to/file.ext` - Why this file is relevant.

## Footprint & PR Size

- Estimated changed lines: [n] (excluding lockfiles, generated schema, snapshots)
- Behavior units / contracts changed: [n]
- Verdict: [fits one PR] | [FLAG: exceeds one review. Suggested split: ...]

## Notes

- Quality gates (type check, linter, test runner) configured as pre-commit hooks before the first implementation task.
- One task = one commit. Never batch commits across tasks.
- Implementer subagents get fresh context: spec, one task, leverage paths. No conversation history.
- Definition of done: every `SC-n` in the spec has passing verification evidence.

## Instructions for Completing Tasks

**IMPORTANT:** As you complete each task, check it off by changing `- [ ]` to `- [x]`. Update after each sub-task, not only after the parent task.

## Tasks

- [ ] 0.0 Create feature branch
  - [ ] 0.1 Create and checkout a branch for this feature
    Depends: none
    Verify: <vcs-command showing the expected branch>

<!-- Include 1.0 ONLY if this change creates new cross-layer structural boundaries -->
- [ ] 1.0 Walking skeleton
  - [ ] 1.1 Create new files as stubs with correct signatures, no behavior
    Spec: [reference architecture section]
    Depends: 0.0
    Footprint: [path/to/file.ext]
    Verify: <build-or-typecheck-command>

- [ ] 2.0 [First vertical slice — user-visible outcome]
  - [ ] 2.1 [Outcome in one sentence]
    Spec: SC-1, SC-2
    Depends: 1.0
    Footprint: [path/to/file.ext], [path TBD — confirm before implementing]
    Leverage: [path/to/existing/module]
    Verify: <test-runner scoped to the touched code>
    Escalate if: [condition requiring a stop and a human decision]

- [ ] 3.0 [Second vertical slice — user-visible outcome]
  - [ ] 3.1 [Outcome in one sentence]
    Spec: SC-3, invariant "[name]"
    Depends: 2.0
    Footprint: [path/to/file.ext]
    Leverage: [path/to/existing/pattern]
    Verify: <test-runner scoped to the touched code>

- [ ] N.0 Verify all spec success criteria
  - [ ] N.1 Confirm every SC-n has passing verification evidence, named against the criterion
  - [ ] N.2 Trace every constraint and invariant through the code, not only through tests
  - [ ] N.3 Run the relevant test scope and confirm no regressions
```

## Final Instructions

1. Do NOT implement anything in this phase. This phase produces a plan.
2. Generate the full plan in one pass, with no confirmation pause between parent tasks and sub-tasks.
3. Save to `tasks/tasks-[feature].md` unless the workspace adapter specifies another location.
4. After saving, the next step is: **run `preflight-plan.md` in a FRESH context** before implementation begins. Do not roll into execution from this session.

## Target Audience

Assume the reader is a developer implementing the feature with AI assistance, and that individual tasks will be handed to implementer subagents with no other context. Each task must be self-sufficient: an implementer reading one task block should know what to build, what it must satisfy, where it lives, what to reuse, how to prove it, and when to stop and ask.
