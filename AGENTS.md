# Portfolio Management - Agent Orientation Guide

This file provides orientation for working on **Portfolio Management**, Sigmance's AI-native adaptive portfolio construction, risk budgeting, rebalancing, and governance system. Use it to understand the project mission, skill architecture, boundaries, and the common working rules that govern implementation.

---

## PROJECT MISSION

**Primary reference (single source of truth / north star)**: `workspace/GOAL.md`

**Goal**: Build an **AI-native adaptive portfolio management system** - the strategic portfolio brain of Sigmance - that understands HOW any mandate should be constructed, risk-budgeted, implemented, and governed, and applies the right methods. Grounded in institutional theory (Grinold & Kahn, *Advances in Active Portfolio Management*).

**Vision**: A skill-based portfolio management engine where an LLM orchestrator navigates a graph of 53+ specialized PM skills, adaptively selecting the right construction methodology, risk framework, signal weighting, implementation policy, and governance structure for each mandate based on its characteristics. Not an optimizer - the encoded portfolio judgment of an entire institutional investment team.

**Core Innovation**: Each portfolio management capability is a discrete, composable skill with its own methodology, guardrails, decision rules, output templates, and worked examples. The intake classifies the mandate, the approach selector routes to the right PM workflow, methodology skills execute with cross-checks, and an auditor validates end-to-end. SymbolicAI contracts ensure deterministic calculations (risk budgets, signal weights, turnover targets) are never hallucinated while LLM reasoning handles qualitative judgment (mandate interpretation, governance framing, crowding assessment).

### Goal Alignment Policy

- The goal is the **north star**: every task should explicitly advance `workspace/GOAL.md`.
- Sidequests are acceptable only if they clearly reduce risk, unlock dependencies, or accelerate delivery toward the goal.
- At the start of each task, include a one-line **Goal alignment** rationale.
- If a requested task does not advance the goal, propose an aligned alternative or record justification before proceeding.

### C2 Role: Portfolio Strategist and Chief Investment Officer

Under the Sigmance Command and Control paradigm, Portfolio Management serves as the **Portfolio Strategist and Chief Investment Officer** of the fleet. When the operator directs "go defensive with 3% tracking error" or "reallocate the risk budget toward macro signals" or "show me where our diversification is illusory," Portfolio Management translates that strategic intent into concrete portfolio construction parameters - position sizes, risk budgets, signal weights, turnover policies, rebalance rules - and reports the portfolio state, implementation efficiency, and governance compliance back through Razor's telemetry loop.

The operator never specifies optimization parameters or rebalance mechanics. They set the strategic objective and constraints. Portfolio Management determines how to construct the portfolio that best expresses that intent within the mandate's governance framework.

### Game-Theoretic Dimension

Portfolio Management must construct portfolios that account for **strategic dynamics**, not just statistical optimization:

- **Crowding risk:** "This factor tilt looks attractive on paper, but every quantitative fund is running the same signal. The crowding itself is a risk factor that should reduce our allocation and widen our cost assumptions."
- **Common factor concentration:** "Our five sleeves look diversified, but they all lean on the same momentum and quality factors. The apparent diversification is illusory - a common factor shock would hit all sleeves simultaneously."
- **Capacity constraints and alpha decay:** "This strategy's alpha is real but capacity-constrained. As assets grow, the expected information ratio compresses. The risk budget must account for capacity-adjusted alpha, not historical alpha."
- **Adversarial fee structures:** "The fee structure makes this fund look cheap on a percentage basis, but after normalizing for the beta component and the expected alpha delivery, the effective fee on the active component is prohibitive."
- **Hidden shared exposures:** "Our multi-manager structure appears diversified across six managers, but three of them are running value strategies with correlated drawdown profiles. The overlap creates tail risk that the variance-covariance matrix does not capture."
- **Strategic timing and edge lifecycle:** "This signal has been decaying for two quarters. If we do not reduce the trade rate and risk allocation, we are paying transaction costs to harvest a dying edge."
- **Reflexivity in portfolio construction:** "If we take a large position, we become the marginal price setter. Our own rebalancing creates the mean-reversion pattern other funds will exploit."

