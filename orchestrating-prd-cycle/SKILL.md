---
name: orchestrating-prd-cycle
description: Use when automating the PRD cycle with fewer user steps (one command runs plan → review plan → code → review code), walking a pack or one remaining PRD, spawning a fresh subagent per role, choosing spawn vs handoff-code, or when the user says orchestrating-prd-cycle, /orchestrating-prd-cycle, "run the PRD cycle", "I already have the PRDs", "start from scheduling", or asks to use the installed orchestrating-prd-cycle skill. Do not use to write PRDs or to code yourself.
user-invocable: true
argument-hint: "[prd or pack path] [handoff|spawn] [skip=…]"
when-to-use: orchestrate PRD cycle, spawn vs handoff, run all PRDs, use orchestrating-prd-cycle
---

# Orchestrating the PRD cycle

You are the parent session. You do not write PRDs. You do not write plans. You do not review. You do not code. You do not reuse a subagent.

If you catch yourself opening `*.plan.md` to author it, or judging PASS/FAIL, or editing app code — **stop**. Spawn a worker. You only route.

**Existing PRDs are the normal start.** Do not run `writing-prds`. Do not rewrite markdown the user already accepted.

Only send them to `writing-prds` if there are **no** feature PRDs in the named folder (and they did not point at files).

## One command — fewer steps

The user used to open a chat per role. **This skill replaces that.** One launch. You stay the parent. You spawn a **new** child per role and chain them. They do not launch plan, then review, then code.

```
/orchestrating-prd-cycle platform_admin/prds/02-sillas.md handoff
/orchestrating-prd-cycle pack=platform_admin/prds handoff skip=01-caparazon
```

For **that PRD** (or each remaining PRD in the pack) you, without waiting for a new slash:

1. Spawn planner — unless a plan already exists and they did not say replan  
2. Spawn plan reviewer — skip if they already said the plan is `PASS`  
3. `PASS` → spawn coder **or** print `Copy this:` and **pause** (handoff)  
4. After code (or after they say the Luna code is done) → spawn code reviewer  
5. `PASS` → next PRD in the schedule. `FAIL` → repair cut → code again → new reviewer  

Every child is a **cold worker**. See **Cold worker** below.

## Cold worker

The parent may have just finished AT03. That memory **must not** enter the AT04 child.

Before each spawn, print this card (and obey it):

```
SPAWN <role>
cwd: <TARGET_GIT_ROOT>
resume_from: none
files: <closed list>
not in prompt: sibling feature PRDs, prior plans, prior reviews, this chat
```

**Closed file lists**

| Role | Files in the worker prompt (only these) |
|---|---|
| Planner | contract (if any), standard (if any), **one** feature PRD, this skill |
| Plan reviewer | contract (if any), **that** PRD, **that** plan, this skill |
| Coder | **that** plan (PRD + contract only if the plan is ambiguous), this skill |
| Code reviewer | contract (if any), **that** PRD, **that** plan, this skill. May *read* earlier `plans/*.plan.md` **file lists only** to subtract a dirty prior cut |

Prompt shape (copy, do not improvise):

```
WORKER:
TARGET_GIT_ROOT: <abs>
cd there.
Read and obey skill <name>.
Read only:
- <file>
- <file>
Do not read other feature PRDs. Do not use the parent chat as context.
resume_from is forbidden.
```

`_run.md` is never in that list except the parent reading settings. Do not attach `_run.md` to the worker.

If the harness spawn tool has `resume_from`, leave it unset. If it has `cwd`, set `TARGET_GIT_ROOT`.

If a child returns **`CONTAMINATED`**: do **not** stop the cycle and do not ask the user to start over. Respawn **immediately** — new worker, same role, closed file list only, `resume_from` unset. Max **once**. If the second child is still `CONTAMINATED`, stop and show both prompts (the bug is the parent). The first contaminated worker is dead; never resume it.

Skip whatever is already done (existing `PASS` plan, user said skip / shipped). Resume mid-cut: if the plan is `PASS` and they said “code in Luna”, start at step 3. Do not replan.

**Only** if they explicitly say `step=plan` / “solo el review” / one verb: do that **one** child and stop. That is the exception, not the product.

## Chain target (`cwd`)

The parent session’s folder is **not** the worker’s repo. A tree can hold several `.git` (`platform_admin/`, `mvp/`, …). Children do not guess.

**Resolve `TARGET_GIT_ROOT` once**, walking **up** from (first hit wins):

1. The named plan file  
2. Else the named feature PRD  
3. Else the pack folder  

The target is the nearest ancestor directory that contains `.git`.

Write it in `_run.md` as `target_git_root: <absolute path>`.

**Every spawn** must set the child’s `cwd` to `TARGET_GIT_ROOT` and put this in the prompt:

```
WORKER:
TARGET_GIT_ROOT: <absolute path>
cd there before git and tests.
```

Handoff `Copy this:` includes the same `TARGET_GIT_ROOT` (Luna must `cd` there).

If the plan also touches a second repo (e.g. one migration under `mvp/`), name that path; `git` there separately. The **primary** target stays the pack’s repo.

Spawning without `cwd=TARGET_GIT_ROOT` is a bug. A child that runs `git` at the parent folder and finds no repo has no product verdict — respawn with the right `cwd`.

## 1. Resolve the pack

A pack is a directory of PRD markdown, or an explicit list of files the user named.

1. Use the path they gave (`pack=…`, `@prds/…`, `platform_admin/prds`, …).
2. Else look for `_pack.md` in the repo. If several, ask.
3. Else ask which folder. Do **not** invent a pack. Do **not** start `writing-prds` because `_pack.md` is missing.

