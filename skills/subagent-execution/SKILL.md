---
name: subagent-execution
description: Execute a feature's tickets under .scratch/<feature>/issues/ by dispatching a fresh implementer subagent per ticket, a ticket review (spec + quality) after each, and a broad whole-branch review at the end. Use once to-tickets has produced the ticket set. Not for writing the tickets — use to-tickets for that; not for the red-green loop inside a ticket — use tdd for that; not for the closing whole-branch review — use code-review for that (this skill dispatches it automatically at the end).
---

# Subagent Execution

Execute a feature's tickets by dispatching a fresh implementer subagent per ticket, a ticket review (spec compliance + code quality) after each, and a broad whole-branch review at the end.

**Why subagents:** You delegate tickets to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their ticket. They should never inherit your session's context or history — you construct exactly what they need. This also preserves your own context for coordination work.

**Core principle:** Fresh subagent per ticket + ticket review (spec + quality) + broad final review = high quality, fast iteration

**Narration:** between tool calls, narrate at most one short line — the ledger and the tool results carry the record.

**Continuous execution:** Do not pause to check in with your human partner between tickets. Execute the frontier without stopping. The only reasons to stop are the four named below, or every ticket resolved. "Should I continue?" prompts and progress summaries waste their time — they asked you to execute the tickets, so execute them.

**Rulings, not stalls.** Running execution does not wait on a human. Conflicts, ambiguities, ticket defects, a cap you would have asked to exceed — decide them. The spec is the binding authority, the ticket is its argument, and your judgment settles what neither answers. Record every decision in the ledger as `Ruling: <what you decided> — <why> — <what it costs if wrong>`, and keep going. A wrong ruling costs rework your human partner can see and undo; a session parked on a question costs their whole day and buys nothing.

Four things stop you, and only these: an irreversible or destructive operation; a security-sensitive action; a side effect outside this branch that norms say you ask about first (a merge, a push to a shared branch, a publish); and a ticket set so broken that every path forward is a guess. For those, stop and ask.

## Setup

No worktree: this pipeline runs on a **plain feature branch**. Worktree isolation has repeatedly broken autonomous runs here (most likely the dependency-install step upstream tooling bundles with it), so this skill doesn't create one.

1. **Create or check out the feature branch.** Never start implementation on main/master without your human partner's explicit consent.
2. **Skip dependency install.** No `npm install`/`cargo build`/etc. step — if the branch genuinely needs one, that's the human's call, not this skill's.
3. **Verify a clean test baseline** before dispatching anything: run the project's test command. If tests fail, report the failures and ask whether to proceed or investigate — a dirty baseline makes every later failure ambiguous, so don't proceed past it silently. If tests pass, record `MERGE_BASE` (`git merge-base main HEAD`, or the branch point) for the closing review, and report ready.
4. Conversation memory does not survive compaction. Track progress in `.scratch/<feature>/ledger.md`, not only in todos — it is the recovery map: the commits it names exist in git even when your context no longer remembers creating them. After compaction, trust the ledger and `git log` over your own recollection.
5. Read every ticket under `.scratch/<feature>/issues/` once, and the spec at `.scratch/<feature>/spec.md` if it exists — the spec is the authority the tickets argue from, and conflicts inside a ticket resolve against it. A ticket set with no reachable spec gets a ledger note saying so — rulings made without one are provisional. Create a todo per ticket.

Before dispatching the first frontier ticket, scan the ticket set once for conflicts, writing down what you checked as you check it: tickets that contradict each other or the spec's Implementation/Testing Decisions; anything a ticket explicitly mandates that the review rubric treats as a defect. Write the scan to the ledger. Rule on everything you find before execution begins, record each ruling in the ledger, then dispatch the first frontier ticket.

## Model Selection

Use the least powerful model that can handle each role to conserve cost and increase speed.

**Mechanical implementation tickets** (isolated functions, clear acceptance criteria, 1-2 files): use a fast, cheap model.

**Integration and judgment tickets** (multi-file coordination, pattern matching, debugging): use a standard model.

**Architecture and design tickets**: use the most capable available model. The final whole-branch review is one of these — dispatch it on the most capable available model, not the session default.

**Review tasks**: choose the model with the same judgment, scaled to the diff's size, complexity, and risk. Scoped re-reviews of small fix diffs take a cheap-to-mid tier.

**Fix-loop escalation (rounds 4-5)**: use a model at least one tier above the implementer that got stuck.