The PortfolioConstructionPack should carry not just position sizes and risk budgets, but **strategic risk assessments** - crowding discounts, capacity-adjusted alpha estimates, overlap risk scores, fee normalization, and edge decay indicators.

---

## PORTFOLIO MANAGEMENT'S ROLE INSIDE SIGMANCE

Portfolio Management sits between pricing/valuation and capital execution in the Sigmance operating loop. It is the **bridge between knowing what assets are worth and deciding what to own** - translating views, valuations, and scenarios into implementable portfolio decisions with explicit risk governance.

### Position in the Operating Loop

```text
Deep Research + Market Matrix -> Multiverse -> Investment Valuation -> PORTFOLIO MANAGEMENT -> UATS
         ^                                                                                      |
         |_________________ Razor + Reasoning Framework (continuous improvement) ________________|
```

Investment Valuation answers: "What is this asset worth?"
Portfolio Management answers: "Given what things are worth, what should the portfolio look like?"
UATS answers: "How do we execute and maintain that portfolio?"

### Upstream Dependencies

- **Investment Valuation**: provides ValuationPackets (fair-value ranges, scenario-conditioned assumptions, peer sets, sensitivity analysis) that anchor the alpha signals and expected-return inputs for portfolio construction.
- **Multiverse**: provides UniverseCards / ScenarioRegistries (probability-weighted futures, triggers, falsifiers, regime assumptions) that condition portfolio construction on different realities and inform scenario-based vs mean-variance selection approaches.
- **Market Matrix**: provides WorldStateSnapshots (current event state, exposure maps, ripple-state deltas, dependency graphs) that ground risk models in live market conditions, inform constraint calibration, and surface crowding or consensus fragility signals.
- **Deep Research**: provides EvidenceBundles (evidence, claims, contradictions, causal structure) that inform mandate interpretation, governance assumptions, and the qualitative judgment layer of portfolio construction.

### Downstream Consumer

- **UATS**: consumes PortfolioConstructionPacks to execute portfolio decisions - position sizes, rebalance rules, risk budgets, turnover policies, signal-weight allocations, trade-rate targets, constraint specifications, and monitoring hooks. UATS does not decide what to own or how much risk to take. It executes the portfolio plan that Portfolio Management produces.

### Boundary Rule

Portfolio Management should stay focused on **portfolio construction, risk budgeting, and governance**:
- It is **not** the pricing engine (that is Investment Valuation).
- It is **not** the scenario generator (that is Multiverse).
- It is **not** the live market data engine (that is Market Matrix).
- It is **not** the evidence synthesizer (that is Deep Research).
- It is **not** the trade executor or strategy runtime (that is UATS).
- It **is** the system that answers: given views on assets and scenarios, what should the portfolio look like, how should risk be budgeted, how should signals be weighted, how should implementation be governed, and how should performance be attributed?

### Simple Stack Model

- **Deep Research** provides the evidence.
- **Market Matrix** provides the live state.
- **Multiverse** provides the futures.
- **Investment Valuation** prices the outcomes.
- **Portfolio Management** constructs the portfolio.
- **UATS** executes and adapts.
- **Razor** builds and improves the engine.

---

## THE SKILL ARCHITECTURE

Portfolio Management is skill-centric. The system's intelligence lives in 53+ discrete, composable skills organized into functional groups, plus 5 shared utility wrappers for overlap with Investment Valuation.

### Intake and Routing

