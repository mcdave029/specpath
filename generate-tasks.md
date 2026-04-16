# Rule: Generating a Task List from a Spec

## Goal

To guide an AI assistant in creating a detailed, step-by-step task list in Markdown format based on a behavioral spec. The task list should guide a developer through implementation with atomic tasks, walking skeleton structure, vertical slices, and leverage information.

## Output

- **Format:** Markdown (`.md`)
- **Location:** `/tasks/`
- **Filename:** `tasks-[feature-name].md`

## Process

1. **Check for Spec:** Before generating tasks, check if `tasks/spec-[feature].md` exists.
   - **If spec exists:** Read both the spec and PRD fully. Tasks must reference spec sections and frame all implementation work as "satisfy these constraints and success criteria."
   - **If no spec exists:** STOP and warn the user: "No spec file found for this feature. For any feature where files affected > 5 or requirements are unclear, run `generate-spec.md` first. Skipping the spec means the AI will guess at constraints during implementation. Proceed without a spec only for trivial changes (single-file fixes, config tweaks, copy updates). Reply 'skip spec' to proceed anyway."
   - Wait for the user to confirm before continuing if no spec was found.
2. **Receive Requirements:** The user provides a feature request, task description, or points to an existing spec and PRD.
3. **Analyze Requirements:** Analyze the functional requirements, constraints, and implementation scope from the provided information.
4. **Phase 1: Generate Parent Tasks:** Create the file and generate the main, high-level tasks. **Always include task 0.0 "Create feature branch" as the first task** unless the user requests otherwise. **Always include task 1.0 "Walking skeleton"** as the first implementation task — stubs, interfaces, type definitions, empty modules with correct signatures, no behavior. When a spec exists, include a final task "Verify all spec success criteria." Present parent tasks to the user and say: "I have generated the high-level tasks. Ready to generate the sub-tasks? Respond with 'Go' to proceed."
5. **Wait for Confirmation:** Pause and wait for the user to respond with "Go".
6. **Phase 2: Generate Sub-Tasks:** Break down each parent task into smaller, actionable sub-tasks. When a spec exists, each sub-task must cite the relevant spec section (e.g., "see spec §3 Constraints"). Include leverage information pointing to existing code to reuse.
7. **Identify Relevant Files:** List files that will need to be created or modified, including the spec and PRD if they exist.
8. **Generate Final Output:** Combine parent tasks, sub-tasks, relevant files, and notes into the final Markdown structure.
9. **Save Task List:** Save to `/tasks/tasks-[feature-name].md`.

## Task Design Rules

### Walking Skeleton First

Task 1.0 is ALWAYS the walking skeleton: stubs, interfaces, type definitions, and empty modules with correct signatures. No behavior. No logic. This forces architectural thinking before implementation and gives all subsequent tasks a clear structure to fill in.

If you skip the walking skeleton, the AI will make architectural decisions while writing logic — the most expensive time to discover wrong choices.

**Stub patterns for walking skeleton tasks:**

| Pattern | When to use | Example |
|---|---|---|
| Return empty or hardcoded value | Data layer not yet implemented | `return []` or `return { id: 1 }` |
| Comment placeholder | Function wiring exists, logic deferred | `// TODO: implement in task 2.1` |
| Throw "not implemented" | Required interface method, must exist now | `throw new Error("not implemented — see task 2.2")` |
| `.skip` or `.todo` tests | Test wiring needed, behavior deferred | Confirms test structure without failing the suite |

The goal of the skeleton is to prove the wiring is correct before adding any logic. Every stub should be replaceable by a later task without touching surrounding code.

### Vertical Slices

Slice tasks by user-visible outcome, not by layer. Each task should deliver something observable end-to-end.

**Wrong (layer slicing):**
- Task 2.0: Build all data access
- Task 3.0: Build all business logic
- Task 4.0: Build all UI

**Right (vertical slicing):**
- Task 2.0: User can [do X] (includes data access + logic + UI for this slice)
- Task 3.0: User can [do Y] (includes data access + logic + UI for this slice)

### Task Sizing

Every sub-task must meet these criteria before it is considered atomic enough to delegate:

| Criterion | Target |
|---|---|
| File scope | 1–3 related files maximum |
| Time to complete | 15–30 minutes |
| Outcomes | One testable outcome per task |
| File paths | Exact files to create or modify must be named |
| Task type | Coding only — no deployment, no user testing, no documentation-only tasks |

**Title discipline:** Avoid vague words in task titles. If the title contains "system", "integration", "complete", or "setup", the task is almost certainly too broad — split it.

If a task exceeds these bounds, split it before presenting parent tasks to the user. An AI agent that receives an oversized task will make architectural decisions mid-implementation.

### Leverage Information

Every sub-task that touches existing code must include a Leverage line pointing to the relevant existing module, pattern, or utility. Each sub-task should also name the exact files it will create or modify:

```markdown
- [ ] 2.1 Implement [behavior — see spec §3 Constraints]
  Files: path/to/file-to-create.ext, path/to/file-to-modify.ext
  Leverage: path/to/existing/module.ext
```

The `Files:` field makes scope explicit before the agent starts. The `Leverage:` field prevents the AI from reinventing patterns that already exist. Both are required for any sub-task that touches code.

**Grounding rule:** Only include exact file paths if confirmed from the repo — through the spec, research summary, or a direct codebase read. If a path is unknown, write `[path TBD — confirm before implementing]`. Do not invent paths with false precision. Fabricated file paths are the most common cause of tasks that fail on the first attempt.

