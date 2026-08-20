---
name: to-spec
description: Turn the current conversation into a spec at .scratch/<feature>/spec.md, no interview, just synthesis of what's already been discussed. Use once grilling (or decision-map) has reached a shared understanding. Not for interviewing the user — use grilling for that; not for breaking the spec into tickets — use to-tickets for that.
---

This skill takes the current conversation context and codebase understanding and produces a spec. Do NOT interview the user; just synthesize what you already know.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

Check with the user that these seams match their expectations.

3. Write the spec using the template in `CONVENTIONS.md` ("Spec format"), then save it to `.scratch/<feature-slug>/spec.md` (creating the directory if needed).