| Skill | Purpose |
| ----- | ------- |
| `portfolio-management-intake-skill` | Classify the mandate, benchmark, universe, horizon, turnover tolerance, leverage policy, and decision problem |
| `pm-approach-selector-skill` | Route to alpha research, dynamic implementation, attribution, smart beta, or portfolio construction workflow |
| `pm-assumption-checker-skill` | Audit load-bearing assumptions: skill, breadth, decay, cost, constraints, benchmark, capacity |
| `pm-bias-check-skill` | Detect overconfidence, data mining, stale beliefs, manager-selection bias, hidden crowding |
| `pm-model-auditor-skill` | Test internal coherence across forecasts, risk model, turnover, costs, and implementation logic |
| `benchmark-and-mandate-reader-skill` | Extract true benchmark, active universe, responsibility split, and policy constraints from a mandate |
| `risk-budget-router-skill` | Translate mandate-level objectives into tracking-error, active-risk, leverage, capacity, and signal-budget targets |

### Alpha Signal and Fundamental Law

| Skill | Purpose |
| ----- | ------- |
| `active-management-insights-skill` | Apply the seven core active-management insights as a structured diagnostic |
| `fundamental-law-diagnostics-skill` | Decompose expected information ratio into skill, breadth, and efficiency |
| `breadth-skill-time-skill` | Estimate breadth from information turnover and universe size; explain skill-time dynamics |
| `alpha-signal-calibration-skill` | Convert raw signals into standardized alphas controlling for expectations, skill, and volatility |
| `data-mining-sanity-check-skill` | Red-team backtests and signal research for false discovery and overfit |
| `transfer-coefficient-diagnostics-skill` | Measure how constraints and costs degrade paper portfolio to real portfolio |
| `constraint-and-cost-impact-skill` | Quantify how long-only constraints and costs erode expected information ratio |
| `market-efficiency-beliefs-check-skill` | Map belief set required to justify passive, active, or factor/smart-beta exposure choices |
| `benchmark-efficiency-test-skill` | Test whether the benchmark is efficient relative to the opportunity set |

### Risk Framework

| Skill | Purpose |
| ----- | ------- |
| `risk-framework-skill` | Shared risk taxonomy - priced vs unpriced risk framing with PM-specific logic |
| `downside-risk-lab-skill` | Compare variance-based with downside and scenario-based risk measures |
| `optimal-gearing-skill` | Analyze leverage/gearing choices and target risk interdependence |
| `tail-risk-concentration-check-skill` | Identify where leverage, common factors, or hidden overlap concentrate tail risk |
| `diversification-danger-check-skill` | Detect overdiversification, diluted alpha, and hidden common-factor concentration |
| `common-factor-crowding-check-skill` | Check whether diversified sleeves are all leaning on the same factors |
| `multi-manager-overlap-risk-skill` | Measure overlap and unintended shared exposures across managers or products |
| `capacity-alpha-growth-skill` | Model how asset growth and shared capacity compress expected alpha |

### Dynamic Portfolio and Implementation

| Skill | Purpose |
| ----- | ------- |
| `dynamic-portfolio-analysis-skill` | Analyze the portfolio as a moving object: signal speed, trade speed, age profile, backlog |
| `signal-weighting-skill` | Allocate risk across signals with different strength, decay, and cost impact |
| `signal-decay-estimator-skill` | Estimate information turnover, half-life, and aging behavior per signal family |
| `trade-rate-optimizer-skill` | Choose cost-aware trading pace balancing staleness against transaction costs |
| `backlog-and-age-profile-analyzer-skill` | Measure how far the live portfolio lags the target and how old embedded information is |
| `linear-trading-rules-skill` | Build and evaluate linear rebalance rules |
| `nonlinear-trading-rules-skill` | Build and evaluate nonlinear or no-trade-region rules for friction-heavy environments |
| `turnover-policy-designer-skill` | Design turnover policy consistent with signal half-life, cost model, and efficiency target |
| `transaction-cost-opportunity-loss-skill` | Attribute implementation drag between explicit cost and opportunity loss |
| `implementation-efficiency-audit-skill` | Break implementation shortfall into opportunity loss, direct cost, backlog, and transfer effects |
| `scenario-vs-mean-variance-skill` | Compare mean-variance and scenario-based portfolio selection approaches |
| `scenario-process-controller-skill` | Set governance rules for scenario-based construction to prevent arbitrary scenario choice |

