---
name: writing-prds
description: Use when writing a PRD, specifying a product or feature before any plan or code, closing product gaps by asking questions, or when the user says writing-prds, /writing-prds, "write the PRD", "spec this", "don't assume", "develop this functionality", or asks to use the installed writing-prds skill. Do not use for implementation plans or coding.
user-invocable: true
argument-hint: "[what to spec]"
when-to-use: write a PRD, spec a feature, story first, don't assume, use writing-prds
---

# Writing PRDs

You write product specs as markdown. You do not plan files. You do not code.

There are no stupid questions. The expensive failure is inventing the product. If a fact is missing, ask. Wait. Then write.

## Solo or cycle

This skill is complete by itself. When the pack is written, **stop**. Do not schedule or orchestrate unless the user asks.

## Before any file

Talk to the user until gaps are closed.

1. **Why** this change exists — who hurts, when, what they want to feel instead.
2. On every real fork: **pros / cons** and one recommendation that is efficient *for this use case*. Do not critique everything. Do not open extra fronts.
3. Wait for the answer. User feedback is the source of truth.
4. If a new gap appears while writing, **stop and ask again**. Do not fill it with a guess.

Do not produce a PRD from a one-liner if you had to assume the story, the entities, or the promises.

## Anatomy (this order, always)

### 0 · Header

Title. Status. Owner. Created. **Scope** — what this document touches and what it does not, on the same line.

### 1 · Summary

**Today:** …  
**After:** …  
Two lines. The cheapest drawing that exists.

### 2 · The story

**Before** / **After** with a **name and a moment**. No jargon.

The story says who the user is, how they use it, what hurts, and what experience they want. Everything else in the document exists to make that story true. If the story does not convince, do not write the rest.

### 3 · Goals / Non-goals

- **O1**, **O2**, … — named. Later sections cite them ("meets O2").
- **NO1**, **NO2**, … — named. These stop "while we're here…".

### 4 · How it works today → how it will work

The flow, drawn twice. Reuse what already exists.

### 5 · The data

Entities, the switch, the lock. What fires the change, what blocks it. No final SQL, no shipping schemas, no env.

### 6 · Pseudo-code — the agreement

```
WHEN <event>
  is <condition>?     → if yes/no, <what happens>
THEN <what we do>

Promises: <guarantees, not implementation>
```

What fires it, what stops it, what we promise. Zero final code.

## Hard rule

A PRD fixes **structure** — the story, which entities change, and how they change — in prose and pseudo-code.

A PRD **never** contains: final code, the exact implementation, finished screens, configuration.

## Size decides how many files

| Change | What you write |
|---|---|
| A tweak | One PRD, ~1 page |
| A feature | One PRD, 3–8 pages |
| A large feature | One PRD, 10+ pages |
| A new product | Nested PRDs. The parent tells the product story and is **spec-only** (it is not implemented alone). Each child has its own story. No child carries the whole product. |

## Pack layout

Ask where to write. Default: `prds/<slug>/`.

```
prds/<slug>/
  _pack.md
  00-<product>.md          # parent, spec-only, if this is a product
  01-<slice>.md
  02-<slice>.md
```

`_pack.md` (you write this; do not invent waves):

```markdown
# Pack

parent: 00-<product>.md
spec_only:
  - 00-<product>.md
implementable:
  - 01-<slice>.md
  - 02-<slice>.md
```

Omit `parent` / `spec_only` when there is a single implementable PRD.

Use `.md` only. Never emit PDF.

## After the pack exists

Tell the user the paths. Optional next skills (only if they ask): `scheduling-prds`, then `orchestrating-prd-cycle`. Or they can run `planning-from-prd` on one child.

## Red flags — stop and ask

- You are about to invent a name, role, entity, or screen the user did not confirm
- Two options are both plausible and you picked one silently
- The story only works with jargon
- You wrote code, SQL, or a finished UI
- You started planning files or coding
