---
name: decision-map
description: Chart a huge or foggy chunk of work as a local map of decision tickets at .scratch/<effort>/map.md, and resolve them one at a time until the way to a spec or change is clear. Use when the idea is too big for one session or too fuzzy to spec directly. Not for a clear, single-session decision — use grilling directly; its exit artifact feeds to-spec.
---

A loose idea has arrived, too big for one agent session, and wrapped in fog: the way from here to the **destination** isn't visible yet. This skill charts the way as a **local map** at `.scratch/<effort>/map.md`, then works its **decision tickets** (questions whose resolution is a decision, not slices of a build to execute) one at a time until the route is clear.

The destination varies per effort, and naming it is the first act of charting: it shapes every ticket. It might be a spec to hand off to `to-spec`, a decision to lock before ticketing starts, or a change made in place like a data-structure migration. The map is domain-agnostic: engineering work, course content, whatever fits the shape.

## Plan, don't do

This skill is **planning** by default: each ticket resolves a decision, and the map is done when the way is clear, with nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal you've reached the edge of the map and it's time to hand off. An effort can override this in its **Notes**, carrying execution into the map itself, but absent that, produce decisions, not deliverables.

## The map

The map lives at `.scratch/<effort>/map.md`, in the format defined in `CONVENTIONS.md`. It is an **index**, not a store: it lists the decisions made and points at the tickets that hold their detail; a decision lives in exactly one place, its ticket, so the map never restates it, only gists it and links.

Load the whole map at low resolution once per session — don't load every ticket body up front.

### Tickets

Each ticket is one file at `.scratch/<effort>/issues/<NN>-<slug>.md`, per `CONVENTIONS.md`'s ticket format, with a `## Question` body sized to one session:

```markdown
## Question

<the decision or investigation this ticket resolves>
```

Each ticket also carries a `Type:` line, one of `research`, `prototype`, `grilling`, `task` (see Ticket Types below), placed alongside its `Status:` and `Blocked by:` lines.

A session **claims** a ticket by setting `Status: claimed` before any work, per `CONVENTIONS.md`. Blocking is the `Blocked by:` line; a ticket is **unblocked** when every ticket it names is `Status: resolved`. The **frontier** is the open, unblocked, unclaimed tickets — the edge of the known — as defined in `CONVENTIONS.md`.

The answer isn't part of the ticket body up front; it's appended on resolution under a `## Answer` heading (see Work through the map). Assets created while resolving a ticket are linked from the ticket, not pasted in.

## Ticket types

Every ticket is either **HITL** (human in the loop, worked _with_ a human who speaks for themselves) or **AFK**, driven by the agent alone. A HITL ticket only resolves through that live exchange; the agent never stands in for the human's side of it — a grilling session that answers its own questions has broken this.

- **Research** (AFK): Reading documentation, third-party APIs, or local resources to surface a fact a decision waits on. Use when knowledge outside the current working directory is required; resolve it by exploring directly or dispatching a sub-agent.
- **Prototype** (HITL): Raise the fidelity of the discussion by making a cheap, rough, concrete artifact to react to (an outline, a rough take, a stub, or UI/logic code). Links the prototype as an asset. Use when "how should it look" or "how should it behave" is the key question.
- **Grilling** (HITL): Conversation. The default case. Call the Skill tool twice, for "pipeline:grilling" and "pipeline:domain-modeling".
- **Task** (HITL or AFK): Manual work that must happen before a _decision_ can be made: nothing to decide, prototype, or research, but the discussion is blocked until it's done. Signing up for a service so its API can be judged, provisioning access, moving data so its shape can be seen. This is the one type that _does_ rather than decides, and it earns its place by unblocking a decision, not by delivering the destination. The agent drives it alone where it can (AFK); otherwise it hands the human a precise checklist (HITL). Resolved when the work is done; the answer records what was done and any resulting facts (credentials location, new paths, row counts) later tickets depend on.

## Fog of war

The map is _deliberately_ incomplete: don't chart what you can't yet see. Beyond the live tickets lies the **fog of war**: the dim view of decisions and investigations you can tell are coming but can't yet pin down, because they hang on questions still open. Resolving a ticket clears the fog ahead of it, graduating whatever's now specifiable into fresh tickets, one at a time, until the way to the destination is clear and no tickets remain.

