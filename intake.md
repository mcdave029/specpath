# Rule: Ticket Intake and Risk Classification

## Goal

Establish, before any design or code, that the ticket is a sufficient statement of intent — and classify how much scrutiny the work deserves. Intake is cheap; everything downstream inherits its output. A ticket that cannot answer "what, for whom, and how we know it's done" produces a spec that guesses, and guesses compound.

Intake is ticket-first: the tracker ticket is the normal source of intent. A separate PRD is the exception, not a phase (see `create-prd.md`).

## When to Use This

At the start of every ticket that will change code, before brainstorming, research, or spec work. Read-only investigation does not need intake.

## Process

1. **Capture the anchors.** Record the ticket ID, the current base git SHA of the target repository, and the working branch/worktree. Everything downstream — evidence, reviews, approvals — binds to these anchors.
2. **Load the ground rules.** Read the repository's contributing guide and steering documents (`CLAUDE.md`, `AGENTS.md`, or equivalent). They are authoritative for repo-specific commands and conventions.
3. **Evaluate intent sufficiency.** Check the ticket against the sufficiency checklist below. The checklist tests whether intent is established, not whether the ticket is beautifully formatted.
4. **Repair or escalate.** Hygiene gaps (missing formatting, missing links, an absent estimate) are repaired in place and noted. Intent gaps — you cannot tell what behavior is wanted, for whom, or how success is judged — go to the human as targeted questions about the missing intent only. Never interview for ceremony, and never start substantive work on insufficient intent.
5. **Decide whether a PRD is needed.** Only when the ticket, even after clarification, does not sufficiently establish intent, scope, behavior, and success — then run `create-prd.md`. Never reproduce a good ticket into a second document.
6. **Classify risk.** Apply the risk rubric below. Record the tier and a one-paragraph rationale.
7. **Record the outcome.** Ticket status moves to in-progress; the intake result (sufficiency verdict, tier, rationale, anchors) is recorded wherever your workspace keeps workflow state, so a later session can resume without re-deriving it.

## Sufficiency Checklist

The generic contract — your workspace adapter maps these to its tracker's concrete ticket template:

| Question the ticket must answer | Gate |
|---|---|
| Who is this for and what do they get? (user story or equivalent) | **Blocker** — intent gap, ask the human |
| What testable conditions make it done? (acceptance criteria, binary pass/fail) | **Blocker** — intent gap, ask the human |
| What is broken/missing today vs expected after? (problem statement) | **Blocker** — intent gap, ask the human |
| How is success measured from the stakeholder's view? (success metric) | **Blocker** — intent gap, ask the human |
| How big is this expected to be? (estimate) | Repair — propose one, human confirms |
| Where did the request come from? (provenance) | Repair — backfill from known source |
| What is explicitly out of scope? | Repair — propose boundaries from context |
| What does it depend on? | Repair — link known dependencies |

A "Blocker" verdict means: ask the human the specific missing question(s), then re-check. A "Repair" verdict means: fix it yourself per the tracker's conventions and note that you did.

## Risk Rubric

Three tiers. **Uncertainty rounds UP.** The tier is re-evaluated at plan preflight (actual plan and footprint) and again at final-diff verification; it may stay or escalate, never silently drop.

| Tier | Criteria |
|---|---|
| **HIGH** | Touches protected paths (the per-repository list your workspace adapter maintains — typically: money movement, ledgers, payments, authentication/authorization, deploy and CI workflow files, infrastructure-as-code and IAM, signing keys and cryptography, personal-data handling, and the agent control plane itself). Destructive or risky database changes: renames, drops, non-null conversions, large backfills, locking risk. New external integration. Irreversible data operations. Changes to a durable contract (API shape, event format, schema) consumed by another system. Changes to personal-data handling or data lifecycle. |
| **LOW** | Documentation-only, copy changes, test-only additions, single-file changes off protected paths with deterministic verification. Dependency patch bumps only when they arrive through a verification path your workspace trusts (existing dependency, no new packages in the lockfile delta, no protected subsystem touched, scans green) — otherwise STANDARD. |
| **STANDARD** | Everything else, including additive backward-compatible no-backfill migrations (nullable columns, safe indexes), which carry enhanced migration verification downstream. |

What the tier buys:

- **LOW** — the pipeline flows without human touch until the pull-request review.
- **STANDARD** — the spec is posted/surfaced when Ready; the human may object at any time; work proceeds.
- **HIGH** — explicit human approval of the spec before implementation begins. Consider having your strongest reasoning configuration drive the design session.

## Output

Intake produces no repository artifact of its own. It produces a recorded decision: sufficiency verdict, risk tier + rationale, base SHA, and ticket status — stored in your workspace's workflow state so downstream phases (and future sessions) read it instead of re-deriving it.

## Final Instructions

1. Do NOT begin design, research, or implementation inside intake.
2. Ask the human only about missing **intent**; repair hygiene yourself.
3. Record the tier with its rationale — a bare "STANDARD" with no reasoning cannot be re-evaluated honestly later.
4. Next step: brainstorm/design the change (invoking `research.md` whenever design hits a question evidence can answer), then `generate-spec.md`.
