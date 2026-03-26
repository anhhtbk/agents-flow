# Creation Log: khuym:dream

## Source Material

Origin:
- Dream behavior adaptation from `DREAM_FEATURE_SPEC.md`
- Khuym process contracts from `AGENTS.md`
- Locked decisions from `history/dream-skill/CONTEXT.md` (D1-D6)

What the source does:
- Defines a manual, repo-scoped dream consolidation pass over Codex artifacts and learnings files.
- Constrains consolidation with safety gates around ownership, ambiguity handling, and critical promotions.

Khuym context:
- Supports the dream-skill epic `br-2z8`.
- This bead (`br-13f`) is RED baseline only and intentionally does not finalize `SKILL.md`.

## Extraction Decisions

What to include:
- Bootstrap-vs-recurring risk tied to `last_dream_consolidated_at` because D2 requires correct first-run behavior.
- Exact-one-owner rewrite guard because validation spike requires rewrite only when ownership is clear.
- Candidate-specific ambiguity interaction because D5 requires explicit candidate files, reasons, and options.
- Critical-pattern promotion gating because D4 forbids direct edits without explicit user approval.

What to leave out:
- Full workflow wording for GREEN/REFACTOR because this bead is RED baseline setup only.
- Implementation-level heuristics for matching because those belong in later contract authoring.

## Structure Decisions

1. RED baseline artifacts are split into scenario catalog (`references/pressure-scenarios.md`) and this log to keep pressure cases reusable in later phases.
2. Every scenario records `Combined pressures` and `Exact rationalization` so future GREEN drafting can target observed failure language directly.
3. Added one combined-pressure scenario where multiple locked decisions can fail together; this prevents false confidence from isolated tests.

## RED Phase: Baseline Testing

Status:
- Baseline scenario definitions are complete in `references/pressure-scenarios.md`.
- This log captures exact baseline rationalization targets for the first RED execution pass.

Scenario coverage:
- Scenario: Bootstrap Timestamp Missing But Run Continues
- Scenario: Multi-Match Rewrite Without Exact-One-Owner Guard
- Scenario: Ambiguous Match Prompt Lacks Candidate-Specific Options
- Scenario: Critical Pattern File Edited Without Approval
- Scenario: Combined Pressures Across Timestamp, Rewrite, And Ambiguity

Combined pressures used:
- Time, pragmatic, social, sunk-cost, authority, economic

Exact rationalization targets from RED baseline:
1. "No `last_dream_consolidated_at` probably means first run already happened somewhere else, so I will continue with a short window."
2. "Both files are close enough, so rewriting the top one is still better than asking."
3. "I can ask a simpler question first; candidate-specific details can come later if needed."
4. "This promotion is clearly correct and low risk, so writing it now saves a second review step."
5. "Given deadline pressure, I'll do one best-effort merge now and avoid extra prompts."

## GREEN/REFACTOR Note

GREEN and REFACTOR are intentionally pending:
- `SKILL.md` is not authored in this bead.
- Next bead must use the RED rationalizations above to write the minimal contract and then verify.

## Final Outcome For br-13f

- RED baseline scenario pack created.
- Exact rationalization baselines documented for dream-specific high-risk constraints.
- Combined pressures included, including a multi-risk scenario spanning timestamp, rewrite ownership, and ambiguity prompt requirements.

Created: 2026-03-26
Bead: br-13f
Phase: RED baseline