`_pack.md` is optional. Without it, treat every `*.md` in that folder as a PRD except `_schedule.md`, `_run.md`, files under `plans/`, and obvious non-PRDs the user excludes (`README.md`, `LICENSE`, `00-standard.md` if they say it is a style guide not a product PRD).

If `_schedule.md` is missing, stale, or they said **start from scheduling**: run `scheduling-prds` on that folder first (in-session is fine). Then continue.

Skip spec-only (contract, standard, parent). Skip anything under `## Skip` or that the user named as already shipped. **Do not replan a feature whose plan already exists** unless they say replan. **Do not recode a feature they said is done.**

## 2. Ask how this run works (once)

`_run.md` is **settings only** (`mode`, models, `target_git_root`, `per_prd`). It is not memory. It is not a worker. It is not AT03’s plan.

If `_run.md` exists, copy **those keys** if they still match what the user said. Then spawn a **new** planner anyway. Never `resume_from`. Never paste a prior PRD, prior plan, prior review, or parent transcript into the child.

If `_run.md` names another `current:` PRD, overwrite `current` for this launch. Do not keep the old child’s notes as instructions.

Otherwise ask, in this order, unless the slash args already answered:

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
target_git_root: /absolute/path/to/platform_admin
current: 04-clientes-soporte.md
per_prd:
  04-clientes-soporte.md: spawn
```

Do not put plans, reviews, or chat notes in this file. `notes:` if present is discarded by workers (they never read `_run.md`).

`per_prd` overrides `mode`. Unlisted implementable PRDs use `mode`.

Every spawn reads this file and **must** pass `model=` from it. Do not silently inherit the parent model when the user chose one. If the harness cannot spawn that model, stop and ask again.

Parse slash args when present, e.g. `handoff juicio=grok-4.6 pack=platform_admin/prds` or `spawn juicio=grok-4.6 codeo=grok-4.5`.

## 3. Walk the schedule

For each implementable PRD that is **not** skipped / already done, in **code-wave** order (plan-waves may fan out earlier):

### Plan wave (may `parallel`)

For each PRD in the wave, spawn a **new** subagent:

- Skill: `planning-from-prd`
- Prompt: `WORKER:` + `TARGET_GIT_ROOT` + pack folder + contract path if any + standard path if any + **that one** feature PRD. The pack is the product; do not re-ask locked cuts; do not soften this PRD because a later PRD is unfinished. `EDGE_CASES` only for real forks this PRD + contract cannot close. If that list is empty, write the plan. If a plan file already exists, do not spawn a planner unless they asked to replan.
- `cwd`: `TARGET_GIT_ROOT`
- `model`: `planning_model`
- `capability_mode`: `read-write` (plan file only)

If it returns edge cases: surface them to the user, get decisions, spawn a **new** planner with `CONFIRMED_EDGES` (never resume the first).

### Review the plan (may `parallel`)

Spawn a **new** subagent:

- Skill: `reviewing-prd-plans`
- Prompt: `WORKER:` + `TARGET_GIT_ROOT` + contract if any + that PRD + that plan. Nothing else.
- `cwd`: `TARGET_GIT_ROOT`
- `model`: `planning_model`
- `capability_mode`: `read-only`

**FAIL** → you argue the deviation (PRD said X, plan says Y). Ask: adjust the plan toward the PRD, adjust the PRD, or stop. Apply only what they chose. Spawn a **new** plan reviewer. Do not code.

**PASS** → continue.

### Code

Resolve mode for **this** PRD: `per_prd` → schedule `code:` → `_run.md` `mode`. If still unset, ask. Do not assume.

**Spawn:** new subagent, skill `executing-prd-plan`, prompt = `WORKER:` + `TARGET_GIT_ROOT` + the plan path (PRD + contract only if the plan is ambiguous), `cwd` = `TARGET_GIT_ROOT`, `model` = `coding_model`, `capability_mode` = `all`.

**Handoff:** do **not** spawn a coder. Print exactly one packet, then **pause**. Do not start the next PRD.

```
Copy this:

You are the coder. Do not reopen the product.
TARGET_GIT_ROOT: <absolute path>
cd there before git and tests.
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
- Prompt: `WORKER:` + `TARGET_GIT_ROOT` + contract if any + that PRD + this cut. **Run** `git` and the plan's test command **there**. Do not FAIL because a prior session's paste is missing.
- `cwd`: `TARGET_GIT_ROOT`
- `model`: `planning_model`
- `capability_mode`: `read-only` plus shell for git and tests

**PASS** → next PRD / next wave. Tell the user to **commit this cut** (or otherwise isolate it) before the next PRD so the next review is not a mixed tree. Do not commit for them unless they asked.

**FAIL** → you write a **repair cut**: the files and behaviors that must change to clear the failures. Immovable. No extra product. That cut is the new plan for `executing-prd-plan` (spawn or another `Copy this:`). Then a **new** code reviewer. If it FAILs again, another repair cut — never reopen the product.

## Isolation rules

| Forbidden | Why |
|---|---|
| `resume_from` | Decisions leak across roles and PRDs |
| Feeding the child this session’s earlier PRD | The parent remembers AT03; the worker must not |
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
- You “reused _run.md” as if it were the planner
- The worker prompt mentions a sibling feature PRD
- You treated `CONTAMINATED` as the end of the run instead of respawning clean
- You spawned a coder in handoff mode
- You offered Luna / OpenCode / another tool as a **spawn** model
- You continued past FAIL