### Atomic Commits

Every completed task must be committed before moving to the next one. This is a hard rule, not a suggestion.

One task = one commit. If a task is partially done, do not commit. If a task fails, the rollback does not affect any previously committed task. This makes recovery possible at any point without losing all prior work.

### Role Assignment

Before generating any tasks, declare roles explicitly. The main agent is the orchestrator — its job is to plan, delegate, and verify. Subagents are the implementers — each receives one task, executes it, commits, and stops.

**The orchestrator must not drift into direct implementation.** When you find yourself writing implementation logic in the main session, stop. Create a task and delegate it.

Prompt pattern to establish roles at the start of task execution:
```
You are the orchestrator. Your subagents are the developers.
Your job: delegate each task to a subagent, verify the result, and sequence the work.
Do not implement directly unless no subagent capability is available.
```

Role boundaries:

| Role | Responsibilities | Must NOT |
|---|---|---|
| Orchestrator (main agent) | Plan tasks, delegate, sequence, verify commits | Implement features directly |
| Developer (subagent) | Implement one task, commit, report result | Take on adjacent tasks |

---

### Context Isolation for Task Execution

When delegating tasks to subagents, each subagent receives only:
1. The spec (`tasks/spec-[feature].md`)
2. The specific task it is implementing
3. The relevant leverage file paths

Do NOT pass the full conversation history. Accumulated context from earlier tasks introduces assumptions and patterns that contaminate downstream implementations. Fresh context per task is a hard rule, not a performance optimization.

### Quality Gates

Set up your quality gates before Task 2.0. At minimum: a type check, a linter, and your test runner. **Configure them as pre-commit hooks** so they fire automatically on every commit attempt — not just as reminders to run manually.

```
# pre-commit hook (tool-agnostic example)
your-typecheck-command && your-lint-command && your-test-command
```

When a subagent's commit is rejected by the hook, it sees the error output immediately and self-corrects before continuing. This removes the human from the inner loop. Broken code is caught at the source, not discovered two tasks later.

Each parent task should leave the codebase in a passing state before the next one starts. If a parent task ends with a failing hook, do not proceed to the next task — fix it first.

### Final Task: Spec Verification

When a spec exists, the final task is always "Verify all spec success criteria." The agent checks each Given/When/Then criterion in the spec §5 and confirms it is met — not by running tests, but by tracing the behavior through the implementation.

## Output Format

```markdown
# Tasks: [Feature Name]

## Relevant Files

<!-- If a spec was generated, always list it first -->
- `tasks/spec-[feature-name].md` - Behavioral spec — read before writing any code.
- `tasks/prd-[feature-name].md` - Product requirements — context for why this is being built.
- `tasks/research-[feature-name].md` - Research summary (if it exists).

- `path/to/file.ext` - Brief description of why this file is relevant.
- `path/to/another/file.ext` - Brief description.

### Notes

- Set up quality gates (type check, linter, test runner) before Task 2.0 and run them after every parent task.
- One task = one commit. Commit before moving to the next task. Never batch commits across tasks.
- When delegating to subagents: give each subagent fresh context — the spec, the task, and leverage paths only. No conversation history.
- **When a spec exists:** Definition of done = all spec §5 Success Criteria met, verified by the final task.

## Instructions for Completing Tasks

**IMPORTANT:** As you complete each task, check it off by changing `- [ ]` to `- [x]`. Update after completing each sub-task, not just after completing a parent task.

## Tasks

- [ ] 0.0 Create feature branch
  - [ ] 0.1 Create and checkout a new branch for this feature

- [ ] 1.0 Walking skeleton
  - [ ] 1.1 Create all new files with empty stubs and correct signatures — no behavior yet
  - [ ] 1.2 Define all interfaces, types, and module boundaries [see spec §1 Reference Architecture]
  - [ ] 1.3 Confirm the skeleton compiles / imports cleanly before adding any logic

- [ ] 2.0 [First vertical slice — user-visible outcome]
  - [ ] 2.1 [Sub-task — see spec §3 Constraints]
    Files: [path/to/file.ext]
    Leverage: [path/to/existing/module]
  - [ ] 2.2 [Sub-task — see spec §4 Phase 2]
    Files: [path/to/file.ext]
    Leverage: [path/to/existing/pattern]

- [ ] 3.0 [Second vertical slice — user-visible outcome]
  - [ ] 3.1 [Sub-task — see spec §3 Constraints]
    Files: [path/to/file.ext]
    Leverage: [path/to/existing/module]
  - [ ] 3.2 [Sub-task — see spec §4 Phase 2]
    Files: [path/to/file.ext]
    Leverage: [path/to/existing/pattern]

- [ ] N.0 Verify all spec success criteria
  - [ ] N.1 Trace each Given/When/Then in spec §5 through the implementation
  - [ ] N.2 Confirm all spec §3 Constraints are enforced — not just in tests, in the code
  - [ ] N.3 Run full test suite — confirm no regressions
```

## Interaction Model

The process explicitly requires a pause after generating parent tasks to get user confirmation ("Go") before proceeding to sub-tasks. This ensures the high-level plan aligns with user expectations before diving into details.

## Target Audience

Assume the primary reader of the task list is a developer implementing the feature with AI assistance. Tasks should be atomic enough that an AI agent can complete each one in a single focused session without needing to make architectural decisions mid-task.