**Always specify the model explicitly when dispatching a subagent.** An omitted model inherits your session's model — often the most capable and most expensive — which silently defeats this section.

## The Ticket Loop

Walk the **frontier** as defined in `CONVENTIONS.md`: tickets that are `Status: open` and whose every `Blocked by:` entry is `Status: resolved`, lowest number first.

**Batch small same-shape work.** When several tickets are each a small, independent edit of the same kind — the same one-line fix, constant change, or field addition repeated across files — do not dispatch one subagent per ticket. Compose ONE dispatch listing every file and its change, send the whole batch to a single subagent, and review its diff as one unit. Reserve one-dispatch-per-ticket for work that needs its own judgment, its own tests, or its own review surface.

Everything you paste into a dispatch prompt — and everything a subagent prints back — stays resident in your context for the rest of the session and is re-read on every later turn. Hand artifacts over as files.

### 1. Dispatch the implementer

Record BASE (`git rev-parse HEAD`) before dispatching — the review package and fix-round diffs need it.

- Set `Status: claimed` on the ticket file before dispatch.
- The ticket file **is** the implementer's brief — no extraction step needed, since a `.scratch` ticket is already one file. Point the dispatch at it directly: "read this first — it is your requirements, with the exact values to use verbatim."
- Your dispatch should also contain: (1) one line on where this ticket fits in the feature; (2) interfaces and decisions from earlier tickets the ticket file cannot know; (3) your resolution of any ambiguity you noticed in the ticket; (4) the report-file path and report contract.
- Report file: `.scratch/<feature>/reports/<NN>-<slug>-report.md`.
- A dispatch prompt describes one ticket, not the session's history. Do not paste accumulated prior-ticket summaries into later dispatches — a fresh subagent needs its ticket, the interfaces it touches, and the global constraints. Nothing else.
- The dispatch carries the no-subagents contract (it is in the implementer template): the implementer never dispatches subagents — not helpers, and never a reviewer. Review arrives from you, after the report.
- If an earlier ticket parked a finding in the area this ticket touches, carry a pointer to that ledger entry in the dispatch.
- Record the implementer's agent identity from the dispatch result — fix-loop rounds 1-3 resume this agent.
- Never dispatch multiple implementation subagents in parallel (conflicts).

Template: [implementer-prompt.md](implementer-prompt.md)

### 2. Handle the report

Implementer subagents report one of four statuses. Handle each appropriately:

**DONE:** Generate the review package (`scripts/review-package .scratch/<feature> BASE HEAD` — it prints the unique file path it wrote; BASE is the commit you recorded before dispatching the implementer — never `HEAD~1`, which silently drops all but the last commit of a multi-commit ticket), then dispatch the ticket reviewer with the printed path.

**DONE_WITH_CONCERNS:** The implementer completed the work but flagged doubts. Read the concerns before proceeding. If the concerns are about correctness or scope, address them before review. If they're observations (e.g., "this file is getting large"), note them and proceed to review.

**NEEDS_CONTEXT:** The implementer needs information that wasn't provided. Provide the missing context and re-dispatch.

**BLOCKED:** The implementer cannot complete the ticket. Assess the blocker:
1. If it's a context problem, provide more context and re-dispatch with the same model
2. If the ticket requires more reasoning, re-dispatch with a more capable model
3. If the ticket is too large, break it into smaller tickets
4. If the ticket itself is wrong, rule on the correction, ledger it, and re-dispatch with the ruling carried in the dispatch

**Never** ignore an escalation or force the same model to retry without changes. If the implementer said it's stuck, something needs to change.

If the implementer asks questions — before starting or mid-ticket — answer clearly and completely, provide additional context if needed, and don't rush it into implementation.

### 3. Review the ticket

Per-ticket reviews are ticket-scoped gates. The broad review happens once, at the final whole-branch review. Never skip the ticket review, and never accept a report missing either verdict — spec AND quality are both required. Implementer self-review never replaces the ticket review; both are needed.

