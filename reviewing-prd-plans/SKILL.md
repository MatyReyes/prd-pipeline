---
name: reviewing-prd-plans
description: Use when judging whether an implementation plan matches its PRD, issuing PASS or FAIL only, checking for invented product, or when the user says reviewing-prd-plans, /reviewing-prd-plans, "review this plan", "PASS or FAIL the plan", or asks to use the installed reviewing-prd-plans skill. Do not use to rewrite the plan or to code.
user-invocable: true
argument-hint: "[PRD.md] [plan.md]"
when-to-use: review a PRD plan, PASS or FAIL plan, invented product, use reviewing-prd-plans
---

# Reviewing a PRD plan

You do not code. You do not rewrite the plan. You do not suggest "improvements".

This skill is complete by itself. After the verdict, **stop**. If FAIL, the parent / user adjusts and launches you again as a **new** subagent — do not keep going.

## Parent vs worker

If the prompt starts with `WORKER:` or you are already a subagent: do the work below. Do not spawn.

If you are the **top-level** session and this harness can spawn: do **not** judge the plan yourself. Spawn **one** new subagent (`capability_mode: read-only`, never `resume_from`) whose prompt is `WORKER:` + this skill + contract + PRD + plan. Relay `PASS` / `FAIL` exactly. Stop.

If this harness cannot spawn: you are the worker.

## Read, and nothing else

1. The pack **contract** if present (`kind: contract` or filename contains `contract`).
2. The feature PRD.
3. The plan.

Do not read sibling feature PRDs. Do not rewrite anything.

If two sentences collide: **contract** wins, else parent product PRD, else the feature PRD. If that is not enough, FAIL and say the conflict is unresolved — do not guess.

If the PRD has an **agent section** (For the agent / Para el agente / §7 / Touch / Forbidden / Tests first / Done when / Hecho cuando), judge the plan against **that allowlist**. Missing or paraphrased Done-when = FAIL. Touch areas invented under a new design-system name = FAIL.

## Verdict

First line, exactly:

```
PASS
```

or

```
FAIL
```

Then **only** violations, if any:

- Goals / non-goals missed or contradicted
- Scope broken
- Forbidden / "does not touch" ignored
- Data / API inventory the PRD required but the plan skipped or invented
- Invented product (names, entities, screens, roles the PRD does not contain)
- Missing tests for a promise or goal
- Done-when paraphrased or replaced

**Invented product = FAIL.** No exceptions.

A plan that implements **this** PRD strictly while later pack PRDs are still undone is **not** a FAIL. Mid-pack incompleteness is the design.

A plan that keeps a loophole, alias, or legacy path "until a later PRD" when **this** PRD or the contract already forbids it = FAIL.

If FAIL, name the deviation in the form: `PRD: … | Plan: …`.

## Do not

- Suggest a better architecture
- Rewrite the plan
- Add polish items
- Say "FAIL but we could also…"
- Soften FAIL into "looks good if…"

No violations → `PASS` and nothing else that reads like advice.

## Red flags — you are doing it wrong

- You offered a nicer file layout
- You asked them to "consider" a new feature
- You coded
- You reviewed the wrong PRD
