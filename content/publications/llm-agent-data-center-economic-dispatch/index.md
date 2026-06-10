---
title: "Multi-Agent Large Language Model-Based Economic Dispatch Framework for Optimal Energy Management of Data Center Microgrids"
authors:
  - me
  - Changhyun Jeon
  - Cheongju Noh
date: "2026-01-01T00:00:00Z"
weight: 20
publishDate: "2026-01-01T00:00:00Z"
publication_types: ["paper-conference"]
peer_reviewed: false
featured: false
reading_time: true
math: true
image:
  preview_only: true
links:
  - name: Live Demo
    url: https://datacenteredagent-dbdhwlxmcal5b5vyjlbk4k.streamlit.app
summary: "A multi-agent LLM framework that turns natural-language prompts into optimal economic dispatch for data center microgrids, validated on a 400 MW MIT TX-GAIA load profile."
tags:
  - LLM Agent
  - Energy Management
  - Economic Dispatch
  - Data Center Microgrid
  - Multi-Agent System
---

---

<div style="display:grid;grid-template-columns:160px 1fr;gap:1.5rem 2rem;margin-bottom:2rem;">

<div style="font-weight:700;padding-top:.1rem;">Abstract</div>
<div>

The rapid expansion of artificial intelligence (AI) workloads and cloud computing drives a significant rise in hyperscale data centers. This growth in power demand, coupled with extreme load volatility inherent to AI operations, severely destabilizes power grids and complicates energy management. Consequently, data centers increasingly evolve into localized microgrids equipped with distributed energy resources (DERs) and energy storage systems (ESS). However, conventional economic dispatch models—typically based on Mixed-Integer Linear Programming (MILP)—demand specialized optimization expertise and make it difficult to respond flexibly to dynamic business environments, often requiring time-consuming model redesigns whenever tariffs (e.g., Time-of-Use rates) or environmental regulations (e.g., Renewable Energy Certificates) change.

To address these challenges, this paper proposes a **multi-agent large language model (LLM)-based economic dispatch framework** for data center microgrids. The model operates through a **four-stage sequential agent architecture** that lets operators execute complex optimization tasks from natural-language inputs. The efficiency of the framework is validated through simulations using the MIT TX-GAIA data center load profile scaled to 400 MW, configured with small modular reactors (SMR), gas turbines (GT), photovoltaics (PV), and a BESS.

</div>

<div style="font-weight:700;padding-top:.1rem;">Type</div>
<div>Conference</div>

<div style="font-weight:700;padding-top:.1rem;">Publication</div>
<div>To be submitted to <em>CIRED 2026</em></div>

<div style="font-weight:700;padding-top:.1rem;">Demo</div>
<div><a href="https://datacenteredagent-dbdhwlxmcal5b5vyjlbk4k.streamlit.app" target="_blank" rel="noopener">Interactive Streamlit app ↗</a></div>

<div style="font-weight:700;padding-top:.1rem;">Keywords</div>
<div>LLM agent · economic dispatch · data center microgrid · energy management · multi-agent system</div>

</div>

---

## Motivation

AI workloads are pushing hyperscale data centers toward extreme, volatile power demand that destabilizes grids. To stay reliable and cost-efficient, data centers are becoming **localized microgrids** with distributed energy resources and storage. But the conventional way to operate them has real barriers:

- **High expertise barrier:** MILP-based economic dispatch requires specialized optimization and modeling skills that facility operators rarely have.
- **Rigid to change:** Whenever tariffs (Time-of-Use rates) or regulations (Renewable Energy Certificates) change, the entire model often needs a slow, complete redesign.
- **Slow scenario testing:** Validating "what-if" operational strategies is time-consuming under rigid formulations.

---

## System Configuration

The target is a **grid-connected hyperscale data center microgrid** sized for AI workloads.

| Component | Parameter | Value |
|:---|:---|:---|
| **Data Center Load** | Peak load | 400 MW (AI/HPC workload) |
| **SMR** | Unit capacity / ramp / cost | 100 MW · 0.75 MW/15 min · ≈ 7 USD/MWh |
| **Gas Turbine (GT)** | Unit capacity / ramp / fuel | 170 MW · 15 MW/min · LNG |
| **ESS** | Energy / power / efficiency / SOC | 160 MWh · 40 MW · 95% · 10–90% |

