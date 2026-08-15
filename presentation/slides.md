# Apex Autonomous AI Procurement Agent
## Engineering Approach, Hybrid Architecture & Production Roadmap
**Role**: Forward Deployed Engineer (Reflection AI)  
**Client**: Apex Manufacturing (Head of Operations & Engineering Review)  
**Date**: August 2026 | Milestone Interim Prototype Review (45-50 min)

---

## Slide 1: Title & Approach Overview
### Forward Deployed Engineering Methodology
- **The Problem**: Automate manual BOM explosion, stock netting, and policy-compliant purchasing without human intervention.
- **The 4-Step Engineering Journey**:
  1. *First Principles*: Understanding why pure LLM prompting fails in manufacturing.
  2. *Hybrid Architecture*: Isolating 100% deterministic math & policy gatekeeping from probabilistic reasoning.
  3. *Systematic Diagnosis*: Taxonomizing baseline loss patterns via an automated 66-run evaluation harness.
  4. *Data-Driven Hill Climbing*: Versioned prompt engineering driving accuracy from 94.7% to 99.4%.

---

## Slide 2: Step 1 — First Principles Problem Decomposition
### Why Naive LLM Wrappers Fail in Manufacturing
- **Failure Modes of Pure Prompting**:
  - *Arithmetic Drift*: LLMs hallucinate subtle multiplication and float errors during BOM explosion.
  - *Policy Dilution*: Temporal management memos (e.g. `MEMO-072` expiry) get ignored as context length increases.
  - *Stochastic Risk*: Unapproved or uncertified suppliers accidentally selected due to prompt nuances.
- **The FDE Engineering Insight**:
  - *Separate Precision from Reasoning*: Exact math and ASL filtering must be **100% deterministic (0% error rate)** in pure Python.
  - *Empower LLMs Where They Excel*: Multi-supplier volume splits, domestic preference trade-offs, defensible rationales, and proactive risk alerts.

---

## Slide 3: Step 2 — Core Engineering Design
### The Hybrid Deterministic-LLM Architecture
- **Layer 1: Deterministic Core (Pure Python)**:
  - `DemandAnalyzer`: Explodes BOM, deducts warehouse stock and in-flight POs, computes deadlines with zero float error.
  - `PolicyEngine`: Pre-filters unapproved or uncertified vendors before the LLM sees them—preventing policy leaks by construction.
- **Layer 2: Pluggable Reasoning Layer (LLM Engine)**:
  - `ProcurementPlanner`: Assembles pre-qualified candidate context, evaluates multi-vendor split rules (50% magnet cap), and generates auditable rationales.
  - `_plan_deterministic()` Fallback: Seamless offline execution if the cloud API disconnects.

---

## Slide 4: Step 3 — Pluggable AI Stack Strategy
### Zero Vendor Lock-In & Reflection Open Model Compatibility
- **Factory Pattern (`src/llm/factory.py`)**:
  - Decoupled provider interface: switch between Google Gemini cloud and on-premise **Reflection-70B** / vLLM via a single environment variable (`LLM_PROVIDER=open_model`).
- **Native Type Safety**:
  - Uses native SDK structured output (`response_schema=ProcurementPlanSchema`), guaranteeing 0% JSON syntax or parsing crashes.
- **Observability Flight Recorder**:
  - Tracks prompt tokens, completion tokens, internal **Thinking Tokens (`thoughts_token_count`)**, and latency (ms) per step into `trajectories/`.

---

## Slide 5: Step 4 — Rigorous Evaluation Harness
### Building the Dual Evaluation Engine Across 11 Scenarios
- **Part I: Official Canonical Scenarios (01–06)**:
  - Base operational workflows: 4-order netting, in-flight POs, rush deadlines, depleted stock, competing demand contention.
- **Part II: Synthetic Robustness Sanity Checks (07–11)**:
  - Boundary stress tests: Sole-source monopolies, $110k+ budget escalation, expired air freight date boundary, extreme 500-vs-8 MOQ ratio, and 70% cheaper blacklisted supplier trap.
- **Dual Scoring Framework**:
  - 60 Points Deterministic (Coverage, MOQ, ASL certs, memos, dates) + 40 Points Qualitative LLM Judge (Rationales & Alerts).

---