### Attribution and Diagnostics

| Skill | Purpose |
| ----- | ------- |
| `attribution-engine-skill` | Perform ex ante or ex post attribution using covariance/correlation framework |
| `portfolio-description-skill` | Describe portfolio through exposures, opportunity-set relations, and mimicking portfolios |
| `exposure-mimicking-portfolios-skill` | Turn portfolio attributes into analyzable portfolios for standard return/risk analysis |
| `opportunity-loss-attribution-skill` | Explain where opportunity loss came from by source, signal family, or implementation stage |
| `ex-ante-ex-post-attribution-check-skill` | Compare model predictions with realized attribution |
| `holdings-flow-diagnostics-skill` | Read the live portfolio in stock and flow terms to explain exposure changes |

### Smart Beta and Product Design

| Skill | Purpose |
| ----- | ------- |
| `active-passive-smart-beta-router-skill` | Recommend passive beta, active alpha, smart beta, or blend for the problem |
| `smart-beta-buyer-fit-skill` | Decide which investors should rationally buy smart beta vs passive vs active |
| `smart-beta-owners-manual-skill` | Due diligence on smart-beta products: factor exposures, implementation, hidden responsibilities |
| `smart-beta-illustrator-skill` | Explain smart beta behavior with worked examples and product comparisons |
| `factor-product-blend-designer-skill` | Design coherent factor product blends rather than overlapping tilts |
| `investor-manager-responsibility-map-skill` | Map responsibilities between investor and manager across product types |
| `asset-manager-dilemma-skill` | Analyze tension between smart-beta offerings and active-management economics |

### Governance and Output

| Skill | Purpose |
| ----- | ------- |
| `fee-myth-buster-skill` | Normalize fee structures for economically meaningful comparison |
| `portfolio-review-memo-writer-skill` | Produce standardized portfolio review memo with benchmark, alpha, risk, implementation, actions |
| `investment-committee-pack-skill` | Assemble committee-ready pack linking return, risk, implementation, attribution, governance |
| `pm-red-team-skill` | Attack PM thesis from breadth, skill inflation, crowding, cost, benchmark weakness, false diversification |
| `portfolio-update-loop-skill` | Re-run diagnostic stack when exposures, signals, costs, or benchmark conditions change |

### Shared Utility Wrappers (Overlap with Investment Valuation)

| Skill | Purpose |
| ----- | ------- |
| `risk-framework-skill` | Shared risk taxonomy helper (reuse for priced-vs-unpriced risk framing) |
| `beta-estimator-skill` | Shared beta helper for security/portfolio beta estimation |
| `equity-risk-premium-estimator-skill` | Shared capital-market-assumption helper for equity premium inputs |
| `riskfree-rate-selector-skill` | Shared rate-selection helper for consistent capital-market assumptions |
| `scenario-analysis-skill` | Shared scenario engine for portfolio-construction and downside-risk workflows |

---

## CORE OPERATING FLOW

The adaptive portfolio management workflow:

