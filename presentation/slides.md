# Apex Autonomous AI Procurement Agent
## Milestone Prototype Walkthrough & Production Roadmap
**Role**: Forward Deployed Engineer (Reflection AI)  
**Client**: Apex Manufacturing (Head of Operations & Engineering Review)  
**Date**: August 2026 | Milestone Interim Review (45-50 min)

---

## Slide 1: Title & Executive Summary
### Autonomous AI Procurement Agent for Apex Manufacturing
- **Context**: Milestone interim prototype check-in for Apex Manufacturing's procurement automation.
- **Mission**: Automate manual BOM explosion, stock netting, policy compliance, and defensible purchase order creation.
- **Core Results Achieved**:
  - **99.6% accuracy** on official canonical scenarios (01–06).
  - **99.4% overall benchmark score** across 11 complex & boundary stress scenarios.
  - **13.9-second parallel execution** across all 11 scenarios.
  - **Zero mathematical errors & zero policy/ASL violations** via Hybrid Architecture.

> **Speaker Notes (Script)**:
> "Good afternoon! As a Forward Deployed Engineer at Reflection AI, I am thrilled to present our working prototype of the Autonomous Procurement Agent for Apex Manufacturing. Today's session is a milestone check-in to demonstrate working capabilities across all 6 base scenarios and 5 stress tests, discuss system design tradeoffs, and align on our production deployment roadmap."

---

## Slide 2: The Customer Problem & Domain Complexity
### Moving from Manual Bottlenecks to Autonomous Sourcing
- **Apex Manufacturing Pain Points**:
  1. *Manual BOM Explosion & Stock Netting*: Error-prone arithmetic across growing product lines.
  2. *Policy & Memo Hierarchy Conflicts*: Base Policy (`POL-PROC-001`) overlaid with dated Management Memos (`MEMO-2025-041`, `072`, `085`).
  3. *Multi-Objective Tradeoffs*: Balancing domestic preference premiums (up to 50%), supplier tiers, MOQ lot rounding, and tight timeline penalties.
  4. *Audit & Executive Governance*: Need for auditable, defensible rationales and proactive operational risk alerts.

> **Speaker Notes (Script)**:
> "Apex's planners were manually cross-referencing BOMs, warehouse inventories, and complex memos. As order volume scales, manual procurement introduces line-stoppage risks and unapproved vendor traps. Our solution needed to read the state of the world autonomously and act decisively without human prompting."

---

## Slide 3: System Architecture — The Hybrid Deterministic-LLM Pattern
### Separation of Concerns: 100% Precision + Contextual Reasoning
```
[ SQLite State Snapshot ]
         │
         ▼
 ┌──────────────────────────────────────────────────────────┐
 │  1. Deterministic Computational Engine (Pure Python)    │
 │     • DemandAnalyzer: BOM explosion, netting, deadlines  │
 │     • PolicyEngine: ASL check, ISO/UL certs, PCB memos   │
 └─────────────────────────┬────────────────────────────────┘
                           │ Pre-filtered clean candidate pool
                           ▼
 ┌──────────────────────────────────────────────────────────┐
 │  2. Pluggable Cognitive Reasoning Layer (LLM Engine)    │
 │     • Reflection Open Model / Gemini (Factory Pattern)   │
 │     • Multi-objective supplier split & MOQ lot sizing    │
 │     • Defensible PO Rationales & Actionable Risk Alerts  │
 └─────────────────────────┬────────────────────────────────┘
                           │ Pydantic Native Structured Outputs
                           ▼
 [ SQLite purchase_orders & alerts Tables ] ──▶ [ Trajectory Logger ]
```

> **Speaker Notes (Script)**:
> "Why not give the raw DB to an LLM prompt? Because math and hard compliance rules must never hallucinate. We designed a Hybrid Architecture: Pure Python handles MRP netting and ASL gatekeeping with 0% error in sub-milliseconds, while the LLM focuses on what it does best: complex multi-supplier trade-offs, dual-sourcing splits, and audit-ready documentation."

---

## Slide 4: Pluggable AI Stack & Reflection Open Model Compatibility
### Zero Vendor Lock-in: Enterprise Flexibility
- **Architecture Requirement**: The client must not be locked into a single proprietary cloud API.
- **Factory Design Pattern (`src/llm/factory.py`)**:
  - `LLM_PROVIDER=gemini`: Supports `gemini-3.7-flash`, `3.5-flash-lite`, `3.1-pro-preview`.
  - `LLM_PROVIDER=open_model`: Native OpenAI-compatible interface ready for **Reflection-70B**, vLLM, Ollama, or on-premise deployments.
  - **Zero code modifications required** to switch from cloud to local open weights.
