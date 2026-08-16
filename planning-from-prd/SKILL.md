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

## Parent vs worker

If the prompt starts with `WORKER:` or you are already a subagent: do the work below. Do not spawn.

If the `WORKER:` prompt names a **sibling feature PRD** (another `NN-*.md` that is not contract/standard/parent), **stop**. Say `CONTAMINATED`. Do not write a plan. The parent must respawn with a closed file list.

If you are the **top-level** session and this harness can spawn: do **not** write the plan yourself. Resolve `TARGET_GIT_ROOT` (nearest `.git` up from the PRD). Spawn **one** new subagent (`cwd` = that path, `capability_mode: read-write`, never `resume_from`) whose prompt is `WORKER:` + `TARGET_GIT_ROOT:` + this skill + the named PRD / pack paths. Relay `EDGE_CASES` or the plan path to the user. Stop.

If this harness cannot spawn: you are the worker.

## Pack roles

In the same folder as the feature PRD (optional `_pack.md` / `_schedule.md`):

| Role | How you recognize it | What you do |
|---|---|---|
| **Contract** | `kind: contract` or filename contains `contract` | Conflict authority. Read it. Never implement it. |
| **Standard** | `kind: standard` or filename contains `standard` | How to *read* a PRD. Not product. Do not implement. |
| **Parent** | `kind: prd` that is spec-only / named `parent:` in `_pack.md` | Product fence if there is no contract. |
| **Feature** | The one PRD you were named | The only thing you plan. |

Do not read sibling **feature** PRDs to redesign them. The pack already decided the product. This cut only implements **this** PRD.

## The pack is the product

The feature PRDs in the folder were written together. The product is finished when **the last implementable PRD in the pack** is done — not when this cut ships.

A mid-pack cut may look useless, break an old URL, or leave a screen empty until a later PRD. **That is the design.** Do not ask to keep a loophole, alias, or legacy path "until two more iterations make it useful." Do not implement a sibling PRD to make this cut feel complete.

Earlier feature PRDs (and their plans, if any) are **locked**:

- Their Done-when, Forbidden, and "this is the next PRD" deferrals are not questions.
- Work they deferred **to this file** (proxy, `can()`, sillas, …) is **this plan's job**. Do it. Do not re-ask whether to do it.
- Do not re-litigate their routes, names, chrome, or nav.

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

Most "edge cases" are already decided. Put them in the plan. Do not quiz the user.

**Write into the plan (do not ask):**

- In scope — the PRD / agent section already says to do it
- Out of scope / Forbidden — already named; list under "will not touch"
- Locked by an earlier PRD in this pack
- Deferred **to this** PRD by an earlier cut
- "If we close this, it will not be useful until PRD N+2"
- "The old URL / old role / old page will break until later"
- Softening this PRD so the mid-pack app still feels finished

**Ask only if this PRD + contract still cannot choose:**

1. Two options both allowed (e.g. 404 **or** home) and neither file picks one.
2. This PRD requires something Forbidden / the workspace makes physically impossible — you need an explicit authorize (one exception), not a redesign.
3. Contract vs this PRD still collide after the authority rule.

If that list is **empty**, write the plan. No questionnaire.

If you must ask: number only the real forks. One line each. No tables of "in scope / out of scope."

Mute subagent: output `EDGE_CASES` **only** for those real forks, then **stop**. If there are none, write the plan. Never resume an earlier planner.

## What the plan must contain

Write **`<pack-dir>/plans/<prd-stem>.plan.md`**.  
Example: feature `platform_admin/prds/01-caparazon.md` → `platform_admin/prds/plans/01-caparazon.plan.md`.  
Do not invent `prds/<slug>/` if the pack already lives somewhere else.

- **Files to create/touch** — agent-section Touch, else Scope + data, mapped onto **this** repo.
- **Catalog fallout (a class, not a closed grep)** — if this cut changes a shared catalog (roles, slugs, CHECK, enum, union), the allowlist includes **existing tests that fixture the old values**. State the class in the plan (`any test still using theater slugs`, etc.). Edits there are fixture-only. Do not pretend you listed every file.
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
- You assumed a real fork (404 vs home, authorize Forbidden, unresolved contract)
- You asked whether to stay compatible until a later PRD
- You re-asked a decision an earlier PRD in this pack already locked
- You planned work Forbidden / Scope said not to touch
- You cited a Done-when the PRD does not contain
- You overwrote a plan nobody asked to replan