```text
Mandate + Views + Scenarios
    |
    v
portfolio-management-intake-skill   -- Classify: mandate type, benchmark, universe, horizon,
    |                                   turnover tolerance, leverage policy, decision problem
    v
pm-approach-selector-skill          -- Route: alpha research, dynamic implementation,
    |                                   attribution, smart beta, or portfolio construction
    v
[Signal Calibration]
    |-- alpha-signal-calibration-skill -> fundamental-law-diagnostics-skill
    |-- breadth-skill-time-skill -> signal-decay-estimator-skill
    |-- data-mining-sanity-check-skill (red-team signals)
    |
    v
[Risk Budgeting]
    |-- risk-budget-router-skill -> risk-framework-skill
    |-- benchmark-efficiency-test-skill -> constraint-and-cost-impact-skill
    |-- downside-risk-lab-skill -> tail-risk-concentration-check-skill
    |
    v
[Portfolio Construction]
    |-- signal-weighting-skill -> dynamic-portfolio-analysis-skill
    |-- scenario-vs-mean-variance-skill -> scenario-process-controller-skill
    |-- optimal-gearing-skill (if leverage allowed)
    |
    v
[Implementation Optimization]
    |-- trade-rate-optimizer-skill -> turnover-policy-designer-skill
    |-- linear/nonlinear-trading-rules-skill
    |-- implementation-efficiency-audit-skill -> transfer-coefficient-diagnostics-skill
    |
    v
[Attribution and Monitoring]
    |-- attribution-engine-skill -> ex-ante-ex-post-attribution-check-skill
    |-- portfolio-description-skill -> holdings-flow-diagnostics-skill
    |-- opportunity-loss-attribution-skill
    |
    v
[Governance and Red Team]
    |-- pm-assumption-checker-skill -> pm-bias-check-skill -> pm-model-auditor-skill
    |-- pm-red-team-skill
    |-- diversification-danger-check-skill -> common-factor-crowding-check-skill
    |
    v
Structured PortfolioConstructionPack  -- Position sizes, risk budgets, signal weights,
                                         turnover policies, rebalance rules, governance memo
```

### Design Principles

1. **Classify before constructing** - Always understand the mandate before choosing a construction methodology.
2. **Never force one approach** - Mean-variance, scenario-based, factor-tilt, risk-parity - the methodology must fit the mandate.
3. **Always red-team** - No portfolio construction is complete without adversarial challenge on breadth, crowding, cost, and false diversification.
4. **Make risk budgets explicit** - Every output carries transparent risk allocation across signals, factors, and implementation.
5. **Handle special mandates natively** - Multi-manager, leveraged, smart-beta, long-only constrained, and scenario-driven mandates get purpose-built workflows.
6. **Separate alpha from beta** - Always decompose expected return into systematic (beta/factor) and idiosyncratic (alpha) components.
7. **Account for implementation** - Paper portfolios are not real portfolios. Every construction accounts for costs, constraints, and transfer coefficient degradation.
8. **Govern the process** - Scenario selection, signal weighting, and rebalance decisions must be governed, not arbitrary.

---

## PROJECT STRUCTURE OVERVIEW

### Core Directories

#### `./portfolio-management/` - Portfolio Management Skill Library
**Purpose**: The 53+ specialized PM skills that form the system's knowledge base.
Each skill directory contains: `SKILL.md` (workflow), `reference.md` (formulas/rules), `template.md` (output format), `examples.md` (worked cases), `scripts/execute.py`.
Shared utilities: `_shared/portfolio_management_helpers.py`.

#### `./workspace/` - Knowledge and Planning Hub
**Purpose**: Planning, references, research, tracking files, and architecture documentation.
Key files: `GOAL.md`, `PROJECT_REFERENCE.md`.

Subdirectories:
- `agent-research/` - Exploratory research and ideation drafts
- `agent-review/` - Pre-implementation reviews
- `brainstorming/` - Free-form ideation sessions
- `concepts/` - Concept extractions and compositions
- `deep-research/` - Deep research outputs
- `evolution/` - Evolutionary improvement artifacts
- `integration/` - Finalized integration designs
- `notes/` - Operator-directed findings and observations
- `references/` - Curated documentation and papers
- `reports/` - Generated reports and analyses
- `research-projects/` - Structured research projects
- `resources-for-comparison/` - Comparison resources
- `system-design/` - System-level design specs