- Hand the reviewer its diff as a file: run `scripts/review-package .scratch/<feature> BASE HEAD` and pass the reviewer the file path it prints. The output never enters your own context, and the reviewer sees the commit list, stat summary, and full diff with context in one Read call. Use the BASE you recorded before dispatching the implementer — never `HEAD~1`, which silently truncates multi-commit tickets. Never dispatch a ticket reviewer without a diff file.
- **Reviewer inputs:** the ticket reviewer gets the ticket file, the report file, and the review package, plus the global constraints that bind the ticket.
- The global-constraints block you hand the reviewer is its attention lens. Copy the binding requirements verbatim from the spec's Implementation Decisions / Testing Decisions: exact values, exact formats, stated relationships between components. The reviewer's template already carries the process rules (YAGNI, test hygiene, review method) — the constraints block is for what THIS feature's spec demands.
- Do not add open-ended directives like "check all uses" or "run race tests if useful" without a concrete, ticket-specific reason.
- Do not ask a reviewer to re-run tests the implementer already ran on the same code — the implementer's report carries the test evidence.
- Do not pre-judge findings for the reviewer — never instruct a reviewer to ignore or not flag a specific issue. If you believe a finding would be a false positive, let the reviewer raise it and adjudicate it in the review loop. If the prompt you are writing contains "do not flag," "don't treat X as a defect," "at most Minor," or "the ticket chose" — stop: you are pre-judging, usually to spare yourself a review loop.

The ticket reviewer may report "⚠️ Cannot verify from diff" items — requirements that live in unchanged code or span tickets. These do not block the rest of the review, but you must resolve each one yourself before marking the ticket complete: you hold the feature and cross-ticket context the reviewer lacks. If you confirm an item is a real gap, treat it as a failed spec review — it enters the fix loop with the other findings.

Template: [reviewer-prompt.md](reviewer-prompt.md)

### 4. The fix loop

The loop triggers when the review reports spec ❌, any Critical or Important finding, or a ⚠️ item you confirmed as a real gap.

Before the loop starts, two routes leave it immediately:

- Record Minor findings in the ledger as you go (`Task <NN>: minor (deferred): <one-liner>`, per `CONVENTIONS.md`), and point the final whole-branch review at that list so it can triage which must be fixed before merge. A roll-up nobody reads is a silent discard. Minor findings never enter the loop.
- A finding labeled plan-mandated — or any finding that conflicts with what the ticket's text requires — is yours to rule on: weigh the finding against the ticket text, decide with the spec as the binding authority, and ledger the ruling before you act on it. Do not dismiss the finding because the ticket mandates it, and do not dispatch a fix that contradicts the ticket without a recorded ruling.

Everything else enters the loop. A fix round is one fix dispatch plus one scoped re-review. Five rounds maximum per ticket:

**Rounds 1-3 — resume the original implementer.** Send it the open findings verbatim. Its context is intact: it knows the ticket, the code, and its own choices. If your harness cannot send another message to a live subagent, dispatch a fresh implementer carrying the ticket path, the report-file path, and the findings — the report file is the persistent memory either way.

**Rounds 4-5 — dispatch a fresh implementer on a more capable model** (per Model Selection), with the ticket path, the report-file path, the open findings, and this framing: "A prior implementer attempted this ticket [N] times; you own it now. Read the report file for what was tried." A loop that survives three resumes usually means the implementer cannot see its own problem — fresh eyes and a capability bump in one move.

**Every round, either way:** the implementer fixes, re-runs the tests covering the amended code, appends its fix report to the same report file, and returns the short contract. Before re-dispatching the reviewer, confirm the fix report contains the covering tests, the command run, and the output; dispatch the re-review once all three are present. Name the covering test files in the fix message — a one-line fix does not need the whole suite.

**The re-review is scoped.** Run `scripts/review-package .scratch/<feature> FIX_BASE HEAD` where FIX_BASE is the head the previous review saw, and dispatch [re-review-prompt.md](re-review-prompt.md) with the findings list, the ticket file, the report file, and the printed diff path. The re-reviewer verdicts each finding ADDRESSED or NOT ADDRESSED and flags new breakage in the fix diff only. New Critical/Important breakage in the fix diff joins the open findings list. Out-of-scope observations go to the ledger as deferred minors — they never extend the loop.

**After each round,** append to the ledger: `Task <NN>: fix round <R>/5 (<X> addressed, <Y> open — <finding one-liners>; commits <a7>..<b7>)`

Never fix findings yourself in the controller session — your context stays clean for coordination, and controller fixes skip review.

**The breaker.** When round 5's re-review still leaves findings open, stop dispatching. Adjudicate each open finding yourself — you hold the feature and cross-ticket context the reviewer lacks:

- **The reviewer is wrong, or the point is contestable:** park it — `Task <NN>: parked — <finding> — Ruling: <why the code stands>`. The final review sees both sides.
- **Real, but nothing downstream builds on it:** park it the same way, with a ruling that says it's real and deferred.
- **Real and load-bearing** — a later ticket builds on it, or it reveals a spec/ticket defect: rule on the smallest change that unblocks the dependent work, ledger it as `Task <NN>: Ruling: <finding> — <what you decided and why>`, and carry it into the next ticket's dispatch. Parking a structural failure silently lets every dependent ticket build on it. Stop only when the defect leaves every path forward a guess.

Adjudicate only at the cap. Adjudicating earlier to end a loop is pre-judging with a different name. Every adjudication is a ledger entry — a silent discard is forbidden.

### 5. Complete the ticket

When the review comes back clean — or every open finding is parked with a ruling at the cap — set `Status: resolved` on the ticket and append the completion line to the ledger in the same message as your other bookkeeping:

- `Task <NN>: complete (commits <base7>..<head7>, review clean)`
- `Task <NN>: complete (commits <base7>..<head7>, <K> parked)` after a tripped breaker

Then mark the todo complete and move on. Never move to the next ticket while the review has open Critical/Important issues that are neither fixed nor parked-with-ruling at the cap.

## Final Review

Once every ticket is `resolved`, the final whole-branch review gets a package too: run `scripts/review-package .scratch/<feature> MERGE_BASE HEAD` (MERGE_BASE recorded during Setup) and include the printed path in the final review dispatch, so the final reviewer reads one file instead of re-deriving the branch diff with git commands. Dispatch on the most capable available model (see Model Selection), calling the Skill tool with "pipeline:code-review" over `MERGE_BASE..HEAD`. Point it at the ledger's deferred-minor and parked lines so it can triage which must be fixed before merge.

If the final whole-branch review returns findings, dispatch ONE fix subagent with the complete findings list — not one fixer per finding. Per-finding fixers each rebuild context and re-run suites; a real session's final-review fix wave cost more than all its tickets combined. Then run exactly one scoped re-review of the fix wave (`scripts/review-package .scratch/<feature> FIX_BASE HEAD` over the fix range, [re-review-prompt.md](re-review-prompt.md)). Adjudicate any residual findings as in the ticket loop's breaker: park with rulings, or rule on the load-bearing ones and ledger what you decided. Only the four classes above stop you here. There is no second fix wave — residual load-bearing findings surface to your human partner.

## Finish

Before you delete anything, collect every ledger line containing `Ruling:` — preflight rulings, parked findings, breaker adjudications, all of them — into your final message under "Rulings I made", in the order you made them, each with what it costs if wrong. The list is exhaustive: if the ledger holds a ruling, the list holds it. That list is the only place the decisions you took on your human partner's behalf reach them — they read it and rework whatever you got wrong. A ruling that dies with `.scratch/` was a decision made in secret.

When the final whole-branch review is clean and its fixes are merged, hand off to merging the branch and cherry-picking any durable decisions out of `.scratch/<feature>/` before deleting it (see the route skill's Close stage). Do not delete `.scratch/<feature>/` yourself — that's the human's call once they've reviewed the rulings list.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Close enough on spec compliance" | Reviewer found spec gaps = not done. Fix or hit the cap and adjudicate — those are the only exits. |
| "I'll fix it myself, dispatching is overhead" | Controller fixes pollute your context and skip review. Resume the implementer. |
| "One more round will converge" | Past the cap, rounds don't converge — the failure is structural. Adjudicate and route. |
| "The reviewer will just find something new anyway" | Scoped re-reviews verify fixes; they cannot wander. New findings on untouched code go to the ledger, not the loop. |
| "This finding is obviously wrong, I'll drop it" | You adjudicate only at the cap, and every ruling is a ledger entry. Silent discards are forbidden. |
| "The fix was small, skip the re-review" | Unreviewed fixes are how regressions land. Every round ends with a scoped re-review. |
| "Reviews slow the loop down" | The loop without reviews is just unverified churn. Reviews are the loop's brakes and steering. |
| "Ledger bookkeeping is overhead" | The ledger is what survives compaction. Controllers without one have re-dispatched entire completed ticket sequences. |
| "The implementer spawned its own reviewer — free extra assurance" | It's a duplicate seat reviewing the same diff; the ticket review is the gate. A worker-spawned reviewer is a defect to flag, not rigor. |