**Load modeling.** The MIT Supercloud *TX-GAIA* HPC dataset — chosen for its high volatility relative to commercial data centers — was normalized from its native 300 kW scale up to a 400 MW peak and resampled to a 15-minute resolution (96 steps over 24 hours):

$$P_{load}(t) = \frac{P_{raw}(t)}{P_{raw}^{max}} \times 400\ \text{MW}$$

**Generator modeling.** The GT uses a quadratic cost function $C_{GT}(P) = aP^2 + bP + c$ (derived via GasTurb at 800 KRW/kg LNG) capturing partial-load efficiency drop, with a 15 MW/min ramp limit. The SMR (SMART-100, 100 MW) operates as baseload at 7 USD/MWh with a strict 0.75 MW/15 min ramp limit for operational safety.

---

## Method Overview

We propose a **four-stage sequential multi-agent LLM pipeline** that converts a natural-language prompt into an optimized dispatch schedule — no manual model rebuilding required.

<figure style="margin:1.5rem auto;max-width:560px;">
  <img src="/uploads/papers/fig_llm_agent_architecture.png"
       alt="Four-stage multi-agent LLM architecture: parsing, formulation, solver, explanation"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    Four-stage agent architecture. An infeasibility-feedback loop returns from the solver to the formulation agent when constraints cannot be satisfied.
  </figcaption>
</figure>

1. **Parsing Agent** — extracts optimization parameters from the user's natural-language prompt.
2. **Formulation Agent** — translates the parameters into a formal mathematical model (objective + constraints).
3. **Solver Agent** — executes the computational dispatch and returns the optimal schedule.
4. **Explanation Agent** — synthesizes the numerical results into a comprehensive, human-readable strategic report.

A feedback loop returns infeasibility signals from the solver back to the formulation stage, enabling automatic correction without operator intervention.

---

## Mathematical Formulation

**Objective** — minimize total daily operational cost $J$ (base demand charge + generation + grid purchase − sales revenue + ESS degradation):

$$\min J = C_{base} + \sum_{t=1}^{T} \Big[ \sum_{i \in \mathcal{G}} C_{gen,i}(P_i(t)) + \rho_{grid}(t)P_{grid}(t) - R_{sales}(t) + \sum_{k \in \mathcal{E}} \lambda_{deg} P_{dis,k}(t) \Big]$$

**Key constraints:**

- **Power balance** — supply equals demand at every step $t$ (grid + generators + PV + ESS discharge = load + ESS charge).
- **ESS dynamics** — $SOC(t) = SOC(t-1) + \big(P_{chg}(t)\eta - P_{dis}(t)/\eta\big)\Delta t / E_{cap}$, with round-trip efficiency $\eta = 0.95$ and $10\% \le SOC(t) \le 90\%$.
- **Generator limits** — capacity bounds $P_{min,i} \le P_i(t) \le P_{max,i}$ and ramp limits $|P_i(t) - P_i(t-1)| \le Ramp_i$.

The Solver Agent casts this as a **Mixed-Integer Quadratic Program (MIQP)** and solves it with **Gurobi** for the global cost-minimizing optimum.

---

## Results

The framework was validated on the **MIT TX-GAIA data center load profile scaled to 400 MW** over a 24-hour horizon, with a portfolio of small modular reactors (SMR), gas turbines (GT), photovoltaics (PV), and a BESS.

<!-- <figure style="margin:1.5rem 0;">
  <img src="/uploads/papers/fig_llm_optimization_result.png"
       alt="Optimization result: cost-based dispatch schedule across generation sources over 24 hours"
       style="width:100%;border-radius:8px;border:1px solid rgba(148,163,184,.25);background:#fff;padding:.5rem;">
  <figcaption style="font-size:.8rem;color:#94a3b8;margin-top:.5rem;text-align:center;">
    Cost-based optimal dispatch over 24 hours. SMR provides stable baseload, while ESS and gas turbines are dispatched for peak shaving during expensive Time-of-Use periods.
  </figcaption>
</figure> -->