#### Adjacent/upstream workspaces to consult
- `../investment-analysis-workspace/` (ValuationPackets - pricing and fair-value inputs)
- `../multiverse-workspace/` (UniverseCards - scenario-conditioned inputs)
- `../market-matrix-workspace/` (WorldStateSnapshots - live market state)
- `../deep-research-workspace/` (EvidenceBundles - evidence and causal structure)
- `../trading-system-workspace/` (UATS - downstream execution consumer)
- `../copilot-workspace/` (Razor - orchestration and improvement)

---

## INTEGRATION RULES

### Investment Valuation Integration

Use Investment Valuation for:
- fair-value ranges and expected-return inputs that feed alpha signal calibration,
- scenario-conditioned valuation assumptions that inform portfolio tilts,
- peer-set and sensitivity analysis that ground relative positioning decisions.

Do **not** duplicate valuation methodology inside Portfolio Management.

### Multiverse Integration

Use Multiverse for:
- scenario families and probability-weighted futures that condition portfolio construction,
- regime assumptions that drive mean-variance vs scenario-based approach selection,
- triggers and falsifiers that define rebalance and monitoring hooks.

Do **not** build scenario generation logic inside Portfolio Management.

### Market Matrix Integration

Use Market Matrix for:
- live market state and exposure context that ground risk model calibration,
- dependency graphs and crowding signals that inform constraint and risk-budget decisions,
- event-driven context for dynamic portfolio adjustment triggers.

Do **not** reimplement market data ingestion inside Portfolio Management.

### Deep Research Integration

Use Deep Research for:
- evidence backing governance assumptions and mandate interpretation,
- causal structure informing signal quality assessment and red-teaming,
- contradiction surfaces that challenge portfolio construction premises.

Do **not** duplicate Deep Research's evidence pipeline inside Portfolio Management.

### UATS Integration

UATS is downstream. Portfolio Management should publish structured PortfolioConstructionPacks that UATS can consume directly for execution - position sizes, rebalance rules, risk budgets, turnover policies, signal-weight allocations, trade-rate targets, and monitoring hooks.

Portfolio Management should not become:
- a trade execution system,
- a strategy runtime,
- a live monitoring stack for positions (beyond attribution and diagnostics).

---

## RESOURCE LOCATIONS

### Master Planning and Goals

- **Goal (authoritative)**: `workspace/GOAL.md`
- **Architecture reference**: `workspace/PROJECT_REFERENCE.md`

### Research and Knowledge

- **Agent research**: `workspace/agent-research/`
- **Agent reviews**: `workspace/agent-review/`
- **Brainstorming**: `workspace/brainstorming/`
- **References**: `workspace/references/`
- **System design**: `workspace/system-design/`
- **Integration docs**: `workspace/integration/`
- **Concepts**: `workspace/concepts/`
- **Reports**: `workspace/reports/`

### Skill Library

- **All skills**: `portfolio-management/` (53+ directories)
- **Shared helpers**: `portfolio-management/_shared/portfolio_management_helpers.py`
- **Skill format**: Each skill has `SKILL.md`, `reference.md`, `template.md`, `examples.md`, `scripts/execute.py`

---

## DEVELOPMENT PRIORITIES

1. **Implement the adaptive portfolio management engine**
   - Wire the intake -> approach-selector -> signal-calibration -> risk-budgeting -> construction -> implementation -> attribution flow into executable code
   - Build the LLM orchestrator that routes through the skill graph

2. **Build the PortfolioConstructionPack output format**
   - Structured output that UATS can consume directly
   - Position sizes, risk budgets, signal weights, turnover policies, rebalance rules, governance memo
   - Machine-readable with human-readable governance documentation

3. **Connect to upstream subsystems**
   - Accept ValuationPackets from Investment Valuation for expected-return inputs
   - Accept UniverseCards from Multiverse for scenario-conditioned construction
   - Accept WorldStateSnapshots from Market Matrix for live risk calibration
   - Accept EvidenceBundles from Deep Research for governance grounding