## Slide 6: Step 5 — Error Diagnosis & Loss Taxonomy
### Diagnosing Baseline Failure Modes (v1.0 Analysis)
- Documented systematically in [`reports/v1.0_loss_pattern_analysis.md`](../reports/v1.0_loss_pattern_analysis.md) (Baseline Score: 94.7%):
  1. *MOQ Excess Holding Risk (Scenario 10: 78.0%)*: Agent ordered 500 units for 8 needed, but omitted an alert quantifying the $28,536 excess carrying cost.
  2. *Multi-Order Delay Aggregation (Scenario 05: 89.5%)*: Agent flagged the earliest shortfall, but failed to itemize per-order delivery delays across all 4 customer POs.
  3. *Implicit Compliance Logging*: Documented sole-source justifications inside PO rationale fields, but omitted standalone Section 4 operational risk alerts.

---

## Slide 7: Step 6 — Systematic Prompt Versioning & Hill-Climbing
### Data-Driven Optimization (v1.0 ➔ v1.1)
- **Implementation in `src/llm/prompts.py`**:
  - Injected 7 explicit Alert synthesis directives into the prompt registry without mutating underlying Python math.
- **Benchmark Elevation**:
  - Scenario 10 (Large MOQ): **78.0% $\rightarrow$ 100.0% (+22.0 pts)**
  - Scenario 05 (Competing Demand): **89.5% $\rightarrow$ 100.0% (+10.5 pts)**
  - Scenario 07 (Sole Source): **94.5% $\rightarrow$ 100.0% (+5.5 pts)**
  - **Canonical Score**: **97.3% $\rightarrow$ 99.6% (597.5/600)**
  - **Overall Score**: **94.7% $\rightarrow$ 99.4% (1,093.5/1,100)**

---

## Slide 8: Step 7 — Multi-Model Pareto Frontier
### Empirical Exploration Across 66 Total Benchmark Runs
- **Benchmark Findings**:
  - `gemini-3.7-flash` (strict_compliance): **99.4%** in **44.7s** (44.8k thinking tokens) — *Optimal Pareto Frontier*.
  - `gemini-3.7-flash` (balanced): **99.2%** in **29.1s** (44.2k thinking tokens).
  - `gemini-3.1-pro-preview`: **98.9%** in **154.1s** (131.7k thinking tokens).
  - `gemini-3.5-flash-lite`: **97.6%** in **29.0s** (5.4k thinking tokens) — *Extreme Cost Efficiency*.
- **Takeaway**: With a robust deterministic core, Flash-tier models achieve superior speed and accuracy at 1/3 the latency of Pro models.

---

## Slide 9: Step 8 — Operational Readiness & Live Demo
### Sub-15s Parallel Benchmark & Trajectory Telemetry
- **Live Terminal Execution**:
  ```bash
  python3 run_all_and_eval.py   # All 11 scenarios parallel in 13.9s
  python3 leaderboard.py        # Multi-model matrix benchmark
  ```
- **Blackbox Telemetry**:
  - Complete prompt text, raw model JSON responses, and token consumption preserved in `trajectories/`.

---

## Slide 10: Step 9 — Production Scaling & Multi-Agent Roadmap
### From Prototype to Enterprise Manufacturing Plant ERP
1. **Multi-Agent Supervisor Committee**:
   - Decompose into specialized sub-agents: *Sourcing Specialist*, *Compliance Officer*, *Logistics Auditor* coordinated by a Supervisor Agent.
2. **Human-in-the-Loop (HITL) Governance Gates**:
   - Auto-commit routine POs $<\$50,000$.
   - Escalate POs $>\$150,000$ or Sole-Source exceptions to Plant Managers via Slack/Email approval hooks.
3. **Event-Driven ERP Pipeline (Kafka / SAP / NetSuite)**:
   - Real-time event streaming when production schedules shift or inventory drops.

---

## Slide 11: Final Scoreboard & Operational Next Steps
### Delivering Autonomous Procurement for Apex Manufacturing
- **Final Benchmark Verification**:
  - **99.6% Canonical Score** (597.5 / 600.0) | **99.4% Overall Benchmark Score** (1,093.5 / 1,100).
  - **13.9s Parallel Wall Time** | **0.0% Math & ASL Policy Violations**.
- **Open Alignment Questions for Apex Operations**:
  1. Preferred ERP integration protocol (REST vs Kafka event stream)?
  2. Granularity of Human-in-the-Loop escalation thresholds?
  3. On-premise Reflection model infrastructure sizing?
