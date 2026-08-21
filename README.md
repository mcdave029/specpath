# specpath

A Spec-Driven Development workflow toolkit for AI-assisted engineering.

Works with any AI coding assistant: Claude Code, Amp, Windsurf, Cursor, Copilot, or any tool that accepts markdown prompts.

---

## The Problem

Vibe coding degrades at scale.

When you describe a feature and ask an AI to build it, the first response looks great. The second looks good. By the fifth, you are debugging hallucinated APIs and fixing patterns the AI invented from context drift.

This is not an AI capability problem. It is an information problem. The AI cannot know what it does not know: your architecture, your constraints, your existing patterns. Without that context, it guesses. Guesses compound. Output degrades.

**The Vibe Dip is not random. It is mathematically guaranteed when requirements are ambiguous and context is implicit.**

Two failure modes drive it:

**Agent Amnesia** — each session starts from zero. Without external memory, context discovered in one session is lost when it ends. The AI re-discovers — or re-invents — the same constraints every time.

**Context Pollution** — within a session, failed attempts and debugging artifacts fill the context window. When space runs out, the agent starts dropping previously discovered bugs and constraints to make room. Output degrades from the inside.

The spec eliminates both. It is the external memory that survives sessions, and the clean source of truth that each subagent reads instead of the polluted conversation history.

---

## The Solution

Context persistence + explicit constraints + fresh-context review = reliable output.

Specs survive session restarts. Constraints eliminate guessing. Fresh-context interrogation and preflight catch problems at the cheapest possible moment — and they catch the problems the authoring context cannot see in its own work.

The spec is also a recovery mechanism. If a session derails, context fills, or implementation drifts from intent — open a new session, pin the spec, and resume from the last committed task. No context rebuilding required.

The cost of ambiguity escalates sharply once work begins:

| Stage | Cost to fix |
|---|---|
| In the spec | 5 minutes |
| In interrogation / preflight | 10 minutes |
| In code | 30 minutes |
| After first commit | 2-4 hours |
| In production | 8-16 hours |

---

## Steering Documents

Before running the pipeline on any feature, make sure your project has steering documents. These are **per-repository, committed files** — they live in the repo root or a subdirectory, travel with the code, and are available to every developer, every AI tool, and every session that clones the project.

They are the project constitution. Every spec and every task reads them before doing anything. They answer the questions the AI would otherwise guess at.

Three standard files:

| File | What it captures |
|---|---|
| `product.md` | What the product is, who it is for, what problems it solves |
| `tech.md` | Tech stack, architectural patterns, constraints, what NOT to use |
| `structure.md` | Project layout, naming conventions, file organization rules |

If your project already has a `CLAUDE.md`, `AGENTS.md`, or `GEMINI.md`, those serve the same purpose — you do not need separate steering files. The content matters more than the filename. Keep them short, update them when the project changes, commit every update.

---

## The Pipeline (v3)

specpath is **ticket-first**: the tracker ticket is the normal source of intent, and depth of process is decided by **risk tier**, not by ceremony.

| File | Role | When it runs |
|---|---|---|
| `intake.md` | Ticket sufficiency check + risk classification (LOW / STANDARD / HIGH) | Every ticket, first |
| `create-prd.md` | Product Requirements Document | **Optional** — only when the ticket cannot establish intent |
| `research.md` | Parallel evidence gathering with tagged findings | **On demand** — whenever any phase hits a question evidence can answer |
| `generate-spec.md` | The engineering spec: delta-oriented contract with invariants, constraints, data scope, verification intent | Every non-trivial change |
| `interrogate-spec.md` | Fresh-context interrogation — a context that did not author the spec tries to break it | After the spec; sets `Ready for Implementation` through the tier gate |
| `generate-tasks.md` | One-pass implementation plan — per-task outcomes, dependencies, footprint, leverage, verification commands | After the spec is Ready |
| `preflight-plan.md` | Fresh-context plan review — feasibility, coverage, boundaries, scope | After the plan, before implementation |

### Risk tiers decide the human touchpoints

| Tier | Spec gate |
|---|---|
| **LOW** | Flows to implementation without an intermediate human gate |
| **STANDARD** | Spec is posted/surfaced when Ready; humans may object at any time |
| **HIGH** (protected paths, destructive data changes, new integrations, durable contracts, sensitive data) | Explicit human spec approval before implementation |

Uncertainty rounds UP, and the tier is re-evaluated at preflight and again against the final diff — it can escalate, never silently drop. The concrete rubric (protected paths, what counts as sensitive) is supplied per workspace; `intake.md` carries the generic criteria.

### What changed from v2

- **Ticket-first intake** replaces "PRD always". A good ticket is not reproduced into a second document.
- **Research is an engine, not a phase.** Invoke it the moment design, spec, interrogation, or planning hits an evidence-answerable question.
- **Fresh-context interrogation replaces the human interview.** Evidence questions get researched, mechanical ambiguities get fixed, and only material product/architecture/security judgment reaches the human. (This is the adversarial `review-spec.md` the old README proposed — built, and extended.)
- **The spec is an engineering contract**: delta-oriented, with invariants, implementation boundaries, data scope, verification intent — and durable contracts (API shapes, event formats, schemas) are *included* when they are the architectural decision.
- **One-pass plan generation** — no mid-generation confirmation pause; a fresh-context preflight reviews the plan instead.
- **Walking skeleton is conditional** — only when the change creates new cross-layer structural boundaries.

---

## How to Use

Clone the files to a location your AI tool can access:

```bash
git clone https://github.com/mcdave029/specpath.git
```

Then invoke each phase by pointing your AI at the relevant file:

```
Use @intake.md
Ticket: [id / paste the ticket]
```

```
Use @generate-spec.md
Ticket: [id]  Risk tier: [from intake]
Research: @tasks/research-[feature].md   (if any)
```

```
Use @interrogate-spec.md          <- run in a FRESH context
Spec: @tasks/spec-[feature].md
```

```
Use @generate-tasks.md
Spec: @tasks/spec-[feature].md
```

```
Use @preflight-plan.md            <- run in a FRESH context
Plan: @tasks/tasks-[feature].md
Spec: @tasks/spec-[feature].md
```

Fresh context means a new session or an isolated subagent that receives the artifact and repository access — never the conversation that produced the artifact.

---

## Levels of Commitment

| Level | Name | What it means | When to use |
|---|---|---|---|
| 1 | Spec-First | Spec is a planning artifact, discarded after the build | Prototypes, personal projects |
| 2 | Spec-Anchored | Spec is kept as a **dated decision record** — like an ADR, it captures what was decided and goes stale by design | Team projects, long-lived systems |
| 3 | Spec-as-Source | Code is regenerated from the spec on demand | Experimental — not recommended |

Default: Level 1 for throwaway work, Level 2 for anything with a lifespan. A Level 2 spec is not maintained to mirror the code — it is superseded by the next ticket's spec, the way ADRs supersede each other.

---

## Credits

Phase 0 (`create-prd.md`) and the original task-generation prompt are based on [snarktank/ai-dev-tasks](https://github.com/snarktank/ai-dev-tasks).

SDD methodology informed by the [Panaversity Agent Factory curriculum](https://agentfactory.panaversity.org) and the broader spec-driven development community.

---

## Contributing

Open an issue or submit a pull request. The goal is a toolkit that works across stacks, teams, and project sizes without requiring customization — workspace-specific bindings (output paths, tracker wiring, risk rubrics) belong in thin per-workspace adapters, not in these files.
