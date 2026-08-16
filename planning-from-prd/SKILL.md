---
name: planning-from-prd
description: Use when writing an implementation plan from an existing PRD markdown, listing files and tests without coding, discussing edge cases before burning tokens, or when the user says planning-from-prd, /planning-from-prd, "plan this PRD", "don't code, plan", or asks to use the installed planning-from-prd skill. Do not use to write PRDs or to implement.
user-invocable: true
argument-hint: "[path to PRD.md]"
when-to-use: plan from a PRD, implementation plan, edge cases before coding, use planning-from-prd
---

# Planning from a PRD

You write a plan for **one** PRD. You do not code. You do not reopen the product. You do not invent a design system with a new name.

This skill is complete by itself. After the plan file exists (and the user has settled edge cases), **stop**.

## Pack roles

In the same folder as the feature PRD (optional `_pack.md` / `_schedule.md`):

| Role | How you recognize it | What you do |
|---|---|---|
| **Contract** | `kind: contract` or filename contains `contract` | Conflict authority. Read it. Never implement it. |
| **Standard** | `kind: standard` or filename contains `standard` | How to *read* a PRD. Not product. Do not implement. |
| **Parent** | `kind: prd` that is spec-only / named `parent:` in `_pack.md` | Product fence if there is no contract. |
| **Feature** | The one PRD you were named | The only thing you plan. |

Do not read sibling **feature** PRDs "for context".

## Read, in this order, and nothing else

1. The **contract**, if the pack has one.
2. The **standard**, if the pack has one (anatomy only).
3. The **parent**, only if there is no contract and `_pack.md` / the user names one.
4. The **one** feature PRD the user named.
5. The repo areas that PRD's Scope, data section, and agent section point at.

If the user did not name a PRD, list implementable PRDs and ask.

## Authority

If two sentences collide:

1. **Contract** wins.
2. Else the **parent** PRD wins.
3. Else the feature PRD stands.

If that is not enough, **stop and ask**. Do not guess.

## Agent section (allowlist)

If the feature PRD has a section for the implementing agent — headings such as "For the agent", "Para el agente", "§7", or blocks named Touch / Forbidden / Tests first / Done when / Hecho cuando — **that section is the allowlist**.

- **Touch** → files / areas the plan may name. Do not invent a design-system name the PRD did not use.
- **Forbidden** / do not touch → the plan's "will not touch".
- **Tests first** → required tests; add only what the repo still needs to *prove* them.
- **Done when** / Hecho cuando → **quote verbatim**. Do not paraphrase.

If there is no such section, derive the same four things from Scope, the data section, goals, non-goals, and promises.

## Skip if already planned

If `<pack>/plans/<prd-stem>.plan.md` already exists, **do not overwrite it**. Tell the user it exists. Only rewrite if they explicitly say to replan.

## Edge cases before the plan file

A missed edge case is either a bad assumption or a cheap discovery. Surface it **before** you write the plan.

1. Pull candidates from the PRD (WHEN / conditions / locks / promises / agent section) **and** from the repo.
2. Present each: in scope / out of scope / needs a PRD change.
3. Wait for the user.

If you are a mute subagent (orchestrated): output a heading `EDGE_CASES` and **stop**. Do not write the plan file unless the prompt contains `CONFIRMED_EDGES` with the user's decisions. Never resume an earlier planner; the parent will spawn you again.

An edge case does not sneak in as "an implementation detail".

## What the plan must contain

Write **`<pack-dir>/plans/<prd-stem>.plan.md`**.  
Example: feature `platform_admin/prds/01-caparazon.md` → `platform_admin/prds/plans/01-caparazon.plan.md`.  
Do not invent `prds/<slug>/` if the pack already lives somewhere else.

- **Files to create/touch** — agent-section Touch, else Scope + data, mapped onto **this** repo.
- **Step order**
- **Tests first** — agent-section tests + gaps the repo must prove. Name files and the command.
- **What you will not touch** — agent-section Forbidden + non-goals + Scope "does not" + contract / parent fences. **Do not implement sibling PRDs.**
- **Done when** — quoted from the agent section, else from goals and promises.

The PRD is not code. The plan is the first place files, tests, and order appear.

## Do not

- Code
- Change the PRD, the contract, or the standard
- Invent product
- Implement a sibling PRD
- "Improve" the design
- Leave TBD, "add tests later", or "handle edge cases"

## After

Tell the user the path. Optional next skills: `reviewing-prd-plans`, then `executing-prd-plan`.

## Red flags — stop

- You are reaching for a file editor to implement
- You assumed an edge case
- You planned work Forbidden / Scope said not to touch
- You cited a Done-when the PRD does not contain
- You overwrote a plan nobody asked to replan
