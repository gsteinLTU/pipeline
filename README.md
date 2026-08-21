# pipeline

A personal skills repo for Claude Code, composing the best of two upstream projects into one
explicit pipeline:

- **[mattpocock/skills](https://github.com/mattpocock/skills)** — requirements interrogation,
  spec/ticket pipeline, TDD, two-axis code review, domain modeling
- **[obra/superpowers](https://github.com/obra/superpowers)** — subagent-per-task execution with
  per-task review, hard TDD enforcement

Everything here is **vendored and adapted**, not installed as a dependency. Upstream is a quarry —
see `VENDORED.md` for exactly what was copied from where, at which commit, and how it was changed.
For what's actually different in the result — not just the provenance — see `OVERVIEW.md`.

Licensed MIT (see `LICENSE`). Both upstream projects are also MIT-licensed; their license text and
copyright notices are preserved in `NOTICE`, as MIT requires for reused and adapted material.

This repo assumes a solo practitioner: no team, no concurrency, no external issue tracker. All
feature-work state lives in a disposable local `.scratch/` directory instead — see
`CONVENTIONS.md` for its layout and every shared format (tickets, specs, decision maps, review
findings) this pipeline reads and writes.

## Install

```
/plugin marketplace add /path/to/this/repo
/plugin install pipeline
```

Skills are namespaced under `pipeline:` — e.g. `pipeline:route`, `pipeline:tdd`,
`pipeline:code-review`.

## The pipeline

```
foggy / multi-session ──▶ decision-map ──▶ to-spec ──▶ to-tickets ──▶ subagent-execution ──▶ code-review ──▶ merge, cherry-pick, delete .scratch/
clear / nontrivial     ──▶ grilling     ──▶ to-spec ──▶ to-tickets ──▶ subagent-execution ──▶ code-review ──▶ merge, cherry-pick, delete .scratch/
trivial                ──▶ just make the change
```

Start with `pipeline:route` if it's unclear which stage to use — it's the router, not a stage
itself. In short:

- **Foggy or bigger than one session:** `pipeline:decision-map` charts the destination as a local
  map of decision tickets, resolved one at a time, until the way is clear. Its exit artifact feeds
  `to-spec`.
- **Clear but nontrivial:** `pipeline:grilling` → `pipeline:to-spec` → `pipeline:to-tickets`.
  Grilling is a one-question-at-a-time interview with a recommended answer on every question, so
  you can usually just say "yes." `to-spec` synthesizes the conversation into a spec, no further
  interview. `to-tickets` breaks the spec into tracer-bullet tickets with blocking edges.
- **Execution (either path):** `pipeline:subagent-execution` creates a feature branch, verifies a
  clean test baseline, then walks the ticket frontier — a fresh subagent per ticket driving
  `pipeline:tdd` at pre-agreed seams, one refactor pass per ticket, reviewed on Spec and Quality
  before being marked resolved. A fix loop handles review findings (5 rounds max); anything
  unresolved is parked with a recorded ruling, never silently dropped.
- **Close:** the whole-branch `pipeline:code-review` runs automatically at the end of execution.
  Once it's clean: merge, cherry-pick any durable decision out of `.scratch/<feature>/` into
  permanent docs, then delete `.scratch/<feature>/`.
- **Trivial edits:** skip the pipeline. A typo fix doesn't need a ticket.

Two more skills sit outside the linear flow and get called into whichever stage needs them:
`pipeline:domain-modeling` (glossary/ADR discipline) and `pipeline:codebase-design` (deep-module
vocabulary — where a seam goes, what an interface is). `pipeline:diagnosing-bugs` is a standalone
diagnosis loop for hard bugs, not part of the feature pipeline. `pipeline:vendoring` is the
checklist for pulling in more upstream skills later.

## Design decisions worth knowing

- **Fully auto-invocable.** Every skill here can be triggered by its description matching the
  task — there's no `disable-model-invocation` anywhere, and there's no mandatory session-start
  routing hook either. This is a deliberate middle ground: reachable, never forced.
- **Branches, not worktrees.** Worktree isolation has repeatedly broken autonomous runs here (most
  likely the bundled dependency-install step). `subagent-execution` uses a plain feature branch
  and keeps upstream's baseline-test-verification idea instead, so "tests fail" stays unambiguous.
- **One conventions doc.** `CONVENTIONS.md` is the single source of truth for every shared format.
  No skill restates a template inline — this is the direct fix for a live bug in upstream Pocock
  (issue #795) where the same `Status:` field was defined three incompatible ways across three
  files.

## Post-build step

This repo overlaps six skills in `superpowers@claude-plugins-official` (`tdd` /
`test-driven-development`, `subagent-execution` / `subagent-driven-development`, `diagnosing-bugs`
/ `systematic-debugging`, `grilling` / `decision-map` / `brainstorming`), and that plugin's
session-start hook injects mandatory-routing language into every session. Once this pipeline is
working, disable it:

```json
// settings.json
"enabledPlugins": {
  "superpowers@claude-plugins-official": false
}
```

## Explicitly dropped

`ask-matt`, `setup-matt-pocock-skills`, `triage`, `grill-me`, `grill-with-docs`,
`improve-codebase-architecture`, `resolving-merge-conflicts`, `prototype`, `handoff`, `teach`,
`research`, `implement`, `wizard`, and superpowers' `brainstorming`, `writing-plans`,
`executing-plans`, `using-git-worktrees`, `finishing-a-development-branch`, `using-superpowers`,
`dispatching-parallel-agents`, `verification-before-completion`, `receiving-code-review`,
`writing-skills`, `systematic-debugging`. See `VENDORED.md` for why each was dropped.

`wizard` is the most plausible future add, if infrastructure- or credential-walkthrough work
becomes routine.

## Watch items

- Coarse-ticket drift in `subagent-execution` — tickets are coarser than upstream's 2-5 minute
  tasks. Fallback is a per-ticket mini-plan step, not a return to feature-level plan-writing.
- Whether full auto-invocation misroutes once superpowers is disabled.
- Grilling question length, even with the one-question-at-a-time rule and word cap.
- Quarterly diff from the pinned upstream SHAs in `VENDORED.md`.
