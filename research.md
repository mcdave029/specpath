# Rule: Parallel Research for Feature Development

## Goal

Compress hours of sequential investigation into minutes. Produce a research summary that directly feeds the spec, so the AI generating the spec has zero information gaps to guess at.

## When to Use This

Run this step after `create-prd.md` and **before** `generate-spec.md` when:

- Files affected > 5
- Requirements are unclear or involve unfamiliar territory
- The feature touches existing systems you need to understand before designing

Skip this step for single-file changes or features in well-understood domains.

## Process

1. **Check for steering documents:** Before decomposing into threads, check if project constitution files exist (`CLAUDE.md`, `AGENTS.md`, `product.md`, `tech.md`, `structure.md`). If they exist, read them — they may answer questions that would otherwise require a full research thread, and they define the architecture the spec must respect. If none exist, note it in the synthesis: architectural assumptions in this research are inferred from codebase evidence only.
2. **Receive the PRD:** The user provides a path to `tasks/prd-[feature].md` or describes the feature directly.
3. **Decompose into research threads:** Identify 3–5 independent questions the research must answer. See thread types below.
4. **Spawn parallel subagents:** One subagent per thread. Each subagent starts with fresh context — no shared state, no conversation history passed between threads.
5. **Collect findings:** Each subagent produces a focused written summary of its thread.
6. **Synthesize:** Merge all summaries into `tasks/research-[feature].md`. Explicitly surface conflicts and contradictions between threads.
7. **Present to user:** Show the synthesis before proceeding. Wait for confirmation that it is accurate before moving to `generate-spec.md`.

## Thread Decomposition Criteria

Good research threads are:

- **Independent:** No thread depends on another thread's findings to proceed.
- **Focused:** One answerable question per thread, not broad exploration.
- **Bounded:** Clear scope with a defined stopping point.
- **Complementary:** Together, the threads cover the full problem space.

A thread that requires reading another thread's output first is not independent — split or merge it.

## Standard Thread Types

Use whichever combination covers the feature. Not all threads are needed for every feature.

| Thread | Question it answers |
|---|---|
| Current codebase | What exists today that this feature will touch or extend? |
| Reference implementations | How is this type of problem solved in the ecosystem? |
| Design patterns | Which established patterns apply here? |
| Documentation | What specs, RFCs, or library docs constrain the approach? |
| Integration points | How does this interact with adjacent systems? |

For most features, 3 threads are sufficient. Add a fourth or fifth only when the problem space clearly has a dimension not covered above.

## Subagent Instructions

When spawning each subagent, provide:

1. The specific question this thread must answer
2. The relevant context (PRD excerpt, file paths to read, systems to investigate)
3. A clear stopping condition: "Stop when you can answer [specific question]"
4. Output format: one written summary, findings only, no implementation suggestions

Do NOT:
- Pass conversation history from one thread to another
- Ask a subagent to generate spec content
- Ask a subagent to suggest implementation approaches

The subagent's job is to discover and report, not to design.

## Output Format

Save as `tasks/research-[feature].md`:

```markdown
# Research: [Feature Name]

**PRD:** tasks/prd-[feature].md
**Date:** [date]

---

## Thread 1: [Thread Name]

**Question:** [The specific question this thread investigated]

[Findings — factual, cited where possible, no implementation suggestions]

---

## Thread 2: [Thread Name]

**Question:** [The specific question this thread investigated]

[Findings]

---

## Thread 3: [Thread Name]

**Question:** [The specific question this thread investigated]

[Findings]

---

## Synthesis

### Key decisions implied by research

[What the research clearly points to — patterns to follow, systems to use, approaches to avoid]

### Conflicts and contradictions

[Anywhere two threads produced conflicting findings. Do not resolve here — surface for the spec phase.]

### Recommended approach

[One paragraph. What the research suggests as the starting point for the spec. Not a design — a direction.]

### Evidence quality

[For each key decision above, note whether findings are:
- **Confirmed** — directly observed in source (codebase, docs, reference implementation)
- **Inferred** — logical conclusion from observed evidence, not directly read
- **Unresolved** — could not be determined

Do not present Inferred findings as Confirmed in the synthesis or carry them into the spec as facts.]

---

## Open Questions

[Anything the research could not answer. These become inputs for spec clarifying questions.]
```

## Deepening Research

If the initial synthesis is thin, the Recommended Approach is vague, or Open Questions outnumber resolved findings, run a second pass before moving to the spec.

Do NOT prompt "research longer" or "improve the plan" — open-ended expansion produces hallucinated findings dressed as research.

Instead, run a bounded second pass:

1. **Identify the weak thread** — name it specifically (e.g., "Thread 2: Reference Implementations did not find how Jazz handles conflict resolution")
2. **Define the exact unanswered question** — one question, not a theme
3. **Spawn a targeted follow-up subagent** with fresh context, the specific question, and a clear stopping condition: "Stop when you can answer [exact question]"
4. **Update the research artifact** — add findings to the relevant thread's section
5. **Surface remaining uncertainty** — if the question still cannot be answered after the follow-up, add it to Open Questions; do not fill the gap with inference

A focused second pass on one weak thread costs 5 minutes and is safer than a broad re-run that invites fabricated findings.

---

## Fallback: No Parallel Agent Capability

If your AI tool cannot spawn parallel subagents, run the threads sequentially instead:

1. Complete Thread 1 fully before starting Thread 2
2. Give each thread a fresh context window — do not carry findings from one thread into the next conversation
3. Collect all findings, then write the synthesis manually

The parallelism benefit is reduced but the structured research output still significantly improves spec quality over unguided exploration.

## Final Instructions

1. **Do NOT write a spec.** Research only. The spec is generated in the next phase.
2. **Do NOT suggest implementation code.** Findings only.
3. Each subagent must have fresh context — do not share conversation history between threads.
4. **Explicitly surface conflicts** between threads in the Synthesis section. Unresolved conflicts that are buried will cause problems in the spec.
5. Present the synthesis to the user and wait for confirmation before proceeding.
6. After saving: "Research saved to `tasks/research-[feature].md`. Next step: run `generate-spec.md` and point it at both the PRD and this research file."
