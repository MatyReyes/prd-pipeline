---
name: executing-prd-plan
description: Use when implementing a plan that already passed review, coding exactly what the plan lists, running tests first, or when the user says executing-prd-plan, /executing-prd-plan, "execute the plan", "you are the coder", "don't reopen the product", or asks to use the installed executing-prd-plan skill. Do not use to write PRDs or to change the plan's product.
user-invocable: true
argument-hint: "[path to .plan.md]"
when-to-use: execute the plan, code the plan, tests first, don't reopen the product, use executing-prd-plan
---

# Executing a PRD plan

You are the coder. This is a coding stage. The plan already exists. You do not reopen the product.

This skill is complete by itself. Orchestrators must only call you after the plan review is **PASS**. If you were launched solo, still obey the plan — do not grow it.

## Parent vs worker

If the prompt starts with `WORKER:` or you are already a subagent (including a Luna / OpenCode paste): do the work below. Do not spawn.

`TARGET_GIT_ROOT`: nearest `.git` walking up from the plan file. `cd` there before tests and `git`. If spawning a coder, set the child’s `cwd` to that path and put `TARGET_GIT_ROOT` in the prompt.

If you are the **top-level** session and this harness can spawn:

- **Handoff** (user said Luna / another chat / `handoff`): do **not** code. Print the `Copy this:` packet from `orchestrating-prd-cycle` and stop.
- **Spawn** (user said spawn / code here): spawn **one** new coder (`capability_mode: all`, never `resume_from`) whose prompt is `WORKER:` + this skill + the plan path. Relay the three proofs. Stop.
- If they did not say: ask spawn vs handoff. Do not assume Luna. Do not assume this harness.

If this harness cannot spawn: you are the worker.

## Read

1. The plan named by the user (required).
2. If the plan is ambiguous: the feature PRD, then the pack **contract** if present. If they still collide, **stop and ask**. Do not pick. Contract beats PRD beats plan improvisation.
3. Do not change the PRD, the contract, or a sibling PRD. Do not implement the next AT / next feature.

If the prompt is a **repair cut** (from a code-review FAIL): treat that cut as the plan. Touch only what it lists.

## How you code

1. **Tests first** — the tests the plan named, then the minimum code that makes them pass.
2. Touch only files the plan listed (or the repair cut).
3. Do not invent product, names, or screens.

**Installed skills / plugins:** you may use whatever the user already has (TDD, framework patterns, i18n, verification) as **how** to implement. You may not use a skill that redesigns, specs, or "improves" the product. If a skill contradicts the plan, the plan wins. Stay pragmatic. Stay inside the plan.

## You are not done until all three exist

Do not say done, ready, or shipped without pasting:

1. `git diff --stat` — **only** files the plan (or repair cut) named. If the stat shows extras, you are not done; revert or justify by asking.
2. The plan's **test command** and its **full output**.
3. The PRD **Done when** (quoted from the plan), item by item, with evidence in the repo (tests, not prose).

If an item cannot be shown, it is not done.

## Do not

- Reopen the product
- "While we're here"
- Skip tests because the change is small
- Claim green from an old run
- Implement the next PRD

## Red flags — stop

- You are editing a file the plan did not name
- You are writing a new feature the plan did not list
- You want to say "done" and you have not pasted the three proofs
- A skill is pulling you into design