- **Pydantic Type Safety (`ProcurementPlanSchema`)**: Native schema enforcement guarantees 0% JSON parsing errors.

> **Speaker Notes (Script)**:
> "Per our assignment requirements, we built a pluggable model interface. When Reflection's open models are deployed on-premise, changing a single environment variable immediately routes all structured generation without touching a single line of business logic."

---

## Slide 5: Comprehensive 11-Scenario Evaluation Suite
### Validating Generalization Across Base & Boundary Conditions
- **Part I: Official Canonical Scenarios (Scenarios 01 – 06)**:
  - `01. Baseline Planning`: 4 production orders, 11 component shortfalls, 50% magnet split.
  - `02. Partial Procurement`: In-flight purchase orders (`EXIST-001..004`) netting.
  - `03. Tight Timeline`: Rush customer order (PO-5005) lead time schedule stress.
  - `04. Low Inventory`: Depleted warehouse stock multi-vendor sourcing.
  - `05. Competing Demand`: Shared component contention across 4 customer orders.
  - `06. Simple Single Order`: Single-product baseline validation.
- **Part II: Synthetic Robustness Sanity Checks (Scenarios 07 – 11)**:
  - `07. Sole-Source Monopoly`, `08. Budget Cap Escalation ($110k+)`, `09. Date Boundary (Expired Memo)`, `10. Extreme MOQ Ratio (500 vs 8)`, `11. Banned Supplier Blacklist Trap`.

> **Speaker Notes (Script)**:
> "To prove our agent generalizes beyond the 6 base scenarios, we built 5 synthetic edge cases covering extreme supply chain crises—such as a 70% cheaper blacklisted supplier and sole-source global monopolies. This ensures the agent is ready for held-out evaluation datasets."

---

## Slide 6: Dual Evaluation Harness (60 Deterministic + 40 LLM-as-a-Judge)
### Rigorous, Multi-Dimensional Scoring Engine
- **Deterministic Evaluation (60 Points Max)**:
  - *Demand Coverage (15 pts)*: 100% net shortfall satisfaction.
  - *MOQ Compliance (10 pts)*: Valid lot sizes $\ge$ supplier MOQ.
  - *Policy & ASL Certs (15 pts)*: Zero unapproved or uncertified vendors.
  - *Management Memos (10 pts)*: Magnet 50% cap & PCB restrictions.
  - *Lead Time Dates (10 pts)*: Expected arrival = order date + lead time.
- **Qualitative LLM Judge (40 Points Max)**:
  - *PO Rationale Quality (20 pts)*: Defensible pricing, lead time, tier citations.
  - *Operational Risk Alerting (20 pts)*: Timely schedule delays, Hazmat protocols.

> **Speaker Notes (Script)**:
> "We implemented an automated dual evaluation harness: 60 points of pure mathematical/policy checks, combined with 40 points of independent LLM audit. The judge model is decoupled via environment variables to eliminate self-evaluation bias."

---

## Slide 7: Multi-Model Benchmark & Pareto Frontier Leaderboard
### Quantitative Experiments Across Model Tiers (66 Total Runs)
| Rank | Model | System Prompt | Total Score | Score % | Think Tokens | Wall Time |
| :---: | :--- | :--- | :---: | :---: | :---: | :---: |
| 🥇 1 | `gemini-3.7-flash` | strict_compliance | **1093.5 / 1100** | **99.4%** | 44,846 | **44.7s** |
| 🥈 2 | `gemini-3.7-flash` | balanced | 1091.0 / 1100 | 99.2% | 44,270 | **29.1s** |
| 🥉 3 | `gemini-3.1-pro-preview` | balanced | 1088.0 / 1100 | 98.9% | 131,715 | 154.1s |
| 4 | `gemini-3.1-pro-preview` | strict_compliance | 1083.5 / 1100 | 98.5% | 120,444 | 164.5s |
| 5 | `gemini-3.5-flash-lite` | balanced | 1073.8 / 1100 | 97.6% | 5,468 | **29.0s** |
| 6 | `gemini-3.5-flash-lite` | strict_compliance | 1065.6 / 1100 | 96.9% | 5,928 | **22.9s** |

- **Key Takeaway**: `Flash` with moderate thinking tokens (~45k) represents the optimal Pareto frontier, achieving 99.4% accuracy at 3x the speed of Pro models.

> **Speaker Notes (Script)**:
> "Here are our experimental findings across 66 independent benchmark runs. Gemini 3.7 Flash achieved the highest score (99.4%) while running in under 45 seconds. Notice that even lightweight models score over 97% because our deterministic core carries the mathematical heavy lifting."