4. **Implement the risk-budgeting pipeline**
   - Risk-budget router translating mandate objectives into quantitative targets
   - Tracking-error, active-risk, leverage, capacity, and signal-budget allocation
   - Tail-risk and crowding overlays from game-theoretic analysis

5. **Build the dynamic portfolio and implementation layer**
   - Signal weighting with decay-aware allocation
   - Trade-rate optimization with cost-aware rebalancing
   - Linear and nonlinear trading rules
   - Implementation efficiency attribution

6. **Build the attribution and governance layer**
   - Ex ante and ex post attribution framework
   - Red-team and bias-check pipeline
   - Portfolio review memo and investment committee pack generation

Always apply the Goal Alignment Policy above when choosing, scoping, or sequencing work.

---

## TRACKING, PLANNING, AND KNOWLEDGE FILES

### Required Master Files

Keep these current:
- `workspace/GOAL.md`
- `workspace/PROJECT_REFERENCE.md`

### Tracking Files

For complex tasks, create under `workspace/tracking-files/`:
- `<TaskSlug>_Knowledge.md` - Context, Constraints, Evidence/Citations, Decisions, Rationale, Open Questions
- `<TaskSlug>_TRACKING.md` - Checklist, Timeline/Log, Blockers, Mitigations, Next Steps

### Progress Logging Rule

Progress belongs in task-specific tracking files. Do not put progress logs into `AGENTS.md`.

---

## COMMON WORKFLOW RULES (TAILORED TO PORTFOLIO MANAGEMENT)

These are the common rules used across Sigmance workspaces, tailored to this project.

### General Directives

- Keep outputs and messages concise and information-dense.
- Only modify or create files/directories that the task actually requires.
- Treat Portfolio Management as a skill-centric system; prefer routing through existing skills over building ad-hoc construction logic.
- Always preserve methodology provenance. If a portfolio construction assumption comes from another subsystem (Investment Valuation expected return, Multiverse scenario, Market Matrix crowding signal), trace it.
- Do not silently smuggle valuation/execution logic into this workspace.

### Task Planning and Tracking

- For multi-step or ambiguous work, start with a concise checklist (3-7 conceptual steps, short wording).
- Maintain a plan with one `in_progress` item at a time.
- Update the plan after each completed step and explain changes if the plan shifts.
- If no plan exists for complex work, create one before changing files.

### Tool Call Preambles

- Before each tool call, briefly state purpose and minimal inputs.
- After each tool call or edit, validate the result in 1-2 lines and correct course if needed.

### Ideation Phase Management

- For ideation-only work, write drafts to `workspace/agent-research/`.
- Prefix those files with a timestamp header and a short title.
- If the user asks for `!research`, do not change implementation files.

### Review Before Implementation

- For non-trivial implementation tasks, create a review in `workspace/agent-review/` when asked via `!review` or when the workflow requires it.
- Reviews should focus on: task understanding, relevant skills and modules, risks, and likely failure modes.

### Implementation Assistance Guidelines

- Prioritize files explicitly provided by the user.
- Review related skills and their reference.md before implementing PM logic.
- Keep changes focused and avoid scope creep.
- Do not run tests unless explicitly requested.
- When adding new calculation logic, check if `portfolio-management/_shared/portfolio_management_helpers.py` already provides the function.

---

## ENGINEERING HYGIENE

### Code Style Standards

- Prefer clear, direct, intent-revealing code.
- Use modern type hints.
- Prefer declarative code where practical.
- Keep functions and modules narrowly scoped.
- Add comments only when the context is genuinely non-obvious.
- Always use f-strings, not %s or %d.

### Imports

- Keep imports at the top.
- Group standard library, third-party, and local imports with blank lines between groups.
- Do not place imports inside functions unless absolutely necessary and justified.

### Refactoring Rules

