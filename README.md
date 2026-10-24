# Leash Length: how long should an unattended agent task run?

**Live demo:** https://shriyashekatkar25-byte.github.io/leash-length/
**Gallery:** [APM Lab](https://shriyashekatkar25-byte.github.io/apm-lab/)

An APM Lab decision prototype by **Shriya Shekatkar** (UCLA, Statistics & Data Science minor) · UCLA Stats 102C Monte Carlo Methods.

## The launch that inspired it

On **October 21, 2026**, Databricks announced that **Genie Code tasks in Lakeflow Jobs are generally available**. A Genie Code task runs a natural-language prompt as an autonomous agent step inside a scheduled job: it reads upstream task outputs, calls tools, and returns a link to the resulting chat. Auto-approval is always on for job tasks, and the docs say to limit what an unattended prompt can reach. Retries and duration thresholds are set in the task's advanced settings.

- Release note: [Databricks platform release notes, October 2026](https://docs.databricks.com/aws/en/release-notes/product/2026/october)
- Docs: [Genie Code task for jobs](https://docs.databricks.com/aws/en/jobs/tasks/genie-code)

## The PM decision

**What default time limit (duration threshold) should a new agent task in a scheduled job get, and should it retry once on timeout?**

Agent runs vary a lot. A morning summary takes a couple of minutes, a compliance audit can take half an hour, and a small share of runs get stuck in a loop and never finish. If the limit is too short, we kill good work. If it's too long, a stuck agent blocks downstream tasks and burns compute.

- **Success metric:** delivered-answer rate ≥ 97%
- **Guardrails:** healthy runs killed ≤ 2% · p95 task wall-clock (including retry) ≤ 30 min · agent-minutes per 1k runs ≤ 6,000 · share of minutes burned by stuck runs ≤ 15%
- **Stakeholders:** data engineers, workspace admins, agent/AI platform engineering, jobs orchestration, security & governance

With the default assumptions (seed 102), the tightest limit that passes every guardrail is about **29 minutes with one retry**. It kills about 1.9% of healthy runs. With no limit, stuck loops would burn about half of all agent minutes. The memo on the page rewrites itself whenever you move a slider, and it switches to "hold" (per-task-type limits) when no single number works.

## What the prototype shows about product judgment

| Databricks APM skill | Where it shows up |
|---|---|
| Customer problem framing | A real failure story (a stuck nightly agent, and then self-imposed timeouts that cut audits off), turned into one decision with one metric and four guardrails |
| Prototyping | A live, seeded Monte Carlo pond with sliders, a sweep chart and KPI badges |
| SQL / Python | Illustrative SQL that fits the model inputs from run history, plus a NumPy inverse-CDF simulator |
| Data-driven decisions | Every limit from 2 to 60 min is scored on the same draws, and the recommendation is the tightest limit that passes |
| Clear writing | A director memo plus a one-sentence VP summary, both generated from the simulation |

## Methods

| Piece of the model | Course method (Stats 102C) | How it's used |
|---|---|---|
| Task type (summary / triage / ticket / audit) | Ch3: finite discrete sampling by partitioning [0,1] | One uniform picks the task slice |
| Number of tool steps | Ch3: geometric inverse CDF, K = ⌈ln U / ln(1−p)⌉ | The agent may finish after each step |
| Time per tool step | Ch3: exponential inverse CDF, T = −μ ln U | Many quick calls, a few slow ones |
| Stuck loops | Mixture (Ch5 idea) | A small share of runs never finish |
| Kill rate, p95, cost | Ch1: Monte Carlo estimates via sample proportions (LLN, SE ~ 1/√n) | 20,000 runs, swept over limits |

## Design baseline

Four pins from Shriya's Pinterest home feed, each redrawn from scratch in CSS/SVG/canvas (no pin images are used, embedded or traced):

- **Hero** ← [YoshieHirota](https://www.pinterest.com/pin/316096467574310818/): a hot-pink field, a lemon-yellow cosmic swoosh with a thin brown outline, and twinkling white five-point stars (palette and motifs only, no character).
- **Cards & buttons** ← [Pastry Halloween costume (pendingpersonality)](https://www.pinterest.com/pin/634515035051633288/): coquette cutout stickers with white borders on a cream tulle texture, a blush corset card with cherry-red buttons, a red bow, and chunky rounded display type.
- **KPI badges** ← [Abundance (My Lucky Numbers List)](https://www.pinterest.com/pin/1132092425111735571/): glossy enamel hot-pink "candy" badges with a specular highlight, and bold italic serif numerals.
- **Simulation room** ← [Sanjyot Dhote](https://www.pinterest.com/pin/863354191097234850/): deep-blue bowl water with lotus pinks and cream petals. Each simulated run is drawn as a floating petal.

---

Unofficial concept prototype by Shriya Shekatkar. All data is simulated. Not affiliated with any company.
