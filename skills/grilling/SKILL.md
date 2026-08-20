---
name: grilling
description: Interview the user about a plan, decision, or idea, one short question at a time, until every branch is settled. Use to sharpen a fuzzy plan into a shared understanding before writing a spec. Not for charting multi-session work — use decision-map for that; not for writing the spec itself — use to-spec once this reaches a shared understanding.
---

# Grilling

Interview the user until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

## The rules

**One question at a time.** Walk the tree in dependency order: ask the next question whose prerequisites are already settled, wait for the answer, then ask the next. Do not batch several questions into one message — a pile of open questions is harder to answer than one, and it invites you to guess at answers you haven't heard yet.

**Answerable in one sentence.** If a question needs more than a sentence to answer, it's not one question — split it. Keep the question body itself short: state the decision, not the surrounding context the codebase already tells you.

**Always propose your recommendation.** Every question comes with your recommended answer, so the human can just say "yes" and move on:

```
❓ <question, one sentence>

➡️ <your recommended answer>
```

**Explore, don't ask, anything the codebase can answer.** Finding facts is your job, never the user's. When a question needs a fact from the environment (filesystem, existing config, prior art in the codebase), go look it up yourself — dispatch a sub-agent if it's substantial — rather than asking the user to look it up for you. Only ask about things that are genuinely the user's call: preferences, trade-offs, priorities.

**Never answer your own questions.** If you catch yourself proposing an answer and then adopting it without the user weighing in, stop — that's not a recommendation, that's you deciding alone. A question you asked and then answered yourself has settled nothing.

**If the human goes quiet, stop and wait.** Do not proceed by assuming your recommended answer was accepted. An unanswered question is not a settled one.

## Working the tree

Each answer may reshape the tree: a settled decision can unblock questions that depended on it, or invalidate branches that assumed a different answer. After each answer, re-derive which question comes next — don't work off a question list drafted at the start.

The session is done when every branch of the design tree has been visited: nothing left silently assumed, no question you skipped because it felt obvious. Do not act on the plan until the user has confirmed you've reached a shared understanding — that confirmation is the exit criterion, not "no questions left in your head."
