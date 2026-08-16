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

If you are the **top-level** session and this harness can spawn: do **not** review the diff yourself. Spawn **one** new subagent (`capability_mode: read-only`, shell only for `git diff` / test output, never `resume_from`) whose prompt is `WORKER:` + this skill + contract + PRD + plan. Relay `PASS` / `FAIL` exactly. Stop.

If this harness cannot spawn: you are the worker.

## Read

1. The pack **contract** if present (`kind: contract` or filename contains `contract`).
2. The feature PRD.
3. The plan (or the repair cut) — file allowlist.
4. The diff for **this cut only** (`git diff` against the cut's base, or the files the plan named). Do not review the rest of the repo.

Do not read sibling feature PRDs.

If two sentences collide: **contract** wins, else parent product PRD, else the feature PRD. Unresolved → FAIL.

If the PRD has an **agent section**, Done-when / Forbidden / Touch come from there. Quote them. Do not substitute a vaguer goal.

## Questions (answer from the repo, not from prose)

1. Is every **Done when** (agent section if present, else goals / promises) **green with evidence in the repo** (tests that ran, not a paragraph)?
2. Are there files or behaviors **outside the plan** (or outside the repair cut)?
3. Did Forbidden / non-goals / "does not touch" get violated?

If an item cannot be demonstrated, it is not done.

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
