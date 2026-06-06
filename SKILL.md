---
name: research-records-plan
description: >
  Use this skill whenever a task requires understanding unfamiliar code,
  documentation, APIs, configuration systems, or technical materials before
  acting. Triggers include: "read and implement", "understand then plan",
  "investigate", "research", or any task where acting without prior deep
  reading would risk misunderstanding. This skill enforces a strict
  Research → Plan → Reuse/Cleanup workflow to prevent acting on misread material.
license: MIT
---

# Research → Plan → Reuse / Cleanup Workflow

This skill enforces a three-phase workflow for any task that requires
understanding unfamiliar material before acting. **Never skip or merge phases.**

---

## Phase 1 — Research

Read all referenced code, documentation, and materials thoroughly.
When done, write findings to a temporary file `research.md` in the
current working directory.

### At the start of every new round

If `research.md` already exists, read it before doing anything else and apply
this decision before proceeding:

| Condition | Action |
|-----------|--------|
| Topic and materials are the same as last round. | **Reuse** as-is. Skip to Phase 2. |
| Topic is the same but materials have changed or new ones were added. | **Supplement**: append new findings, mark outdated entries `SUPERSEDED:`. |
| Topic has shifted or `research.md` covers unrelated material. | **Rewrite**: delete and restart Phase 1. |

### research.md format

This file is written **for AI consumption**, not human reading.
Its purpose is to serve as a reliable reference during Phase 2,
preventing semantic drift when the original material is no longer
in the active context window.

Use the following structure. Every entry must be an **explicit assertion**
with a boundary condition or counter-example where ambiguity is possible.
Avoid narrative prose — write declarative statements only.

```markdown
# Research: <topic>

## Key Terms
- `<term>` — <precise definition>. NOT: <common misreading or adjacent concept>.
- `<term>` — <precise definition>. Constraint: <hard limit or invariant>.

## Critical Findings
- <finding>: <explicit assertion of what is true and why it matters>.
- <finding>: <explicit assertion>, confirmed by <source location / line / section>.

## Constraints & Invariants
- <constraint that must not be violated during planning or implementation>.

## Traps & Misreadings
- <thing that looks like X but is actually Y>. Consequence of confusing them: <Z>.
- <parameter / flag / field> is scoped to <X>, not <Y> — do not conflate.

## Open Questions
- <anything unresolved that may affect the plan — flag explicitly>.
```

**Rules for writing research.md:**
- Every ambiguous term gets a "NOT:" clause to pre-empt the most likely misreading.
- Do not summarise — assert. "The default is 512 tokens" not "it seems the default might be around 512".
- Source every non-obvious finding with its location (file path, section heading, line number).
- If a constraint is critical to correctness, mark it `CRITICAL:` on its own line.

---

## Phase 2 — Plan

Before writing the plan, re-read `research.md` in full.
Then construct the execution plan and cross-check each step against `research.md`.

**Cross-check protocol:**
- For every step that touches a term or system documented in `research.md`,
  verify the step is consistent with the recorded definition and constraints.
- If a step conflicts with a finding, resolve the conflict explicitly —
  do not silently proceed.
- If `research.md` contains open questions, decide whether the plan can
  proceed without resolving them or whether they must be resolved first.

**Persistent documentation decision:**

After drafting the plan, decide whether any findings warrant writing to a
persistent document (not subject to cleanup). Apply this decision framework:

1. Check whether an active skill covers persistent documentation
   (e.g. a note-taking or architecture-record skill). If yes, follow
   that skill's conventions for what and where to write.
2. If no relevant skill applies, check the user's prompt for explicit
   instructions about documentation targets or paths.
3. If neither applies, persist findings only when **all three** conditions hold:
   - The finding is non-obvious and unlikely to be re-derivable quickly.
   - The finding affects future tasks beyond the current one.
   - There is a clear, logical place to write it (existing docs folder, ADR, README).
4. If persisting, state explicitly in the plan: what is being written,
   where, and why.

---

## Phase 3 — Reuse or Cleanup

`research.md` is bound to the **task**, not to a single plan. Do not delete it
after a plan is complete — it must remain available for subsequent rounds of
the same task.

### Invalidation conditions (when to delete)

Delete `research.md` only when one of these conditions is explicitly met:

1. **User signals task completion** — e.g. "done", "ship it", "let's move on to X".
2. **Material has changed fundamentally** — a dependency was upgraded, an API
   replaced, or the codebase being studied is no longer the same version.
3. **New task is unrelated** — the incoming task has no meaningful overlap with
   the scope recorded in `research.md`.

When deleting:

```bash
rm research.md
```

Never delete `research.md` between plan rounds on the same task, even if a plan
has been confirmed and execution has begun. It remains the ground truth for the
task until an invalidation condition is explicitly met.

---

## Common Failure Modes

These are the errors this skill is designed to prevent. Recognise them actively.

- **Semantic drift:** A term is correctly understood in Phase 1 but silently
  reinterpreted during Phase 2. The "NOT:" clause in research.md is the
  defence against this.
- **Skipping cross-check:** The plan is written from memory rather than by
  consulting research.md. Always re-read before planning.
- **Premature cleanup:** research.md is deleted after the first plan completes.
  It must persist until an invalidation condition is met.
- **Ignoring existing research.md:** A new round begins without checking whether
  a valid research.md already exists. Always check first.
- **Over-persisting:** Every finding is written to permanent docs, creating
  noise. Use the three-condition check in Phase 2.