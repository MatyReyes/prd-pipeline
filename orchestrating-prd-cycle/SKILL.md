---
name: orchestrating-prd-cycle
description: Use when automating one PRD (plan → review plan → code → review code → commit) in one parent chat with cold subagents that die, asking which git repo before work, or when the user says orchestrating-prd-cycle, /orchestrating-prd-cycle, "run this PRD", "I already have the PRDs", or asks to use the installed orchestrating-prd-cycle skill. Do not use to write PRDs or to code yourself.
user-invocable: true
argument-hint: "[one PRD path] [handoff|spawn]"
when-to-use: orchestrate one PRD, one parent chat, ask git root, commit after PASS
---

# Orchestrating the PRD cycle

**One parent chat = one feature PRD.** Workers spawn, finish, die. This parent does not start AT05 after AT04.

You only route. You do not write PRDs, plans, reviews, or app code. If you catch yourself authoring a plan or judging PASS — **stop** and spawn.

## Harness — gates (ask, then wait)

Do **not** dump a paragraph and continue. Use a real question (the harness ask tool if you have one). **One question. Stop the turn.** No spawn until the answer is in.

### Gate A — which PRD

If they already named **one** feature PRD, skip.

If they named a pack or several files: ask which **one** PRD this chat will run. Options = implementable files, plus “next unfinished” if a schedule exists. Do not walk the pack in this parent.

### Gate B — which git

Find candidate repos: walk up from that PRD for `.git`, and list sibling directories one level up that have their own `.git` (e.g. `platform_admin/`, `mvp/`).

Ask: **Which git is the primary target for this PRD?** Show the candidate paths as options. Wait.

If they already passed `TARGET_GIT_ROOT=` and that path has `.git`, skip the ask.

That answer is `TARGET_GIT_ROOT`. Every spawn uses `cwd` = that path. Handoff `Copy this:` includes it.

A second repo (migration in `mvp/` while the pack lives in `platform_admin/`) is extra, named later at commit. It is not the primary cwd unless they pick it.

### Gate C — where code runs

If they already said `handoff` or `spawn`, skip.

Ask: **Code in this harness (spawn) or another chat (handoff)?** Wait.

Write `_run.md` in the pack folder (**settings only**):

```markdown
mode: handoff
planning_model: <id>
coding_model: <id>
target_git_root: <abs>
current: 05-dinero.md
```

Not memory. Not a worker. Workers never read this file.

Then run **this PRD only** (section Run).

## Run this PRD

Skip steps already done (plan exists and they did not say replan; they said plan is `PASS`; they said code is done).

1. Spawn **planner** (cold). No plan file yet → write it. Edge cases → show user, wait, spawn a **new** planner with `CONFIRMED_EDGES`.
2. Spawn **plan reviewer** (cold). `FAIL` → argue deviation, ask, wait, new reviewer. `PASS` → continue.
3. Code: **spawn** coder (cold) or print `Copy this:` and **pause**. Do not start another PRD.
4. When code is in the tree (or they say Luna finished): spawn **code reviewer** (cold). `FAIL` → repair cut, code again, **new** reviewer.
5. `PASS` → **Commit gate**. This parent then **stops**. Next PRD = **new chat**.

`step=plan` only if they explicitly want a single step.

## Cold worker

Before each spawn, show this card and obey it:

```
SPAWN <role>
cwd: <TARGET_GIT_ROOT>
resume_from: none
files: <closed list>
not in prompt: sibling feature PRDs, prior plans, prior reviews, this chat
```

| Role | Files in the worker prompt (only these) |
|---|---|
| Planner | contract (if any), standard (if any), **this** feature PRD, skill |
| Plan reviewer | contract (if any), **this** PRD, **this** plan, skill |
| Coder | **this** plan (PRD + contract only if the plan is ambiguous), skill |
| Code reviewer | contract (if any), **this** PRD, **this** plan, skill. May read earlier `plans/*.plan.md` **file lists only** to subtract a dirty prior cut |

```
WORKER:
TARGET_GIT_ROOT: <abs>
cd there.
Read and obey skill <name>.
Read only:
- <file>
Do not read other feature PRDs. Do not use the parent chat as context.
resume_from is forbidden.
```

`CONTAMINATED` → discard that worker, **respawn once** clean. Do not end the run. Second `CONTAMINATED` → stop and show both prompts.

## Copy this (handoff)

```
Copy this:

You are the coder. Do not reopen the product.
TARGET_GIT_ROOT: <abs>
cd there before git and tests.
Execute 100% of this plan:
<plan path>

Tests first. Use installed skills as how to code, never as what to build.
Done when every Done-when line is green with evidence.
When finished, return to the orchestrator chat with:
1. git diff --stat (this cut)
2. the plan's test command and its output
3. Done-when, item by item, with evidence
```

## Commit gate (after code review PASS)

Do not start another PRD. Do not say “done” without a commit **or** a blocked proposal.

1. In `TARGET_GIT_ROOT` (and any second repo the plan named): `git status`, `git diff --stat`.
2. Build a commit proposal:
   - **Include:** this plan’s allowlist + catalog-fixture tests + this `plans/<stem>.plan.md`
   - **Exclude:** files owned by an earlier pack plan still dirty (propose those as a **separate** commit, or say they must land first)
3. Ask (wait): **Commit this set?** Options: commit as proposed / change the file set / not now.
4. If yes: `git add` only that set, commit with a message that names the PRD (e.g. `AT05 dinero: …`). Push only if they asked.
5. If the tree cannot be committed cleanly (mixed cuts, two repos, secrets): **do not force it**. Say what is wrong, what you would change (split commits, commit AT01 first, leave `mvp/` for a second commit), and wait.
6. This parent is finished for product work.

## Isolation

| Forbidden | Why |
|---|---|
| Second feature PRD in this parent | One chat, one PRD |
| `resume_from` | Decisions leak |
| Parent writes the plan / verdict / code | You route |
| Worker prompt includes a sibling feature PRD | Contamination |
| Ending the run on first `CONTAMINATED` | Respawn clean |
| Next PRD before commit gate | Dirty tree, mixed reviews |

## Red flags — stop

- You started AT06 in the same chat as AT05
- You assumed the git root and spawned
- You printed a wall of text instead of asking Gate B
- You skipped the commit gate after PASS
- You committed files from another AT “to be helpful”
