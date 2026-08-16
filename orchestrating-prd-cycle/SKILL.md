---
name: orchestrating-prd-cycle
description: Use when running one PRD step or the full pack (plan → review plan → code → review code), spawning a fresh subagent per launch, choosing spawn vs handoff-code, or when the user says orchestrating-prd-cycle, /orchestrating-prd-cycle, "plan this PRD", "review this plan", "review the code", "run the PRD cycle", "I already have the PRDs", "start from scheduling", or asks to use the installed orchestrating-prd-cycle skill. Do not use to write PRDs or to code yourself.
user-invocable: true
argument-hint: "[plan|review-plan|code|review-code] [prd or plan path] [handoff|spawn]"
when-to-use: orchestrate PRD cycle, spawn vs handoff, run all PRDs, use orchestrating-prd-cycle
---

# Orchestrating the PRD cycle

You are the parent session. You do not write PRDs. You do not code. You do not reuse a subagent.

**Existing PRDs are the normal start.** Do not run `writing-prds`. Do not rewrite markdown the user already accepted.

Only send them to `writing-prds` if there are **no** feature PRDs in the named folder (and they did not point at files).

## Solo or cycle

**Default: one launch = one new subagent.** You stay the parent. You never do the child's job.

```
/orchestrating-prd-cycle plan platform_admin/prds/02-sillas.md
/orchestrating-prd-cycle review-plan platform_admin/prds/02-sillas.md
/orchestrating-prd-cycle code platform_admin/prds/plans/02-sillas.plan.md handoff
/orchestrating-prd-cycle review-code platform_admin/prds/02-sillas.md
```

Same if they say “plan 02-sillas” / “review this plan” / “execute in Luna” / “review the code” while this skill is loaded.

| Step | Child skill | `capability_mode` | `model` |
|---|---|---|---|
| `plan` | `planning-from-prd` | `read-write` | `planning_model` |
| `review-plan` | `reviewing-prd-plans` | `read-only` | `planning_model` |
| `code` + spawn | `executing-prd-plan` | `all` | `coding_model` |
| `code` + handoff | none — print `Copy this:`, pause | — | — |
| `review-code` | `reviewing-prd-code` | `read-only` (shell only for `git diff` / tests) | `planning_model` |

After that **one** child returns, **stop**. Do not walk the next step or the next PRD unless they launch again (or they asked for the full pack).

Every child prompt starts with `WORKER:` and names only the files that role may read. Never `resume_from`. Never two roles in one child.

If they named a **pack** and no step, walk the schedule (section 3). If they named a **step**, do not walk the pack.

## 1. Resolve the pack

A pack is a directory of PRD markdown, or an explicit list of files the user named.

1. Use the path they gave (`pack=…`, `@prds/…`, `platform_admin/prds`, …).
2. Else look for `_pack.md` in the repo. If several, ask.
3. Else ask which folder. Do **not** invent a pack. Do **not** start `writing-prds` because `_pack.md` is missing.

`_pack.md` is optional. Without it, treat every `*.md` in that folder as a PRD except `_schedule.md`, `_run.md`, files under `plans/`, and obvious non-PRDs the user excludes (`README.md`, `LICENSE`, `00-standard.md` if they say it is a style guide not a product PRD).

If `_schedule.md` is missing, stale, or they said **start from scheduling**: run `scheduling-prds` on that folder first (in-session is fine). Then continue.

Skip spec-only (contract, standard, parent). Skip anything under `## Skip` or that the user named as already shipped. **Do not replan a feature whose plan already exists** unless they say replan. **Do not recode a feature they said is done.**

## 2. Ask how this run works (once)

If `_run.md` already exists for this paused run, reuse it. Otherwise ask, in this order, unless the slash args already answered:

1. **Default: where does code run?**
   - **Spawn** — a coder subagent in *this* harness.
   - **Handoff** — you print a `Copy this:` block. The user pastes it into another conversation (any model, any tool). You do not spawn a coder.
2. **Do any PRDs differ?** If yes, record `code: spawn` or `code: handoff` per file (in `_run.md` and/or the schedule). Auth / tenancy / "this one stays in this harness" is a reason to differ — only when **they** say so. Do not guess which children are Luna.
3. **Judgment model** — plan + both reviews. Only models **this harness can spawn**.
4. **Coding model** — only for PRDs that will **spawn**. Same spawnable list.

Write `_run.md` **in the pack folder**:

