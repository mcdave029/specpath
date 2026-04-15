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

### Vertical Slices

Slice tasks by user-visible outcome, not by layer. Each task should deliver something observable end-to-end.

**Wrong (layer slicing):**
- Task 2.0: Build all data access
- Task 3.0: Build all business logic
- Task 4.0: Build all UI

**Right (vertical slicing):**
- Task 2.0: User can [do X] (includes data access + logic + UI for this slice)
- Task 3.0: User can [do Y] (includes data access + logic + UI for this slice)

### Leverage Information

Every sub-task that touches existing code must include a Leverage line pointing to the relevant existing module, pattern, or utility:

```markdown
- [ ] 2.1 Implement [behavior — see spec §3 Constraints]
  Leverage: path/to/existing/module.ext
```

This prevents the AI from reinventing patterns that already exist.

### Final Task: Spec Verification

When a spec exists, the final task is always "Verify all spec success criteria." The agent checks each Given/When/Then criterion in the spec §5 and confirms it is met — not by running tests, but by tracing the behavior through the implementation.

## Output Format

```markdown
## Relevant Files

<!-- If a spec was generated, always list it first -->
- `tasks/spec-[feature-name].md` - Behavioral spec — read before writing any code.
- `tasks/prd-[feature-name].md` - Product requirements — context for why this is being built.
- `tasks/research-[feature-name].md` - Research summary (if it exists).

- `path/to/file.ext` - Brief description of why this file is relevant.
- `path/to/another/file.ext` - Brief description.

### Notes

- Run your test suite after each parent task to catch regressions early.
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
    Leverage: [path/to/existing/module]
  - [ ] 2.2 [Sub-task — see spec §4 Phase 2]
    Leverage: [path/to/existing/pattern]

- [ ] 3.0 [Second vertical slice — user-visible outcome]
  - [ ] 3.1 [Sub-task]
  - [ ] 3.2 [Sub-task]

- [ ] N.0 Verify all spec success criteria
  - [ ] N.1 Trace each Given/When/Then in spec §5 through the implementation
  - [ ] N.2 Confirm all spec §3 Constraints are enforced — not just in tests, in the code
  - [ ] N.3 Run full test suite — confirm no regressions
```

## Interaction Model

The process explicitly requires a pause after generating parent tasks to get user confirmation ("Go") before proceeding to sub-tasks. This ensures the high-level plan aligns with user expectations before diving into details.

## Target Audience

Assume the primary reader of the task list is a developer implementing the feature with AI assistance. Tasks should be atomic enough that an AI agent can complete each one in a single focused session without needing to make architectural decisions mid-task.
