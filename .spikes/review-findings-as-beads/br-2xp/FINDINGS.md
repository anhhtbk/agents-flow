# Spike Findings: Determine Non-Blocking Review Bead Linkage Strategy

**Spike bead:** `br-2xp`
**Feature:** `review-findings-as-beads`
**Date:** 2026-03-25

## Question

Can review-created `P2` / `P3` follow-up beads remain traceable to the reviewed feature without blocking current epic closure?

## Method

Ran an isolated `br` experiment in a temporary workspace:

1. Created an epic and an open child task.
2. Checked `br epic status --json`.
3. Deferred the child task and checked status again.
4. Removed the parent link from the child and checked status.
5. Created a separate follow-up task linked by `--external-ref <epic-id>` plus labels and checked status.
6. Created and closed a parented child task to confirm what `eligible_for_close: true` requires.

## Results

- **Open child bead:** epic was **not** eligible for close.
- **Deferred child bead:** epic was still **not** eligible for close.
- **Closed child bead:** epic became **eligible for close**.
- **Non-child bead with external reference + labels:** did **not** count as a child and therefore did not participate in epic-close eligibility.

## Conclusion

**YES** — there is a viable strategy.

The correct linkage model is:

- `P1` review fix beads may be attached to the current epic because they are intentionally blocking and must be resolved before finish.
- `P2` / `P3` review follow-up beads must **not** be children of the current epic.
- For `P2` / `P3`, preserve traceability with metadata instead:
  - `external_ref=<source-epic-id>` or equivalent cross-reference
  - labels such as `review`, `review-p2` / `review-p3`, and feature/source-agent labels
  - optional defer scheduling when the workflow wants them explicitly queued for later

## Recommendation For Implementation

Encode this as an explicit rule in the reviewing contract and reviewer prompts:

- Blocking review work (`P1`) is part of the current fix/close path.
- Non-blocking review work (`P2` / `P3`) is separate follow-up work, traceable but outside the current epic-close path.
