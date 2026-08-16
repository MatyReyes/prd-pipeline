---
name: scheduling-prds
description: Use when ordering existing PRDs, deciding what can run in parallel vs what must wait, building an execution graph, starting the cycle after PRDs already exist, or when the user says scheduling-prds, /scheduling-prds, "I already have the PRDs", "start from scheduling", "what can we parallelize", "dependency graph", or asks to use the installed scheduling-prds skill. Do not use to write PRDs or code.
user-invocable: true
argument-hint: "[path to pack]"
when-to-use: schedule PRDs, parallel vs dependent, execution graph, use scheduling-prds
---

# Scheduling PRDs

You read an entire PRD pack and write the execution graph. You do not rewrite PRDs. You do not code. You do not plan files.

This skill is complete by itself. After `_schedule.md` exists, **stop** unless the user asks to orchestrate.

## Inputs

1. Find the pack: the folder or files the user named, or `_pack.md` if they did not name one. If several folders look like packs, ask. **Existing PRDs without `_pack.md` are valid.**
2. Read **every** markdown that looks like a PRD. Skip `plans/`, `_run.md`, `_schedule.md`. Classify the rest:
   - `kind: contract` or filename contains `contract` → **contract** (spec-only)
   - `kind: standard` or filename contains `standard` → **standard** (spec-only)
   - parent / "do not implement this file alone" → **spec-only parent**
   - `README`, `LICENSE`, `LAUNCH` → not a PRD (skip)
   - remaining `kind: prd` → implementable unless the user marks it **skip** / already shipped
3. Do not rewrite those files. If a file is ambiguous, ask whether to skip it.

If the user says a feature is **already done** (plan PASS + code shipped, or "skip 01"), list it under `## Skip` — do not put it in a code wave.

## Infer edges (general rules)

**A depends on B** when A's Today / flow / data needs B's After or B's promises to already be true.

**A and B cannot share a code wave** when Scope or §5 touch the same entity or the same area.

**A and B may share a plan wave** even when they must code in series. Planning is cheap. Coding on the same area is not.

**Spec-only** when Scope or a non-goal says this file is not implemented alone (typical parent product PRD).

If it is not clear: **serialize** and say why, or stop and ask. Never "I think this can run together".

Optional frontmatter (`blocked_by`, `blocks`, `reads`) is a hint, not a substitute for reading the bodies. Body wins if they disagree.

## Output

Write `_schedule.md` **in the same folder as the PRDs** (next to `_pack.md` if it exists; do not require it):

```markdown
# Schedule

## Roles
contract: 00-contract.md
standard: 00-standard.md
parent: 00-product.md

## Spec-only
- 00-contract.md — authority; do not plan or code
- 00-standard.md — anatomy; do not plan or code
- 00-product.md — parent; do not plan or code this file

## Skip
- 01-slice.md — user: already shipped

## Plan waves
### Wave 1 (parallel)
- 02-slice.md
  unlocks: 03-slice.md
  touch: <entities/areas>

### Wave 2
- 03-slice.md
  needs: 02-slice.md

## Code waves
### Wave 1
- 02-slice.md
  code: spawn
  needs: (none remaining)

### Wave 2
- 03-slice.md
  code: handoff
  needs: 02-slice.md
  cannot-parallel: <shared entity or area>
```

`code: spawn | handoff` is optional per PRD. Fill it only when the user said so, or when you asked and they answered. Do not invent Luna vs Grok. If unset, the orchestrator uses the run default or asks.

Code waves are **stricter** than plan waves. Two PRDs in the same plan wave may still be sequential in code waves.

## After

Tell the user the path. Optional next skill: `orchestrating-prd-cycle`. They can also run `planning-from-prd` on any implementable PRD.

## Red flags — stop

- You scheduled a spec-only parent as implementable
- You marked two PRDs code-parallel while they share a mutable entity
- You guessed an order without reading a PRD
- You started writing a plan or code
