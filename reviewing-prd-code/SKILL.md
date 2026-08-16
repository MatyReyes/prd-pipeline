---
name: reviewing-prd-code
description: Use when judging whether this cut's code meets the PRD with evidence in the repo, issuing PASS or FAIL only, checking for work outside the plan, or when the user says reviewing-prd-code, /reviewing-prd-code, "review the code against the PRD", "is Done when green", or asks to use the installed reviewing-prd-code skill. Do not use to write code or suggest polish.
user-invocable: true
argument-hint: "[PRD.md] [plan.md]"
when-to-use: review PRD code, PASS or FAIL implementation, Done when evidence, use reviewing-prd-code
---

# Reviewing PRD code

You do not code. You do not suggest "improvements". You judge **this cut** against **this PRD**.

This skill is complete by itself. After the verdict, **stop**. FAIL goes back to a repair cut + `executing-prd-plan`, in a **new** subagent.

## Parent vs worker

If the prompt starts with `WORKER:` or you are already a subagent: do the work below. Do not spawn.

If the `WORKER:` prompt names a sibling **feature PRD** (not contract, not this PRD, not this plan, not earlier `plans/*.plan.md` file lists), reply `CONTAMINATED` and the extra path. Do not verdict. The parent respawns clean.

If you are the **top-level** session and this harness can spawn: do **not** review the diff yourself. Resolve `TARGET_GIT_ROOT` (nearest `.git` walking up from the plan, else the PRD, else the pack). Spawn **one** new subagent with `cwd` = that path, prompt `WORKER:` + `TARGET_GIT_ROOT:` + this skill + contract + PRD + plan (never `resume_from`). Relay `PASS` / `FAIL` exactly. Stop.

If this harness cannot spawn: you are the worker.

## Read

1. The pack **contract** if present (`kind: contract` or filename contains `contract`).
2. The feature PRD.
3. The plan (or the repair cut) — file allowlist.
4. The diff for **this cut only**. **You** run the evidence. A previous chat's paste is not enough and is not a FAIL by itself.

## You must run (do not skip)

`cd` to `TARGET_GIT_ROOT` if the prompt set it. Else walk **up from the plan file** (then the PRD) until you find `.git`. Do not use the process cwd if that walk finds another root (a parent folder of several repos is not the target).

If `git rev-parse` still fails, say so and **stop** — that is not a product FAIL; the parent must respawn with `cwd` set.

Then run, and read the output:

1. `git status` and `git diff --stat` (and `git diff --stat --cached` if needed).
2. **Split the tree:** this plan's allowlist vs earlier pack plans vs leftover. Do not treat the whole dirty folder as this cut.
3. The plan's **test command** (from the plan file). If the plan lists several paths, run those. Fresh output. Exit code 0 is not assumed.

This cut's Forbidden (“do not touch the previous shell”) means **new work in this cut** on those files. Uncommitted files a prior plan already listed are not a Forbidden hit.

Missing output from Luna / the coder is **not** a violation. You did not inherit their terminal. Run the commands.

Untracked files the plan said to **create** are in scope. Do not FAIL them for being untracked.

Do not read sibling feature PRDs.

If two sentences collide: **contract** wins, else parent product PRD, else the feature PRD. Unresolved → FAIL.

If the PRD has an **agent section**, Done-when / Forbidden / Touch come from there. Quote them. Do not substitute a vaguer goal.

## Questions (answer from the repo, not from prose)

1. Is every **Done when** (agent section if present, else goals / promises) **green with evidence in the repo** (tests that ran, not a paragraph)?
2. Are there files or behaviors **outside the plan** (or outside the repair cut)?  
   **Not a FAIL:** an *existing* test whose diff is only a fixture/import forced by a catalog/enum/CHECK this plan **did** change (old slug → new slug).  
   **Not a FAIL:** a file that belongs to an **earlier PRD in this pack** still sitting uncommitted. Open `plans/*.plan.md` (file lists only — do not reopen those products). If `01-….plan.md` already named `app/_ui/**` or `/equipo`, that is AT01 in a dirty tree, not this cut redrawing the shell.  
   **Still a FAIL:** new product this plan did not name, and no earlier plan in the pack owns it.
3. Did Forbidden / non-goals / "does not touch" get violated?

If you ran the command and it failed, or a Done-when has no test that passed, it is not done. If you **did not run** the command, you have no verdict — run it or stop. Do not FAIL the product for your missing shell.

## Verdict

First line, exactly:

```
PASS
```

or

```
FAIL
```

Then **only** violations:

- Which Done-when / goal / promise lacks evidence (cite the missing test or the failing command)
- Which file or behavior is outside the plan
- Which Forbidden / non-goal was hit

No violations → `PASS` and no advice.

## Do not

- Rewrite the code
- Propose a cleaner abstraction
- Ask for extra product
- Soften FAIL
- Review files this cut did not touch unless they prove a leak (out-of-plan edit)

## Red flags — you are doing it wrong

- You said "looks good" without a test command's output
- You listed nits
- You coded
- You reviewed a sibling PRD's files "while here"