<div style="display:flex;flex-wrap:wrap;gap:1rem;margin:1.5rem 0;justify-content:center;">
  <img src="/uploads/papers/fig_report_page1.png"
       alt="Data Center Energy Dispatch Report — page 1: summary metrics and dispatch chart"
       style="flex:1 1 320px;max-width:48%;min-width:300px;display:block;border:1px solid rgba(148,163,184,.3);border-radius:10px;background:#fff;">
  <img src="/uploads/papers/fig_report_page2.png"
       alt="Data Center Energy Dispatch Report — page 2: cost, dispatch and TOU analysis"
       style="flex:1 1 320px;max-width:48%;min-width:300px;display:block;border:1px solid rgba(148,163,184,.3);border-radius:10px;background:#fff;">
</div>

Key findings:

- **SMR as baseload:** The framework seamlessly utilizes the SMR for stable baseload supply (~121 MW average, capacity factor ~100%).
- **Peak shaving:** ESS and gas turbines are dispatched strategically for peak shaving during expensive TOU periods.
- **Cost structure:** Total cost was driven primarily by dispatch strategy — baseload generation accounted for only ~2.5% of total cost, while variable operating cost dominated at ~90.9%, confirming that operational decisions matter more than raw asset mix.
- **Interactive testbed:** The pipeline serves as a rapid, interactive testbed for verifying strategies against uncertainties in cost, load dynamics, and grid stability — and scales to other load systems.

---
<!-- 
## Generated Report

The **Explanation Agent** automatically synthesizes the optimization output into a full technical report — dispatch chart, cost structure, dispatch strategy, TOU operation analysis, and recommendations. Below is an example report generated end-to-end by the agent pipeline:

<div style="display:flex;flex-wrap:wrap;gap:1rem;margin:1.5rem 0;justify-content:center;">
  <img src="/uploads/papers/fig_report_page1.png"
       alt="Data Center Energy Dispatch Report — page 1: summary metrics and dispatch chart"
       style="flex:1 1 320px;max-width:48%;min-width:300px;display:block;border:1px solid rgba(148,163,184,.3);border-radius:10px;background:#fff;">
  <img src="/uploads/papers/fig_report_page2.png"
       alt="Data Center Energy Dispatch Report — page 2: cost, dispatch and TOU analysis"
       style="flex:1 1 320px;max-width:48%;min-width:300px;display:block;border:1px solid rgba(148,163,184,.3);border-radius:10px;background:#fff;">
</div>

<div style="margin:1rem 0;">
  <a href="/uploads/papers/llm-data-center-energy-report.pdf" target="_blank" rel="noopener"
     style="display:inline-flex;align-items:center;gap:.5rem;background:rgba(59,130,246,.15);border:1px solid rgba(59,130,246,.4);border-radius:8px;padding:.5rem 1rem;font-size:.9rem;color:#60a5fa;text-decoration:none;font-weight:600;">
    📄 Download full report (PDF)
  </a>
</div> -->

---

## Future Work

- **Capacity optimization:** Move from fixed capacities to optimal sizing of GTs and ESS for a given load profile.
- **PV curtailment & RE100:** Add objective penalty terms for curtailed renewables and minimum-usage constraints to meet sustainability targets.
- **Conformal prediction:** Integrate distribution-free prediction intervals with guaranteed coverage into the demand-forecasting module, letting the Formulation Agent build **uncertainty-aware** robust optimization models that hedge against AI-workload volatility — reducing the risk of shortage or over-generation.

---

## Try the Live Demo

The full agent pipeline is deployed as an interactive web app. Enter a natural-language scenario and watch the agents parse, formulate, solve, and explain the optimal dispatch in real time.

<div style="margin:1rem 0;">
  <a href="https://datacenteredagent-dbdhwlxmcal5b5vyjlbk4k.streamlit.app" target="_blank" rel="noopener"
     style="display:inline-flex;align-items:center;gap:.5rem;background:rgba(34,197,94,.15);border:1px solid rgba(34,197,94,.4);border-radius:8px;padding:.55rem 1.1rem;font-size:.95rem;color:#4ade80;text-decoration:none;font-weight:600;">
    🚀 Launch the Streamlit App
  </a>
</div>

<!--more-->