- Remove obsolete code immediately after refactors.
- Update all callers when interfaces change.
- Eliminate dead wrappers and duplicates.
- Verify schema/interface changes across the repo with search.

### Layout

- Two blank lines between top-level functions/classes.
- One blank line between methods inside classes.
- Keep module structure predictable and small.

---

## TESTING RULES

- Only run tests if explicitly requested.
- If asked to design tests, prefer:
  - portfolio construction accuracy tests,
  - skill routing and selection tests,
  - risk-budget calculation validation tests,
  - signal-weighting and decay estimation tests,
  - attribution reconciliation tests,
  - SymbolicAI contract compliance tests.
- Do not mention or inspect `tests/` unless relevant to the user's request.

---

## PORTFOLIO-MANAGEMENT-SPECIFIC DESIGN RULES

### 1. Classify before constructing

Do not run portfolio construction models without first classifying the mandate through the intake skill. The methodology must fit the mandate.

### 2. Never force one approach

If the approach selector recommends mean-variance, explain why scenario-based or risk-parity was not chosen. If it recommends factor tilts, explain why pure alpha was weaker for this mandate.

### 3. Always red-team

No portfolio construction output is complete without adversarial challenge from breadth, crowding, cost underestimation, benchmark weakness, and false diversification angles.

### 4. Make risk budgets explicit

Every portfolio output must explicitly state: how risk is allocated across signals and factors, what the tracking-error and active-risk targets are, what constraints bind, and what the expected transfer coefficient is.

### 5. Handle special mandates natively

Multi-manager, leveraged, smart-beta, long-only constrained, and scenario-driven mandates get purpose-built workflows. Do not force them through a generic mean-variance optimizer.

### 6. Separate alpha from beta

Always decompose expected return into systematic and idiosyncratic components. Know which part of the portfolio is paying for beta and which for alpha.

### 7. Account for implementation

Paper portfolios are not real portfolios. Every construction must account for costs, constraints, signal decay, and transfer coefficient degradation.

### 8. Govern scenario-based construction

When using scenario-based portfolio selection, the scenario process controller must enforce governance rules so scenario choice does not become arbitrary.

### 9. Keep outputs machine-readable

Produce structured PortfolioConstructionPacks that UATS and other subsystems can consume. The human-readable governance memo is important, but the structured output is what closes the loop.

---

## NON-GOALS / SCOPE CONTROLS

For the Portfolio Management workspace:
- Do **not** build a valuation or pricing engine here (that is Investment Valuation).
- Do **not** build scenario generation here (that is Multiverse).
- Do **not** build market data ingestion here (that is Market Matrix).
- Do **not** duplicate Deep Research's evidence pipeline.
- Do **not** build a trade execution or strategy runtime here (that is UATS).

Portfolio Management is a **portfolio construction, risk budgeting, and governance engine**, not a pricing system or trade executor.

---

## SAFETY AND SCOPE CONTROLS

- Never commit, expose, or log secrets.
- Only modify files and directories that fall within the active task plan.
- Preserve provenance and structured outputs.
- Be explicit when something is inferred, uncertain, or unsupported by available data.
- Portfolio construction calculations must be deterministic - use SymbolicAI syntactic mode for arithmetic (risk budgets, signal weights, turnover targets), semantic mode for qualitative judgment only (mandate interpretation, governance framing).

---

## CANONICAL SHORTHAND

Use this shorthand when thinking about the project:

**Classify the mandate -> Select the approach -> Calibrate signals -> Budget risk -> Construct portfolio -> Optimize implementation -> Attribute performance -> Govern and red-team -> Produce PortfolioConstructionPack**

And at the system level:

**Deep Research provides evidence. Market Matrix provides live state. Multiverse provides futures. Investment Valuation prices the outcomes. Portfolio Management constructs the portfolio. UATS executes and adapts. Razor builds and improves the engine.**