The map's **Not yet specified** section is where that dim view is written down: the suspected question, the area to revisit later. It's the undiscovered frontier _toward_ the destination: everything here is in scope, just not sharp enough to ticket. Write as loosely or as fully as the view allows; it doubles as a signpost for anyone reading where the effort is headed.

**Fog or ticket?** The test is whether you can state the question precisely now, _not_ whether you can answer it now.

- **Ticket when** the question is already sharp, even if it's blocked and you can't act on it yet.
- **Not yet specified when** you can't yet phrase it that sharply. Don't pre-slice the fog into ticket-sized pieces: it's coarser than a ticket, and one patch may graduate into several tickets, or none, once the frontier reaches it.

**Not yet specified** excludes what's already decided (Decisions so far), what's already a live ticket, and what's out of scope (the next section).

## Out of scope

Fog only ever gathers _toward_ the destination. The destination fixes the scope, so work beyond it is **out of scope**: it isn't fog, and it doesn't belong in **Not yet specified**. It gets its own **Out of scope** section on the map: work you've consciously ruled out of _this_ effort. Scope, not sharpness, lands it here.

Out-of-scope work never graduates (the frontier stops at the destination), so it returns only if the destination is redrawn, and then as a fresh effort, not a resumption.

Ruling something out of scope is a scoping act, not a step on the route. When a ticket that already exists turns out to sit past the destination (mis-scoped in while charting, or exposed by a resolution), set `Status: resolved` with an `## Answer` of "out of scope" (a resolved ticket is unambiguously off the frontier) and leave one line in the **Out of scope** section: the gist plus why it's out of scope, linking the ticket. It stays out of **Decisions so far**, which records the route actually walked; a scope boundary isn't a step on it.

## Invocation

Two modes. Either way, **never resolve more than one ticket per session**, with the exception of research tickets.

### Chart the map

User invokes with a loose idea.

1. **Name the destination.** Call the Skill tool twice, for "pipeline:grilling" and "pipeline:domain-modeling", to pin down what this map is finding its way to: the spec, decision, or change. The destination fixes the scope, so it's settled first.
2. **Map the frontier.** Grill again, **breadth-first** this time: fan out across the whole space rather than deep on any one thread, surfacing the open decisions and the first steps takeable now. **If this surfaces no fog** (the way to the destination is already clear, the whole journey small enough for one session), you don't need a map. Stop and ask the user how they'd like to proceed — probably straight to `to-spec`.
3. **Create the map** at `.scratch/<effort>/map.md`: Destination and Notes filled in, Decisions-so-far empty, the fog sketched into **Not yet specified**.
4. **Create the tickets you can specify now** under `.scratch/<effort>/issues/`, per `CONVENTIONS.md`'s ticket format (`Status: open`), numbered and wired with `Blocked by:` in the same pass — local files have no id-ordering problem. Wiring sorts them into the frontier and the blocked; everything you can't yet specify stays in the fog: the **Not yet specified** section.
5. **Resolve research tickets now.** For each `research` ticket you just created, resolve it directly or via a sub-agent, in parallel, capturing findings in the ticket's `## Answer`.
6. Stop: charting is one session's work; it hand-resolves nothing beyond research tickets.

### Work through the map

User invokes with an effort slug. A ticket is **optional**: without one, you pick the next decision, not the user.

1. Load the **map**: the low-res view at `.scratch/<effort>/map.md`, not every ticket body.
2. Choose the ticket. If the user named one, use it. Otherwise take the first frontier ticket in order. **Claim it**: set `Status: claimed` before any work.
3. Resolve it. **Zoom as needed**: read the full body of any related or resolved ticket on demand; call the Skill tool for whichever skills the `## Notes` block names. If in doubt, call the Skill tool twice, for "pipeline:grilling" and "pipeline:domain-modeling".
4. Record the resolution: append the answer under a `## Answer` heading in the ticket, set `Status: resolved`, and append a context pointer to the map's Decisions-so-far.
5. Add newly-surfaced tickets; graduate any fog the answer has made specifiable, clearing each graduated patch from **Not yet specified** so it lives only as its new ticket. If the answer reveals that a ticket (this one or another) sits beyond the destination, **rule it out of scope** rather than resolving it on the route. If the decision invalidates other parts of the map, update or delete those tickets.

When the map is fully resolved and the destination is a spec, hand off to `to-spec`; if it's already a set of implementation-ready decisions, hand off to `to-tickets`.
