# Conventions

Every shared artifact this pipeline reads or writes — tickets, specs, decision maps, review
findings — has exactly one format, defined here. No skill restates these templates inline; each
references this file instead. This is deliberate: upstream `mattpocock/skills` has a live bug
(issue #795) where three different skill files each restated a `Status:` schema, and the three
definitions drifted apart until they conflicted. A conventions doc is the only defense a
prompt-driven skill has against that, because there's no compiler to catch the drift.

If you're writing or editing a skill and find yourself typing out a template instead of linking
here, stop — either the template belongs here, or this doc is missing something the skill needs.
See `skills/vendoring/` for the checklist that enforces this when pulling in new skills.

## `.scratch/` layout

All feature-work state lives under `.scratch/<feature-slug>/` in the target repo. Nothing here
touches an external tracker — no GitHub issues, no Linear, no setup step. `.scratch/` is
disposable: gitignore it, or delete it once a feature ships. Before deleting, cherry-pick any
durable decision (an ADR, a `CONTEXT.md` update, a standing convention) into permanent docs — the
scratch directory is where decisions get made, not where they live afterward.

```
.scratch/<feature-slug>/
├── spec.md              # written by to-spec
├── map.md                # written by decision-map, only for foggy/multi-session efforts
├── issues/
│   ├── 01-<slug>.md      # written by to-tickets, one file per ticket
│   ├── 02-<slug>.md      # numbered from 01 in dependency order, blockers first
│   └── ...
├── ledger.md             # written by subagent-execution: rulings, parked/deferred findings
└── reports/
    ├── 01-<slug>-report.md     # implementer report for ticket 01
    ├── 01-<slug>-diff.txt      # review-package output for ticket 01
    └── ...
```

A `<feature-slug>` is a short kebab-case name for the effort, chosen when `grilling` or
`decision-map` starts (e.g. `.scratch/user-auth/`).

## `Status:` vocabulary

Every ticket file carries exactly one `Status:` line, near the top, as plain text:

```
Status: open
```

Never bold (`**Status:**`), never a heading (`## Status`), never inside a table. This is the only
line format — upstream drifted across four different renderings of the same concept (`**Status:**`,
`## Blocked by`-style headings, bare lines, and omission); this repo has one.

**Allowed values, lifecycle only:** `open | claimed | resolved`. There is no triage vocabulary
(`ready-for-agent` and similar role strings are dropped along with the triage skill they came from)
and no value for "blocked" — blocked-ness is derived from the `Blocked by:` line and the frontier
rule below, not stored as a status.

**Every value has exactly one writer**, named here so the vocabulary can't go stale the way
upstream's did (nothing upstream ever wrote a transition out of its initial value):

| Value | Meaning | Written by |
|---|---|---|
| `open` | Ticket exists, not yet claimed | `to-tickets` (implementation tickets, at creation); `decision-map` (decision tickets, at creation) |
| `claimed` | A subagent — or the driving session — has started work on this ticket | `subagent-execution` (implementation tickets, before dispatch); `decision-map` (decision tickets, before resolving one) |
| `resolved` | Review passed (or the final adjudication ruled the code stands), or a decision ticket's answer was recorded | `subagent-execution` (implementation tickets, after review passes or cap-adjudication rules "stands"); `decision-map` (decision tickets, on recording the answer — including "out of scope") |

If you are reading a ticket and its `Status:` doesn't match what actually happened to it, that's a
bug in whichever skill should have written the transition — fix the skill, not the vocabulary.

## `Blocked by:` line

One syntax, a bare line near the top of the ticket, directly under `Status:`:

```
Blocked by: 02, 04
```

or, if nothing gates it:

```
Blocked by: None
```

Values are the two-digit ticket numbers it depends on. Never `**Blocked by:**`, never a `##
Blocked by` heading, never omitted.

## Frontier

Defined once, here, instead of the four incompatible definitions upstream scattered across
`grilling`, `to-tickets`, `wayfinder`, and the local-tracker doc:

> The **frontier** is the set of tickets in `.scratch/<feature>/issues/` that are `Status: open`
> and whose every `Blocked by:` entry points at a `Status: resolved` ticket. Within the frontier,
> lowest ticket number first.

`subagent-execution` walks the frontier this way; nothing else needs its own definition.

## Ticket format

One file per ticket, `.scratch/<feature-slug>/issues/<NN>-<slug>.md`:

```markdown
# <NN>: <Ticket title>

Status: open
Blocked by: <NN, NN, ... | None>

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective,
not a layer-by-layer implementation list.

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2
```

Never a single combined tickets file. Numbering is dependency order, blockers first, starting at
`01`.

## Spec format

Written by `to-spec` to `.scratch/<feature-slug>/spec.md`:

```markdown
## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

This list of user stories should be extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can
(state machine, reducer, schema, type shape), inline it within the relevant decision and note
briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.
```

## Decision-map format

Written by `decision-map` to `.scratch/<effort>/map.md`:

```markdown
## Destination

<What reaching the end of this map looks like: the spec, decision, or change this effort is
finding its way to. One or two lines; every session orients to it before choosing a ticket.>

## Notes

<Domain context; skills every session should consult; standing preferences for this effort>

## Decisions so far

<!-- The index: one line per resolved ticket, enough to judge relevance, then zoom the link for
     the detail the ticket holds. -->

- [<resolved ticket title>](issues/NN-slug.md): <one-line gist of the answer>

## Not yet specified

<!-- In-scope fog you can't ticket yet, because you can't state the question precisely. Graduates
     into fresh tickets as the frontier advances — see "fog vs ticket" in decision-map's SKILL.md. -->

## Out of scope

<!-- Work ruled beyond the destination. Never graduates; returns only as a fresh effort if the
     destination itself is redrawn. -->
```

Each map ticket's body is one line:

```markdown
## Question

<The decision or investigation this ticket resolves>
```

## Review axes and severity

Both the per-ticket gate (`subagent-execution`) and the closing whole-branch review (`code-review`)
judge a diff on the same two axes, at different scopes — per-ticket vs. whole-branch. One
vocabulary, two scopes; findings are never merged or reranked across axes, because a
standards-pass/spec-fail diff and a spec-pass/standards-fail diff are both real failure modes, and
merging them lets one mask the other.

**Spec** — does the diff match what was asked for?
- **Missing** — requirements skipped, missed, or claimed without being implemented
- **Extra** — features that weren't requested, over-engineering, unneeded additions
- **Misunderstood** — the right feature built the wrong way, or the wrong problem solved

**Quality** — is the diff well-built?
- Clean separation of concerns, proper error handling, DRY without premature abstraction, edge
  cases handled
- Tests verify real behavior, not mocks; the ticket's edge cases are covered
- Structure: each file has one clear responsibility; the change didn't balloon an existing file or
  create an already-oversized new one
- Reference baseline, applied as labelled heuristics ("possible Feature Envy"), never hard
  violations — skip anything tooling already enforces, and a documented repo standard always wins
  over these: Mysterious Name, Duplicated Code, Feature Envy, Data Clumps, Primitive Obsession,
  Repeated Switches, Shotgun Surgery, Divergent Change, Speculative Generality, Message Chains,
  Middle Man, Refused Bequest.

**Severity ladder**, used to bucket every finding on either axis:
- **Critical** — bugs, security issues, data loss risks, broken functionality
- **Important** — this task cannot be trusted until it is fixed: incorrect or fragile behavior, a
  missed requirement, or maintainability damage you'd block a merge over (verbatim duplication of
  a logic block, swallowed errors, tests that assert nothing)
- **Minor** — coverage could be broader, polish suggestions, style

Not everything is Critical. Categorize by actual severity, and never pre-judge a finding for a
reviewer — don't instruct a reviewer to ignore or cap the severity of something before it's found.
A finding that conflicts with what the spec or ticket explicitly mandates is still a finding,
labeled `plan-mandated`; the human rules on it, the plan's authorship doesn't grade its own work.

## Ledger format

`subagent-execution` writes one line per ledger event to `.scratch/<feature>/ledger.md`:

```
Task <NN>: minor (deferred): <one-liner>
Task <NN>: parked — <finding> — Ruling: <why the code stands>
Task <NN>: Ruling: <finding> — <decision and why>
```

Minor findings are logged, never enter the fix loop, and get triaged by the closing `code-review`.
Every adjudication at the fix-loop's round cap is a ledger line — a silent discard is never
acceptable.
