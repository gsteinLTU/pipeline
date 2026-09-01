---
name: vendoring
description: Checklist for pulling a new upstream skill into this repo. Use when adding or updating a skill copied from another project. Not for using the pipeline itself — see route for that.
---

# Vendoring

Everything in this repo is copied and adapted from an upstream skill repo (or written from scratch to fill a gap one leaves), never installed as a live dependency. Upstream is a quarry: pull what's useful at a known commit, adapt it fully, record where it came from. Follow this checklist for any new skill pulled in later — see `VENDORED.md` for the existing entries this checklist already produced.

## 1. Copy at a known commit

Resolve the exact commit or tag SHA you're copying from (`gh api repos/<owner>/<repo>/tags` if pulling a tagged release — the SHA a local plugin install records may belong to a *marketplace* wrapper repo, not the skill repo itself; check before trusting it). Copy the skill's files as they exist at that SHA. Add a row to `VENDORED.md`: skill name → upstream path → SHA → date → a one-line summary of what you changed. Fill the modifications column as you go, not after — it's easy to forget a change once the diff is old.

## 2. De-tracker

Grep the copied files for:

- `docs/agents/issue-tracker.md` or any other tracker-abstraction reference
- setup-skill preconditions (anything telling the user to run a setup command before this skill works)
- triage labels or a `ready-for-agent`-style vocabulary
- "publish to the tracker" / "fetch from the tracker" language
- GitHub/GitLab-specific mechanics (issue numbers, native blocking, assignee-as-claim)

Replace every hit with the hardcoded `.scratch/` conventions in `CONVENTIONS.md`, or delete it if it has no local equivalent (e.g. tracker-native blocking has no analogue — it's just the `Blocked by:` line).

## 3. Trace cross-skill references

Find every place the skill calls another skill by name — `Skill tool with "<name>"`, a slash command, or a prose mention like "see the X skill." For each:

- If the target is vendored in this repo, repoint the reference at this repo's skill name (which may differ from upstream's — check `VENDORED.md`), and namespace it `pipeline:<name>` in the literal argument passed to the Skill tool. Unqualified names resolve ambiguously (or silently wrong) the moment another installed plugin happens to have a same-named skill — this bit `pipeline:code-review` in practice once a standalone `code-review` skill was installed alongside it.
- If the target is not vendored, either vendor it too, stub the step out explicitly (say what would have happened and why it's skipped here), or inline the relevant behavior as prose.

**Unresolved references fail silently** — the agent just improvises the step — so none may remain when you're done. Grep for `Skill tool with` across the repo and check every match both resolves to a real directory under `skills/` and is namespaced `pipeline:`.

## 4. Rewrite the frontmatter description

Every description in this catalog must be mutually exclusive with its neighbors — a small catalog only stays reliable if routing doesn't have to guess between near-duplicates. When you add a skill:

- Check it doesn't overlap an existing one closely enough to cause routing noise (upstream's `grill-me` / `grill-with-docs` / `grilling` overlap is exactly the failure mode to avoid — this repo dropped two of the three for it).
- Add explicit negative space where two skills are adjacent: "not for X — use Y for that."
- Decide `disable-model-invocation` for *this* context. This repo runs fully auto-invocable (no `disable-model-invocation` anywhere) rather than inheriting upstream's per-skill setting — don't assume upstream's choice was made for this catalog's routing needs.

## 5. Conventions pass

Any format the skill reads or writes that's shared with another skill — a ticket, a spec, a map, a ledger line, a review-axis vocabulary — must reference `CONVENTIONS.md`, never restate its own copy of the template inline. This is the single defense against the drift upstream's `Status:` field suffered (three files, three incompatible definitions, never reconciled): a conventions doc only works if nothing bypasses it.

If the skill needs a format `CONVENTIONS.md` doesn't have yet, add it there first, then reference it — don't let the new skill originate a template of its own.
