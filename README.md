<p align="center">
  <img src="docs/mark.svg" width="88" height="88" alt="PRD Pipeline" />
</p>

<h1 align="center">PRD Pipeline</h1>

<p align="center">
  <strong>Story first. Plan second. Code last. Review at every gate.</strong>
</p>

<p align="center">
  Seven agent skills that keep product decisions out of the code session —<br />
  and keep the coder from inventing the product.
</p>

<p align="center">
  <img alt="Skills" src="https://img.shields.io/badge/skills-7-0B1220?style=flat-square" />
  <img alt="License" src="https://img.shields.io/badge/license-MIT-5EEAD4?style=flat-square&labelColor=0B1220" />
  <img alt="Harness" src="https://img.shields.io/badge/Grok%20·%20Claude%20·%20Codex%20·%20OpenCode-ok-5EEAD4?style=flat-square&labelColor=0B1220" />
</p>

---

Each skill runs **alone**. Install one, or install all seven and let the orchestrator walk the pack.

They also wake up from ordinary sentences:

```
Would you like to use writing-prds that you have installed
to develop this functionality?
```

```
Use planning-from-prd on prds/checkout/01-wallet.md
Don't code.
```

```
/orchestrating-prd-cycle handoff juicio=grok-4.6
```

---

## The cycle

```mermaid
flowchart LR
  W["writing-prds"] --> S["scheduling-prds"]
  S --> O["orchestrating-prd-cycle"]
  O --> P["planning-from-prd"]
  P --> RP["reviewing-prd-plans"]
  RP -->|FAIL| P
  RP -->|PASS| C["executing-prd-plan"]
  C --> RC["reviewing-prd-code"]
  RC -->|FAIL| C
  RC -->|PASS| N["next PRD"]
```

| Skill | Unit of work | Writes | Must not |
|---|---|---|---|
| [`writing-prds`](writing-prds/SKILL.md) | The whole request | `*.md` PRDs + `_pack.md` | Assume, plan files, code |
| [`scheduling-prds`](scheduling-prds/SKILL.md) | The whole pack | `_schedule.md` | Rewrite PRDs, code |
| [`orchestrating-prd-cycle`](orchestrating-prd-cycle/SKILL.md) | The graph | `_run.md`, subagents / handoff packets | Code, reuse a subagent |
| [`planning-from-prd`](planning-from-prd/SKILL.md) | One PRD | `plans/<stem>.plan.md` | Code, invent product |
| [`reviewing-prd-plans`](reviewing-prd-plans/SKILL.md) | One PRD + its plan | `PASS` / `FAIL` | Rewrite, “improvements” |
| [`executing-prd-plan`](executing-prd-plan/SKILL.md) | One passed plan | The cut + evidence | Reopen the product |
| [`reviewing-prd-code`](reviewing-prd-code/SKILL.md) | This cut | `PASS` / `FAIL` | Code, polish nits |

Gates are hard:

- No code until the plan is **PASS**.
- FAIL on the plan → argue the deviation, ask the human, adjust, **new** reviewer.
- FAIL on the code → a **repair cut** (files only, no creative freedom) → code again → **new** reviewer.
- Every child is a **fresh** subagent. Never `resume_from`.

---

## What a PRD is

`writing-prds` owns the shape. It is not a PDF and it does not send the model to read one. The anatomy lives in the skill.

| # | Section | What it is |
|---|---|---|
| 0 | Header | Status, owner, created, **scope** (touches / does not, same line) |
| 1 | Summary | Today / After — two lines |
| 2 | The story | Before / After with a **name and a moment**. If it does not convince, stop |
| 3 | Goals / Non-goals | O1… / NO1… Later sections cite them |
| 4 | Today → After | The flow, drawn twice |
| 5 | The data | Entities, switch, lock — not shipping SQL |
| 6 | Pseudo-code | `WHEN` / condition / `THEN` / **Promises** |

A PRD fixes **structure**. It never contains final code, finished screens, or env.

Size decides how many files: a tweak is one short PRD; a new product is nested PRDs (the parent is spec-only).

The writer **asks**. There are no stupid questions. Real forks get pros / cons and one efficient recommendation — not a critique of everything.

---

## Install

