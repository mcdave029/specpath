# Rule: Spec Refinement Interview

## Goal

Surface every ambiguity in the spec that could cause implementation failure, at the cheapest possible moment — before a single line of code is written.

**The core prompt:**
> "Read this spec and ask me every question that could cause failure."

## Why This Phase Exists

The cost of ambiguity escalates sharply once implementation begins:

| Stage | Cost to fix |
|---|---|
| In the spec | 5 minutes |
| In this interview | 10 minutes |
| In code | 30 minutes |
| After first commit | 2-4 hours |
| In production | 8-16 hours |

One 10-minute interview eliminates entire categories of production incidents. Skipping it to save 10 minutes is the most expensive optimization in software development.

## When to Use This

Run this step after `generate-spec.md` and **before** `generate-tasks.md`. The spec must exist before the interview can begin.

## Process

1. **Receive the spec:** The user provides a path to `tasks/spec-[feature].md`.
2. **Read it completely.** Do not ask questions until you have read the entire spec. Questions about things the spec already answers clearly are noise and erode trust.
3. **Generate questions** grouped by the 5 ambiguity categories below.
4. **Ask questions.** Numbered list format, one group at a time. Wait for answers before moving to the next group.
5. **Update the spec inline** after each answer is received. Do not batch updates — update immediately so the user can see the spec converging.
6. **Confirm completion:** After all categories are addressed, say: "No remaining ambiguities. Spec is ready for task generation." Update the spec Status field to "Ready for Implementation."

## The 5 Ambiguity Categories

Work through all five. Do not skip a category because it feels covered — ask at least one question per category to confirm.

### 1. Data Decisions

Questions about the values the system will process:

- What happens when required data is missing, null, empty, or malformed?
- What are the boundary values? (minimum, maximum, zero, negative, maximum length)
- What validation rules apply and where are they enforced?
- If [specific field from the spec] is missing at runtime, what should happen?

### 2. Conflict Resolution

Questions about what happens when two valid rules apply simultaneously:

- When two valid rules produce conflicting outcomes, which wins?
- What is the priority order when multiple conditions are true simultaneously?
- If [condition A from spec] and [condition B from spec] are both true, what happens?

### 3. Pattern Selection

Questions about whether the chosen approach holds in all cases:

- The spec references [pattern from Section 1] — are there cases where it does not apply?
- Does any existing code in the codebase contradict the chosen pattern?
- If the reference implementation was built for [different context], which parts do not carry over?

### 4. Failure Recovery

Questions about what happens when things go wrong:

- What is the retry strategy? How many times? What backoff interval?
- What happens after all retries are exhausted?
- Is partial success acceptable, or is this all-or-nothing?
- Where are errors surfaced — to the caller, logged silently, both, or neither?
- What is the rollback or compensation strategy if [specific operation from spec] fails partway through?

### 5. Boundary Conditions

Questions about edge cases, scale, and scope:

- What happens at scale? (high volume, concurrent requests, large payloads)
- Where does this feature stop? What is explicitly out of scope that someone might assume is in scope?
- What adjacent features or systems could interfere with this one?
- What happens if this feature is called before [dependency] is ready?

## When to Stop

Stop asking questions when:

- All five categories have been addressed
- Further questions have become repetitive or clearly covered by existing answers
- Questions have drifted into trivial implementation details, not behavioral contracts

Do not stop early because the spec looks good. Work through all five categories.

## Spec Update Protocol

After each answer, update `tasks/spec-[feature].md` directly:

- If the answer resolves an Open Question, remove it from the Open Questions section
- If the answer adds a constraint, add it to Section 3 (Constraints)
- If the answer clarifies the implementation approach, update Section 4
- If the answer changes a success criterion, update Section 5
- Update the Status field last, only when all questions are resolved

## Final Instructions

1. **Do NOT generate tasks yet.** The task list is generated after the interview in `generate-tasks.md`.
2. **Read the entire spec before generating any questions.** Never ask a question the spec already answers clearly.
3. **Work through all five ambiguity categories** — do not skip one because it feels addressed.
4. **Update the spec inline after each answer.** Do not batch.
5. After all answers are incorporated and the spec is updated: update Status to "Ready for Implementation" and tell the user: "Spec refined and ready. Next step: run `generate-tasks.md`."
