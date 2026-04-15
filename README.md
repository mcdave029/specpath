# specpath

A Spec-Driven Development workflow toolkit for AI-assisted engineering.

Works with any AI coding assistant: Claude Code, Amp, Windsurf, Cursor, Copilot, or any tool that accepts markdown prompts.

---

## The Problem

Vibe coding degrades at scale.

When you describe a feature and ask an AI to build it, the first response looks great. The second looks good. By the fifth, you are debugging hallucinated APIs and fixing patterns the AI invented from context drift.

This is not an AI capability problem. It is an information problem. The AI cannot know what it does not know: your architecture, your constraints, your existing patterns. Without that context, it guesses. Guesses compound. Output degrades.

**The Vibe Dip is not random. It is mathematically guaranteed when requirements are ambiguous and context is implicit.**

---

## The Solution

Context persistence + explicit constraints = reliable output.

Specs survive session restarts. Constraints eliminate guessing. Phase gates catch problems at the cheapest possible moment.

The cost of ambiguity escalates sharply once work begins:

| Stage | Cost to fix |
|---|---|
| In the spec | 5 minutes |
| In the interview | 10 minutes |
| In code | 30 minutes |
| After first commit | 2-4 hours |
| In production | 8-16 hours |

One 10-minute spec review eliminates entire categories of production incidents.

---

## Steering Documents

Before running the pipeline on any feature, make sure your project has steering documents. These are persistent, project-level context files that every spec and every task reads before doing anything. They are the project constitution — they answer the questions the AI would otherwise guess at.

Three standard files:

| File | What it captures |
|---|---|
| `product.md` | What the product is, who it is for, what problems it solves |
| `tech.md` | Tech stack, architectural patterns, constraints, what NOT to use |
| `structure.md` | Project layout, naming conventions, file organization rules |

Place them somewhere your AI tool loads automatically — `.claude/steering/` for Claude Code, `CLAUDE.md` for a single flat file, or wherever your tool looks for persistent context.

Keep them short. Update them when the project changes. Every spec you write will be better for it because the AI enters the conversation already knowing the architecture and constraints it must work within.

If your project already has a `CLAUDE.md`, `AGENTS.md`, or `GEMINI.md`, those serve the same purpose — you do not need separate steering files.

---

## Three Levels of Commitment

Not every feature needs the same investment. Pick your level before starting.

| Level | Name | What it means | When to use |
|---|---|---|---|
| 1 | Spec-First | Spec is a planning artifact, discarded after the build | Prototypes, personal projects, single-session work |
| 2 | Spec-Anchored | Spec and code are both maintained as living artifacts that evolve together | Team projects, systems with a 6+ month lifespan |
| 3 | Spec-as-Source | Code is regenerated from the spec on demand | Experimental — identical specs do not produce identical code; git diffs become unpredictable |

**Default:** Start at Level 1. Graduate to Level 2 when you are collaborating with a team or maintaining a system long-term. Level 3 is a frontier pattern — understand the tradeoffs before committing to it.

The pipeline below applies to all three levels. What changes is whether the spec is kept and maintained after implementation.

---

## The Pipeline

| Phase | File | Output | When to use |
|---|---|---|---|
| 0 | `create-prd.md` | `prd-[feature].md` | Always, before any spec work |
| 1 | `research.md` | `research-[feature].md` | Files affected > 5 OR requirements unclear |
| 2 | `generate-spec.md` | `spec-[feature].md` | After PRD (and research if Phase 1 ran) |
| 3 | `interview-spec.md` | Updated spec | After spec is drafted |
| 4 | `generate-tasks.md` | `tasks-[feature].md` | After interview confirms spec is complete |

### Decision Framework

```
files affected > 5  OR  requirements unclear  →  full pipeline (all 5 phases)
borderline case                                →  lightweight spec + phases 3-4
single file / incident / prototype             →  skip to Phase 4 directly
```

When in doubt, run the spec. The overhead is 15 minutes. The downside of skipping it can be days.

---

## Files

### `create-prd.md` — Phase 0

