---
name: route
description: Which pipeline stage to use for a piece of feature work, from fuzzy idea to merged branch. Use when starting new feature work and it's unclear which skill to reach for first. Not a stage itself — it points at grilling, decision-map, to-spec, to-tickets, subagent-execution, or code-review.
---

# Route

This repo's pipeline, end to end, for one feature at a time. Every stage writes to and reads from `.scratch/<feature-slug>/` — see `CONVENTIONS.md` for the exact layout and formats. Pick the entrance below; each stage's own `SKILL.md` covers its detail.

## Which entrance

**Foggy, or bigger than one session?** Start with `decision-map`. It charts the destination as a local map of decision tickets and works them one at a time — grilling, prototyping, research, whatever each ticket needs — until the way is clear. Its exit artifact (a settled destination) feeds `to-spec`.

**Clear but nontrivial?** `grilling` → `to-spec` → `to-tickets`. Grilling turns a fuzzy idea into a shared understanding, one short question at a time. Once that understanding exists, `to-spec` synthesizes it into a spec (no further interview), and `to-tickets` breaks the spec into tracer-bullet tickets with blocking edges.

**Trivial edits?** Skip this pipeline entirely. A typo fix, a one-line config change, or anything you'd finish faster than you could write a ticket doesn't need `.scratch/`, a branch, or a review gate. Just make the change.

## Execution (either path)

Once tickets exist under `.scratch/<feature>/issues/`, run `subagent-execution`. It creates the feature branch, verifies a clean test baseline, then walks the frontier: a fresh subagent per ticket driving `tdd` at pre-agreed seams, refactoring once per ticket, reviewed on the Spec and Quality axes before the ticket is marked resolved. A fix loop handles review findings up to five rounds; unresolved findings get parked with a ruling, never silently dropped.

## Close

When every ticket is resolved, `subagent-execution` dispatches `code-review` automatically over the whole branch, one more pass on the same two axes plus the deferred-minor ledger. Once that's clean:

1. Merge the branch.
2. Cherry-pick any durable decision out of `.scratch/<feature>/` into permanent docs — an ADR, a `CONTEXT.md` update, a standing convention. `.scratch/` is disposable; nothing that should outlive the feature belongs only there.
3. Delete `.scratch/<feature>/`.

## Summary

```
foggy / multi-session ──▶ decision-map ──▶ to-spec ──▶ to-tickets ──▶ subagent-execution ──▶ code-review ──▶ merge, cherry-pick, delete .scratch/
clear / nontrivial     ──▶ grilling     ──▶ to-spec ──▶ to-tickets ──▶ subagent-execution ──▶ code-review ──▶ merge, cherry-pick, delete .scratch/
trivial                ──▶ just make the change
```
