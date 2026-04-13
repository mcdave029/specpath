# Rule: Generating a Task List from User Requirements

## Goal

To guide an AI assistant in creating a detailed, step-by-step task list in Markdown format based on user requirements, feature requests, or existing documentation. The task list should guide a developer through implementation.

## Output

- **Format:** Markdown (`.md`)
- **Location:** `/tasks/`
- **Filename:** `tasks-[feature-name].md` (e.g., `tasks-user-profile-editing.md`)

## Process

1.  **Check for Spec:** Before generating tasks, check if `tasks/spec-[feature].md` exists.
    - **If spec exists:** Read both `spec-[feature].md` and the test skeleton(s) fully. Tasks must reference spec sections and frame all implementation work as "make these tests pass."
    - **If no spec exists:** STOP and warn the user: "No spec file found for this feature. For any feature that touches an API, changes a data model, or has non-obvious behavior, run `generate-spec.md` first. Skipping the spec means the AI will guess at behavioral contracts during implementation. Proceed without a spec only for trivial changes (config tweaks, copy updates, single-line fixes). Reply 'skip spec' to proceed anyway."
    - Wait for the user to confirm before continuing if no spec was found.
2.  **Receive Requirements:** The user provides a feature request, task description, or points to existing documentation
3.  **Analyze Requirements:** The AI analyzes the functional requirements, user needs, and implementation scope from the provided information
4.  **Phase 1: Generate Parent Tasks:** Based on the requirements analysis, create the file and generate the main, high-level tasks required to implement the feature. **IMPORTANT: Always include task 0.0 "Create feature branch" as the first task, unless the user specifically requests not to create a branch.** When a spec exists, always include task 1.0 "Read spec and confirm tests RED" and a final task "All spec tests green." Use your judgement on how many additional high-level tasks to use. It's likely to be about 5. Present these tasks to the user in the specified format (without sub-tasks yet). Inform the user: "I have generated the high-level tasks based on your requirements. Ready to generate the sub-tasks? Respond with 'Go' to proceed."
5.  **Wait for Confirmation:** Pause and wait for the user to respond with "Go".
6.  **Phase 2: Generate Sub-Tasks:** Once the user confirms, break down each parent task into smaller, actionable sub-tasks necessary to complete the parent task. Ensure sub-tasks logically follow from the parent task and cover the implementation details implied by the requirements. When a spec exists, sub-tasks must cite the relevant spec section (e.g., "see spec §API Contracts").
7.  **Identify Relevant Files:** Based on the tasks and requirements, identify potential files that will need to be created or modified. List these under the `Relevant Files` section, including spec files and test skeletons if they exist.
8.  **Generate Final Output:** Combine the parent tasks, sub-tasks, relevant files, and notes into the final Markdown structure.
9.  **Save Task List:** Save the generated document in the `/tasks/` directory with the filename `tasks-[feature-name].md`, where `[feature-name]` describes the main feature or task being implemented (e.g., if the request was about user profile editing, the output is `tasks-user-profile-editing.md`).

## Output Format

The generated task list _must_ follow this structure:

```markdown
## Relevant Files

<!-- If a spec was generated, always list it first -->
- `tasks/spec-[feature-name].md` - Behavioral contracts — read before writing any code.
- `tasks/spec-[feature-name].test.ts` - Executable spec (TypeScript) — all tests must be GREEN when done.
- `spec/[domain]/[feature-name]_spec.rb` - Executable spec (Rails) — all examples must pass when done.

- `path/to/potential/file1.ts` - Brief description of why this file is relevant (e.g., Contains the main component for this feature).
- `path/to/file1.test.ts` - Unit tests for `file1.ts`.
- `path/to/another/file.tsx` - Brief description (e.g., API route handler for data submission).
- `path/to/another/file.test.tsx` - Unit tests for `another/file.tsx`.
- `lib/utils/helpers.ts` - Brief description (e.g., Utility functions needed for calculations).
- `lib/utils/helpers.test.ts` - Unit tests for `helpers.ts`.

### Notes

- Unit tests should typically be placed alongside the code files they are testing (e.g., `MyComponent.tsx` and `MyComponent.test.tsx` in the same directory).
- Use `npx jest [optional/path/to/test/file]` to run tests. Running without a path executes all tests found by the Jest configuration.
- Use `bundle exec rspec spec/[path]` to run RSpec examples.
- **When a spec exists:** Definition of done = all spec tests GREEN and no regressions in the full suite.

## Instructions for Completing Tasks

**IMPORTANT:** As you complete each task, you must check it off in this markdown file by changing `- [ ]` to `- [x]`. This helps track progress and ensures you don't skip any steps.

Example:
- `- [ ] 1.1 Read file` → `- [x] 1.1 Read file` (after completing)

Update the file after completing each sub-task, not just after completing an entire parent task.

## Tasks

- [ ] 0.0 Create feature branch
  - [ ] 0.1 Create and checkout a new branch for this feature (e.g., `git checkout -b feature/[feature-name]`)

<!-- Include tasks 1.0 and N.0 below when a spec file exists -->
- [ ] 1.0 Read spec and confirm tests RED
  - [ ] 1.1 Read `tasks/spec-[feature-name].md` fully before writing any code
  - [ ] 1.2 Run the test skeleton — confirm every test/example is RED (none passing yet)
- [ ] 2.0 Parent Task Title
  - [ ] 2.1 [Sub-task description — see spec §Section Name]
  - [ ] 2.2 [Sub-task description — see spec §Section Name]
- [ ] 3.0 Parent Task Title
  - [ ] 3.1 [Sub-task description]
- [ ] 4.0 Parent Task Title (may not require sub-tasks if purely structural or configuration)
- [ ] N.0 All spec tests green
  - [ ] N.1 Run full test suite — confirm no regressions
  - [ ] N.2 Run spec skeleton — 0 pending, 0 failing, 0 RED
```

## Interaction Model

The process explicitly requires a pause after generating parent tasks to get user confirmation ("Go") before proceeding to generate the detailed sub-tasks. This ensures the high-level plan aligns with user expectations before diving into details.

## Target Audience

Assume the primary reader of the task list is a **junior developer** who will implement the feature.