Guides the AI through clarifying questions and produces a scored Product Requirements Document. The PRD is the product layer: what you are building and why. It is written for humans and stakeholders.

**Output:** `tasks/prd-[feature].md`

**When:** Always. Even for internal tools and small features. A PRD forces you to articulate the problem before you start solving it.

---

### `research.md` — Phase 1

Compresses hours of investigation into minutes using parallel subagents with context isolation. Each subagent gets a focused research thread (current codebase, reference implementations, design patterns, relevant docs, integration points). Results are synthesized into a single research summary.

**Output:** `tasks/research-[feature].md`

**When:** Files affected > 5 OR requirements are unclear. Skip for single-file changes where the domain is well understood.

---

### `generate-spec.md` — Phase 2

Translates the PRD (and research summary, if available) into a precise behavioral specification. The spec is the technical layer: what the system must and must not do, written for the AI implementing it.

The spec has five sections:
1. Reference Architecture — the pattern to follow
2. Current Architecture — what exists today
3. Constraints — what must NOT happen (the most important section)
4. Implementation Plan — phased approach
5. Success Criteria — Given/When/Then acceptance tests

**Constraints matter more than requirements.** "Do NOT pre-fetch more than 3 items" eliminates wrong implementations. "Make it fast" does not.

**Output:** `tasks/spec-[feature].md`

**When:** After PRD. Always before tasks for non-trivial features.

---

### `interview-spec.md` — Phase 3

The AI reads the spec and asks every question that could cause implementation failure. This is the refinement phase: surface ambiguities before a single line of code is written.

Five ambiguity categories the AI will probe:
1. Data Decisions — boundary values, null handling, validation rules
2. Conflict Resolution — which rule wins when two apply
3. Pattern Selection — edge cases where the spec's chosen pattern does not apply
4. Failure Recovery — retry strategy, partial success, error surfacing
5. Boundary Conditions — scale, scope, adjacent system interference

Each answer updates the spec inline. The interview ends when all five categories are addressed and the spec Status is updated to "Ready for Implementation."

**Output:** Refined `tasks/spec-[feature].md`

**When:** After spec is drafted. Before tasks are generated. Non-negotiable.

---

### `generate-tasks.md` — Phase 4

Breaks the spec into an ordered, atomic task list. Three structural rules:

**Walking skeleton first.** Task 1.0 is always stubs, interfaces, type definitions, and empty modules with correct signatures. No behavior. This forces architectural thinking before logic.

**Vertical slices.** Each task delivers a user-visible outcome end-to-end. Not "backend work" followed by "frontend work" for the same feature.

**Leverage information.** Every task references existing code to reuse:
```
- [ ] 2.1 Implement [behavior]
  Spec: §3 Constraints, §4 Phase 1
  Leverage: path/to/existing/module
```

The final task is always "Verify all spec success criteria" — the AI checks each Given/When/Then in the spec and confirms it is met.

**Output:** `tasks/tasks-[feature].md`

**When:** After the interview confirms the spec is complete.

---

## How to Use

Clone the files to a location your AI tool can access:

```bash
git clone https://github.com/mcdave029/specpath.git
```

Then invoke each phase by pointing your AI at the relevant file:

```
Use @create-prd.md
Here is the feature I want to build: [describe it]
```

```
Use @research.md
PRD: @tasks/prd-[feature].md
```

```
Use @generate-spec.md
PRD: @tasks/prd-[feature].md
Research: @tasks/research-[feature].md
```

```
Use @interview-spec.md
Spec: @tasks/spec-[feature].md
```

```
Use @generate-tasks.md
Spec: @tasks/spec-[feature].md
```

---

## Credits

Phase 0 (`create-prd.md`) and the original Phase 4 (`generate-tasks.md`) are based on [snarktank/ai-dev-tasks](https://github.com/snarktank/ai-dev-tasks).

SDD methodology informed by the [Panaversity Agent Factory curriculum](https://agentfactory.panaversity.org) and the broader spec-driven development community.

---

## Contributing

Open an issue or submit a pull request. The goal is a toolkit that works across stacks, teams, and project sizes without requiring customization.