Each folder is a standard [Agent Skill](https://agentskills.io) (`SKILL.md` + frontmatter). Copy the folders you want.

### One skill

```bash
git clone https://github.com/MatyReyes/prd-pipeline.git
cp -R prd-pipeline/writing-prds ~/.claude/skills/
# same folder name works for the other harnesses:
cp -R prd-pipeline/writing-prds ~/.grok/skills/
cp -R prd-pipeline/writing-prds ~/.codex/skills/
cp -R prd-pipeline/writing-prds ~/.agents/skills/
```

### All seven

```bash
git clone https://github.com/MatyReyes/prd-pipeline.git
for s in writing-prds scheduling-prds orchestrating-prd-cycle \
         planning-from-prd reviewing-prd-plans \
         executing-prd-plan reviewing-prd-code; do
  ln -s "$(pwd)/prd-pipeline/$s" ~/.claude/skills/$s
  ln -s "$(pwd)/prd-pipeline/$s" ~/.grok/skills/$s
done
```

Open a new session (or wait a few seconds — most harnesses reload skills from disk).

Then:

```
/writing-prds
```

or just say it:

```
Use the writing-prds skill you have installed to develop this functionality.
Ask me anything you need. Don't assume.
```

---

## Using one skill

You do not need the orchestrator.

| You say | What happens |
|---|---|
| *Use writing-prds to spec wallet holds* | Questions, then markdown PRDs |
| *scheduling-prds on prds/wallet* | `_schedule.md` — waves and collisions |
| *planning-from-prd prds/wallet/01-holds.md* | A plan. Edge cases first. No code |
| *reviewing-prd-plans* | First line is `PASS` or `FAIL` |
| *You are the coder. executing-prd-plan this plan* | Tests first, evidence on the way out |
| *reviewing-prd-code this cut* | First line is `PASS` or `FAIL` |

The coder may use **other skills you already have** (TDD, framework patterns) as *how* to implement. Those skills must not reopen the product. The plan wins if they collide.

---

## Already have PRDs?

Skip `writing-prds`. Point at the folder and start at scheduling:

```
/scheduling-prds platform_admin/prds
```

```
I already have the PRDs in platform_admin/prds.
Use scheduling-prds, then orchestrating-prd-cycle.
Handoff the coder. Don't rewrite the specs.
```

```
/orchestrating-prd-cycle handoff juicio=grok-4.6 pack=platform_admin/prds
```

`_pack.md` is optional. The scheduler reads the markdown that is already there, writes `_schedule.md` next to it, and stops. The orchestrator picks up from that graph: plan → review plan → code → review code, one **new** subagent per step.

A file with `kind: contract` (or `*contract*` in the name) **wins collisions**. `kind: standard` is anatomy, not product. Neither is implemented. Plans land in `<pack>/plans/<stem>.plan.md`. Already-shipped children go under `## Skip`. Code mode can differ **per PRD** (`handoff` vs `spawn`); the run has a default, not a single law for the whole pack.

If you only want one child:

```
/planning-from-prd platform_admin/prds/01-caparazon.md
```

---

## Using the full cycle

```
/orchestrating-prd-cycle
```

It asks **once** (or you pass args):

1. **Spawn** — the coder is a subagent in *this* harness, with a spawnable model.
2. **Handoff** — no coder subagent. You get a `Copy this:` block for Luna, OpenCode, another Grok window, anything.

Then it asks for the **judgment model** (plan + both reviews). The **coding model** is only asked in spawn mode. Models that this harness cannot spawn are not offered. Luna / OpenCode are handoff destinations, not spawn IDs.

```
/orchestrating-prd-cycle handoff juicio=grok-4.6
/orchestrating-prd-cycle spawn juicio=grok-4.6 codeo=grok-4.5
```

Choices land in `_run.md`. Every spawn passes `model=` from that file.

Handoff pause:

1. Plan review is `PASS`.
2. You get one `Copy this:` packet.
3. You code in the other conversation, in the same repo.
4. You come back and say the code is done.
5. A **new** code-review subagent reads the tree.

---

## Pack files

Default root: `prds/<slug>/`.

```
prds/<slug>/
  _pack.md                 # writing-prds
  _schedule.md             # scheduling-prds
  _run.md                  # orchestrating-prd-cycle
  00-product.md            # spec-only parent, if needed
  01-slice.md
  02-slice.md
  plans/
    01-slice.plan.md       # planning-from-prd
```

Nothing here is tied to a product, a stack, or a numbered “§7”. Sibling skills discover the pack; they do not hardcode your company.

---

## Why this exists

A model that is asked to “build X” will invent structure, then invent code to match. A model that is asked only to **tell the story** cannot hide behind a component tree. A model that is asked only to **execute a passed plan** cannot quietly reopen the product.

Split the jobs. Fresh context on every job. Evidence or it is not done.

---

## License

[MIT](LICENSE)
