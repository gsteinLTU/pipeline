# How this differs from either upstream

This repo is not "Pocock plus Superpowers installed side by side." It's a third thing: a hybrid
built by taking the front half of one pipeline and the back half of the other, replacing what both
of them lean on (an external issue tracker, git worktrees) with a single local mechanism, and
resolving the places where the two upstreams actively disagreed rather than picking one arbitrarily.
This doc is the diff — what came from where, what changed, and why.

## The shape of the merge

Neither upstream is a complete pipeline on its own.

**mattpocock/skills** covers requirements interrogation → spec → tickets in detail (`grilling`,
`to-spec`, `to-tickets`, `wayfinder`), but its execution story is a 9-line `implement` skill: "use
`/tdd`, run tests, use `/code-review`, commit." It never claims a ticket, never flips a status,
never loops on review findings. Its tracker abstraction (`docs/agents/issue-tracker.md`) assumes a
configured GitHub/GitLab/Linear-or-local backend, set up by a dedicated `setup-matt-pocock-skills`
skill this repo doesn't have.

**obra/superpowers** covers execution in detail (`subagent-driven-development`: fresh subagent per
task, per-task review, a five-round fix loop with adjudication, a final whole-branch review), but
it has no requirements-gathering front half at all — it consumes a `writing-plans` plan file, and
getting there is out of scope. It also hard-requires git worktrees (`using-git-worktrees`) and
injects a mandatory-routing instruction into every session via a `SessionStart` hook.

This repo takes Pocock's front half, Superpowers' back half, and welds them at the ticket: Pocock's
`to-tickets` writes files in a format Superpowers' execution engine (renamed `subagent-execution`)
reads directly, no plan-file translation step in between.

## What replaced the tracker: `.scratch/` and `CONVENTIONS.md`

Both upstreams assume infrastructure this repo doesn't have. Pocock assumes a tracker; Superpowers
assumes a plan file plus a worktree. The replacement for both is the same: a disposable
`.scratch/<feature-slug>/` directory, and one file, `CONVENTIONS.md`, that defines every shared
format once.

This is the single biggest structural difference from either original, and it's not just
"local instead of remote" — it fixes a live bug. Upstream Pocock defines a ticket's `Status:` field
three separate times, in three separate files, and the definitions disagree:

- `issue-tracker-local.md` calls it triage state, values like `ready-for-agent`
- `to-tickets`'s own ticket template writes `**Status:** ready-for-agent` — bold, not the bare line
  the tracker doc specifies
- the same tracker doc's *wayfinding* section redefines it as lifecycle state, values `claimed` /
  `resolved`

Nothing in upstream ever transitions a ticket out of its initial value — `to-tickets` writes
`ready-for-agent` and nothing downstream ever changes it. A ticket's status is permanently stale
the moment work starts. This is filed upstream as issue #795.

`CONVENTIONS.md` fixes this by existing exactly once: one `Status:` vocabulary
(`open | claimed | resolved`, lifecycle only, no triage role strings), one plain-line syntax (never
bold, never a heading), and — the part upstream never did — a table naming which skill writes each
value and at which moment (`to-tickets`/`decision-map` write `open`; `subagent-execution`/
`decision-map` write `claimed` then `resolved`). Every skill that touches a ticket, spec, map, or
review finding references this file instead of restating its own copy of the template. That's a
constraint neither upstream repo has, because neither upstream repo has a single shared conventions
doc at all.

## What replaced worktrees: branch + baseline, no dependency install

Superpowers' `subagent-driven-development` opens with "ensure the work happens in an isolated
workspace: use `using-git-worktrees`." That skill does two things before any code gets touched: it
creates the worktree, then runs a project-appropriate dependency install (`npm install`,
`cargo build`, `poetry install`, `go mod download`), then verifies a clean test baseline.

This repo's `subagent-execution` does the third thing and skips the first two. It creates or checks
out a plain feature branch — worktree isolation has repeatedly broken autonomous runs in this
owner's actual usage, most likely because of exactly that bundled dependency-install step — and then
verifies the clean baseline directly, using upstream's own language for it almost verbatim ("a dirty
baseline makes every later failure ambiguous"). The baseline-verification *idea* survives untouched;
the worktree and the auto-install around it don't. Nothing downstream in either upstream repo
restates that baseline check outside `using-git-worktrees` itself, so dropping the skill without
relocating the check would have silently deleted it — that's a trap this repo specifically avoided,
not an oversight upstream would have caught either.

## Where this repo overrides upstream's own design choices

Two places, this repo didn't just adapt upstream — it deliberately did something upstream doesn't:

**The TDD loop's refactor step.** Pocock's `tdd` skill explicitly *excludes* refactoring from the
loop: "Refactoring is not part of the loop. It belongs to the review stage." Superpowers'
`test-driven-development` puts refactor back *inside* every red-green cycle (its own diagram is
literally `RED → GREEN → REFACTOR → repeat`). These are two different, deliberate positions from two
different upstream authors, not a case where one is obviously right. This repo's `tdd` does neither:
red→green stays a per-cycle loop with no refactoring mixed in (agreeing with Pocock that mid-cycle
refactoring risks coupling tests to implementation), but instead of pushing refactoring off to a
separate review skill (Pocock) or folding it back into every cycle (Superpowers), it adds one
refactor pass at the *end of each ticket*, tests green throughout, right before that ticket goes to
review. Review then sees already-refactored code instead of having to ask for refactoring as a
finding.

**Grilling's actual failure mode.** The build brief that started this project blamed Pocock's
`grilling` skill's "relentlessly" language for a known failure mode — bloated, multi-paragraph
questions. Reading the actual upstream skill turned up a more specific cause: its own question
format block explicitly permits it (`<question body, might be multiple paragraphs, including
multiple choices>`). This repo's `grilling` drops the intensity language too, but the load-bearing
fix is a rewritten format block with a hard one-sentence constraint, plus a switch from upstream's
batched "ask the whole frontier in one round" to strict one-question-at-a-time. The
recommended-answer format and the no-self-answering rule, by contrast, needed no invention —
upstream already had both (the latter stated in `wayfinder`, not in `grilling` itself, and moved
here into `grilling` where it belongs).

## What got renamed, and why the renames aren't cosmetic

| This repo | Upstream | Why the name changed |
|---|---|---|
| `decision-map` | `wayfinder` (Pocock) | The skill kept its conceptual skeleton (destination-first, fog-of-war, decisions-index-not-store) but lost every tracker-specific mechanism the old name's metaphor leaned on — child issues, assignee-as-claim, tracker-native blocking. What's left is a map of decision tickets, so that's the name. |
| `subagent-execution` | `subagent-driven-development` (Superpowers) | Superpowers' name describes a development methodology; this repo's version is narrower and more mechanical — it's specifically the engine that executes an already-ticketed feature, with no plan-writing or requirements step folded in (those are `grilling`/`to-spec`/`to-tickets`, upstream to this skill entirely). |
| `route` | (plugin name would have been `pipeline`) | The plugin itself is named `pipeline`. A skill also named `pipeline` inside a plugin named `pipeline` produces `/pipeline:pipeline`, which is worse than just calling the router `route`. |

## What got dropped that upstream keeps, and what that costs

- **Superpowers' mandatory-routing bootstrap** (`using-superpowers`, its `SessionStart` hook)
  is gone entirely. Superpowers treats "you have no choice, you must use the skill" as a feature.
  This repo treats it as the thing decision 7 of the build brief explicitly rejected — every skill
  here is auto-invocable by its description matching the task, but nothing forces entry the way
  Superpowers' hook does. The cost: routing now depends entirely on how mutually exclusive twelve
  descriptions are, with no forcing function backing them up if they're not.
- **Superpowers' `writing-plans`/`executing-plans`** are gone; a `.scratch` ticket *is* the unit of
  work `subagent-execution` consumes, with no plan-file layer above it. The cost, named explicitly
  as a watch item: tickets here are coarser than Superpowers' 2-5-minute tasks, so if subagents
  drift on under-specified tickets, the fallback is a per-ticket mini-plan step — not resurrecting
  `writing-plans` wholesale.
- **Pocock's tracker-abstraction skills** (`setup-matt-pocock-skills`, `triage`) are gone along with
  the tracker they configure. The cost: this pipeline has no answer for a team, concurrent sessions,
  or a tracker UI a non-technical stakeholder could glance at — all things a real tracker gives you
  for free. That's an explicit, accepted trade for a solo practitioner, not an oversight.
- **`receiving-code-review`** exists in Superpowers but is referenced by nothing else upstream
  either — it's genuinely orphaned there too. This repo folds its "verify before implementing,
  don't just agree" posture into the reviewer prompt templates' "Do Not Trust the Report" section
  instead of vendoring a whole separate skill for it.

## The net result

Neither upstream repo, used as installed, gets you from "I have a fuzzy idea" to "the feature is
merged and the branch is clean" without either configuring an external tracker (Pocock) or writing
a plan file by hand and fighting worktree setup for a solo workflow that doesn't need the isolation
(Superpowers). This repo's twelve skills are the minimum set that closes that gap for one person,
working alone, with nothing outside the git repo itself.