---

## Slide 8: Data-Driven Hill-Climbing (v1.0 ➔ v1.1)
### How Rigorous Error Analysis Drove a +4.7% Benchmark Elevation
- **v1.0 Loss Pattern Analysis ([`REP-EVAL-2026-001`](../reports/v1.0_loss_pattern_analysis.md))**:
  - Identified 5 specific loss patterns (High-MOQ holding risk, multi-order schedule mapping, sole-source standalone alerts).
- **v1.1 Prompt Registry Deployment ([`REP-EVAL-2026-002`](../reports/v1.1_hill_climbing_optimization.md))**:
  - Implemented 7 explicit alert synthesis directives in `src/llm/prompts.py`.
- **Results**:
  - `scenario_10` (Large MOQ): **78.0% $\rightarrow$ 100.0% (+22.0 pts)**
  - `scenario_05` (Competing Demand): **89.5% $\rightarrow$ 100.0% (+10.5 pts)**
  - `scenario_07` (Sole Source): **94.5% $\rightarrow$ 100.0% (+5.5 pts)**
  - **Canonical Score**: **97.3% $\rightarrow$ 99.6%** | **Overall Benchmark**: **94.7% $\rightarrow$ 99.4%**

> **Speaker Notes (Script)**:
> "Rather than guessing, we documented our failure modes in a versioned loss pattern report. In v1.1, we introduced structured prompt directives that transformed our weakest scenario—ordering 500 units for an 8-unit demand—from 78% to a flawless 100% by properly quantifying working capital holding risk."

---

## Slide 9: Observability, Traceability & Live Demo
### Blackbox Execution Tracing & Sub-15s Parallel Benchmark
- **Live Terminal Execution**:
  ```bash
  python3 run_all_and_eval.py   # 11 scenarios parallel in 13.9s
  ```
- **Execution Tracer (`src/trajectory/tracer.py`)**:
  - Step-by-step state logging, raw prompt text, raw model responses.
  - Per-step tracking of **Prompt Tokens, Completion Tokens, Thinking Tokens, and Latency (ms)**.
- **Interactive HTML Dashboard ([`reports/index.html`](../reports/index.html))**:
  - Executive KPI cards, sortable leaderboard, and visual scenario cards.

> **Speaker Notes (Script)**:
> "[Switch to Terminal for Live Demo] Let me demonstrate our parallel execution harness live. In under 14 seconds, all 11 scenario databases are processed and audited. Every decision is saved to trajectories with full token and thinking telemetry."

---

## Slide 10: Production Scaling & Future Roadmap
### What it Takes to Deploy into Real-world Factory ERP
1. **LangGraph Multi-Agent Committee**:
   - Decompose into specialized sub-agents: *Sourcing Specialist*, *Compliance Officer*, *Logistics Auditor*, led by an *Executive Supervisor*.
2. **Human-in-the-Loop (HITL) Governance Gates**:
   - Auto-commit routine POs $<\$50,000$.
   - Route POs $>\$150,000$ and Sole-Source exceptions to plant managers via Slack/Email approval hooks.
3. **Event-Driven ERP Integration (Kafka / SAP / NetSuite)**:
   - Move from batch execution to real-time event streaming when production schedules shift.
4. **3-Model Judge Ensemble**:
   - Multi-evaluator consensus scoring (Gemini + GPT-4o + Claude) for production audit verification.

> **Speaker Notes (Script)**:
> "For our production roadmap with Apex, we propose transitioning from a single-node batch prototype to an event-driven Multi-Agent Committee with Human-in-the-Loop approval gates for capital expenditures exceeding $50k. This gives Apex complete autonomous speed with executive safety."

---

## Slide 11: Conclusion & Q&A Alignment
### Delivering Next-Generation Procurement Autonomy
- **Milestone Summary**:
  - ✅ Working prototype ready for any SQLite scenario snapshot.
  - ✅ 99.6% Canonical / 99.4% Overall accuracy with 0% math/policy errors.
  - ✅ Pluggable architecture ready for Reflection AI open models.
  - ✅ Comprehensive evaluation harness & versioned documentation.
- **Discussion & Open Questions for Apex Operations**:
  1. Preferred ERP integration protocol (REST vs Kafka event stream)?
  2. Granularity of Human-in-the-Loop escalation thresholds?
  3. On-premise Reflection model infrastructure sizing?

> **Speaker Notes (Script)**:
> "Thank you! Our interim prototype is robust, auditable, and production-ready. I look forward to your questions and aligning on the next steps for Apex Manufacturing."
