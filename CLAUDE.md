# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code **plugin** (`.claude-plugin/plugin.json`, name `pipeline`) containing a set of
skills, not an application. There is no build, lint, or test suite in the conventional sense — the
"product" is the prompt text in each `skills/*/SKILL.md`, and the only executable code is a
handful of small shell scripts that support those skills. Every skill here is copied from an
upstream project (`mattpocock/skills` or `obra/superpowers`) at a pinned commit and then adapted —
see `VENDORED.md` for exact provenance and `OVERVIEW.md` for what substantively changed. Nothing is
installed as a live dependency; upstream is a quarry, not a package source.

Read `README.md` first for the pipeline's stage-by-stage flow and design decisions, and
`CONVENTIONS.md` for every shared format (tickets, specs, decision maps, review axes, ledger
lines) skills read and write. This file only adds what those two don't already cover.

## Working in this repo

- **There's no app to run.** "Testing a change" means either invoking the edited skill through
  Claude Code directly, or reading it closely for internal consistency — there's no compiler to
  catch drift.
- **Shell scripts:** `skills/subagent-execution/scripts/review-package` (usage:
  `review-package FEATURE_DIR BASE HEAD [OUTFILE]`) builds the diff package the ticket and
  whole-branch reviewers read; `skills/diagnosing-bugs/scripts/hitl-loop.template.sh` is a
  template, not run directly. Both are the only non-prose code in the repo.
- **Reinstall to see changes reflected:** since skills are vendored into a real plugin, install
  is `/plugin marketplace add /path/to/this/repo` then `/plugin install pipeline` (see README.md).

## Editing a skill: constraints that matter

- **Namespace every cross-skill `Skill` tool call.** Skills in this catalog invoke each other via
  the `Skill` tool by name (e.g. `Skill tool with "pipeline:code-review"`). Always use the
  `pipeline:` prefix in these literal calls, even though the skill also has a standalone name —
  an unqualified name resolves ambiguously (or silently wrong) the moment another installed
  plugin has a same-named skill. This has already caused a real bug once (`code-review` collided
  with a standalone `code-review` skill). Grep for `Skill tool with` across `skills/` and confirm
  every match is both namespaced and resolves to a real directory before considering an edit done.
- **Never restate a shared format inline.** Ticket format, `Status:`/`Blocked by:` syntax, the
  spec template, the decision-map template, the review-axis/severity vocabulary, and ledger line
  formats all live once in `CONVENTIONS.md`. A skill that needs one of these must reference
  `CONVENTIONS.md`, not paste its own copy — this is the direct fix for a live upstream bug
  (Pocock issue #795) where the same `Status:` field was independently defined three incompatible
  ways across three files. If a skill needs a format `CONVENTIONS.md` doesn't have, add it there
  first.
- **Frontmatter descriptions must stay mutually exclusive.** Every skill is fully auto-invocable
  (no `disable-model-invocation` anywhere), so routing depends entirely on descriptions not
  overlapping. When adding or editing a skill, add explicit negative space ("not for X — use Y for
  that") against its nearest neighbors, the way the existing descriptions do.
- **Test behavior-affecting wording like code.** When you edit a routing description, a
  model-selection heuristic, or a new gate, construct 2-3 tiny scenarios designed to make the
  *old* wording fail, and confirm the new wording actually changes the resulting behavior rather
  than just reading better — the same method `vendoring` uses for frontmatter descriptions.
- **No tracker, no worktrees.** This pipeline assumes a solo practitioner with no external issue
  tracker and no git worktree isolation (worktrees have repeatedly broken autonomous runs here).
  Don't reintroduce either — feature-work state belongs under the target repo's disposable
  `.scratch/<feature-slug>/` directory (layout in `CONVENTIONS.md`), and `subagent-execution` uses
  a plain feature branch plus baseline-test verification instead of a worktree.
- **Vendoring a new skill?** Follow `skills/vendoring/SKILL.md`'s checklist exactly (de-tracker,
  trace and repoint cross-skill references, rewrite the description for mutual exclusivity, point
  shared formats at `CONVENTIONS.md`), and add a row to `VENDORED.md` recording upstream path, SHA,
  date, and what changed.

## Structure

Each skill is a directory under `skills/<name>/` containing `SKILL.md` (required) and any
supporting files it references (prompt templates, format docs, scripts) — e.g.
`skills/subagent-execution/` has `implementer-prompt.md`, `reviewer-prompt.md`,
`re-review-prompt.md`, and `scripts/`. The pipeline's own stage order and which skills sit outside
the linear flow (`domain-modeling`, `codebase-design`, `diagnosing-bugs`, `vendoring`) are
documented in `README.md`'s "The pipeline" section — don't re-derive it by reading every
`SKILL.md`, start there.