```markdown
mode: handoff
planning_model: grok-4.6
coding_model: grok-4.6
per_prd:
  02-sillas.md: spawn
  04-clientes-soporte.md: spawn
```

`per_prd` overrides `mode`. Unlisted implementable PRDs use `mode`.

Every spawn reads this file and **must** pass `model=` from it. Do not silently inherit the parent model when the user chose one. If the harness cannot spawn that model, stop and ask again.

Parse slash args when present, e.g. `handoff juicio=grok-4.6 pack=platform_admin/prds` or `spawn juicio=grok-4.6 codeo=grok-4.5`.

## 3. Walk the schedule

For each implementable PRD that is **not** skipped / already done, in **code-wave** order (plan-waves may fan out earlier):

### Plan wave (may `parallel`)

For each PRD in the wave, spawn a **new** subagent:

- Skill: `planning-from-prd`
- Prompt: pack folder + contract path if any + standard path if any + **that one** feature PRD. The pack is the product; do not re-ask locked cuts; do not soften this PRD because a later PRD is unfinished. `EDGE_CASES` only for real forks this PRD + contract cannot close. If that list is empty, write the plan. If a plan file already exists, do not spawn a planner unless they asked to replan.
- `model`: `planning_model`
- `capability_mode`: `read-write` (plan file only)

If it returns edge cases: surface them to the user, get decisions, spawn a **new** planner with `CONFIRMED_EDGES` (never resume the first).

### Review the plan (may `parallel`)

Spawn a **new** subagent:

- Skill: `reviewing-prd-plans`
- Prompt: contract if any + that PRD + that plan. Nothing else.
- `model`: `planning_model`
- `capability_mode`: `read-only`

**FAIL** → you argue the deviation (PRD said X, plan says Y). Ask: adjust the plan toward the PRD, adjust the PRD, or stop. Apply only what they chose. Spawn a **new** plan reviewer. Do not code.

**PASS** → continue.

### Code

Resolve mode for **this** PRD: `per_prd` → schedule `code:` → `_run.md` `mode`. If still unset, ask. Do not assume.

**Spawn:** new subagent, skill `executing-prd-plan`, prompt = the plan path (PRD + contract only if the plan is ambiguous), `model` = `coding_model`, `capability_mode` = `all`.

**Handoff:** do **not** spawn a coder. Print exactly one packet, then **pause**. Do not start the next PRD.

```
Copy this:

You are the coder. Do not reopen the product.
Execute 100% of this plan:
<absolute-or-repo path to .plan.md>

<paste the plan>

Tests first. Use installed implementation skills as how to code, never as what to build.
Done when every Done-when line from the PRD is green with evidence in the repo.
When finished, leave in the repo and return to the orchestrator chat with:
1. git diff --stat (plan files only)
2. the plan's test command and its output
3. the Done-when checklist, item by item, with evidence
```

When the user says coding is finished, continue. The reviewer reads the repo; they do not need a pasted diff if the code landed in this tree.

Code-parallel waves: spawn several coders only in **spawn** mode. Handoff is **one packet at a time**.

Dependents stay in the same working tree so they see the previous cut. Do not isolate a dependent in a fresh worktree.

### Review the code

Spawn a **new** subagent:

- Skill: `reviewing-prd-code`
- Prompt: contract if any + that PRD + this cut (diff / files the plan named)
- `model`: `planning_model`
- `capability_mode`: `read-only` (shell only if needed to read `git diff` / test output)

**PASS** → next PRD / next wave.

**FAIL** → you write a **repair cut**: the files and behaviors that must change to clear the failures. Immovable. No extra product. That cut is the new plan for `executing-prd-plan` (spawn or another `Copy this:`). Then a **new** code reviewer. If it FAILs again, another repair cut — never reopen the product.

## Isolation rules

| Forbidden | Why |
|---|---|
| `resume_from` | Decisions leak across roles and PRDs |
| One subagent, two roles | The reviewer becomes helpful |
| One subagent, two PRDs | The second PRD inherits the first |
| Coding before plan PASS | The plan is not the agreement |
| Starting dependents after FAIL | The floor is not green |
| You (the parent) coding or rewriting the product | You orchestrate |

Each child prompt is self-contained: "Read and obey skill \<name\>. These files only: …"

If a child skill is not installed, paste its contract into the prompt (PASS/FAIL shape, no-assume, tests first). Prefer installed skills.

## Red flags — stop

- You are about to code
- You reused a subagent
- You spawned a coder in handoff mode
- You offered Luna / OpenCode / another tool as a **spawn** model
- You continued past FAIL
