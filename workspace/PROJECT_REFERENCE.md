# PROJECT REFERENCE - Portfolio Management

This document is the living architecture reference for the Portfolio Management subsystem. It summarizes the skill architecture, handoff objects, integration contracts, orchestration flow, and design decisions.

---

## Skill Architecture Overview

Portfolio Management contains **53 portfolio-management-specific skills** plus **5 shared utility wrappers** for overlap with Investment Valuation. Skills are organized into eight functional groups.

### Skill Groups

| Group | Count | Function |
| ----- | ----- | -------- |
| **Intake and Routing** | 7 | Mandate classification, approach selection, assumption/bias/coherence auditing, benchmark reading, risk-budget routing |
| **Alpha Signal and Fundamental Law** | 9 | Active management insights, Fundamental Law diagnostics, breadth/skill estimation, signal calibration, data-mining checks, transfer coefficient, constraint impact, efficiency beliefs, benchmark efficiency |
| **Risk Framework** | 8 | Risk taxonomy, downside risk, gearing, tail risk, diversification danger, factor crowding, multi-manager overlap, capacity/alpha/growth modeling |
| **Dynamic Portfolio and Implementation** | 12 | Dynamic analysis, signal weighting, signal decay, trade-rate optimization, backlog analysis, linear/nonlinear trading rules, turnover policy, transaction cost attribution, implementation audit, scenario-vs-MV comparison, scenario governance |
| **Attribution and Diagnostics** | 6 | Ex ante/ex post attribution, portfolio description, exposure mimicking, opportunity-loss attribution, attribution reconciliation, holdings-flow diagnostics |
| **Smart Beta and Product Design** | 7 | Active/passive/smart-beta routing, buyer fit, product due diligence, illustration, factor blend design, responsibility mapping, asset-manager dilemma analysis |
| **Governance and Output** | 5 | Fee normalization, portfolio review memo, investment committee pack, PM red-team, portfolio update loop |
| **Shared Utility Wrappers** | 5 | Risk framework, beta estimation, equity risk premium, risk-free rate, scenario analysis (shared with Investment Valuation) |

### Skill Structure

Each skill directory follows a consistent structure:

```
<skill-name>/
  SKILL.md          -- YAML frontmatter + workflow instructions
  reference.md      -- Formulas, rules, methodology grounding
  template.md       -- Output format and structure
  examples.md       -- Worked cases and illustrations
  scripts/
    execute.py      -- Executable skill implementation
```

Shared utilities live in `portfolio-management/_shared/portfolio_management_helpers.py`.

### Skill Crosswalk with Investment Valuation

Five skills are shared utility wrappers that overlap with the Investment Valuation skill library:

| Skill | Shared Purpose | PM-Specific Extension |
| ----- | -------------- | --------------------- |
| `risk-framework-skill` | Risk taxonomy (priced vs unpriced) | Wrapped with portfolio-level risk budgeting logic |
| `beta-estimator-skill` | Security/portfolio beta estimation | Used for benchmark exposure diagnostics |
| `equity-risk-premium-estimator-skill` | Capital-market equity premium | Used for expected-return and benchmark-efficiency inputs |
| `riskfree-rate-selector-skill` | Risk-free rate selection | Used for internally consistent capital-market assumptions |
| `scenario-analysis-skill` | Scenario engine | Used underneath portfolio-construction and downside-risk workflows |

If the two repositories are merged, these wrappers can be removed in favor of the existing Investment Valuation implementations.

---

## Handoff Objects

### Produced: PortfolioConstructionPack

The primary output object consumed by UATS.

| Field | Type | Description |
| ----- | ---- | ----------- |
| `mandate_id` | string | Unique identifier for the mandate/strategy |
| `mandate_classification` | object | Mandate type, benchmark, universe, horizon, constraints, leverage policy |
| `approach` | string | Selected construction methodology (mean-variance, scenario-based, factor-tilt, risk-parity, blended) |
| `position_sizes` | array | Target position weights with rationale |
| `risk_budget` | object | Tracking-error target, active-risk allocation by signal/factor, leverage budget, capacity constraint |
| `signal_weights` | array | Allocation across signal families with strength, decay, cost impact, and breadth metadata |
| `turnover_policy` | object | Target turnover rate, cost model, signal half-life alignment, trade-rate parameters |
| `rebalance_rules` | object | Linear/nonlinear trading rules, no-trade regions, trigger conditions |
| `constraint_spec` | object | Long-only, sector limits, position limits, benchmark-relative constraints |
| `implementation_efficiency` | object | Expected transfer coefficient, opportunity-loss estimate, cost breakdown |
| `scenario_conditioning` | object | If scenario-based: active scenarios, probabilities, governance rules |
| `attribution_baseline` | object | Ex ante expected attribution (alpha, factor, timing, implementation) |
| `monitoring_hooks` | array | Signal decay triggers, rebalance triggers, constraint-breach alerts, attribution drift thresholds |
| `governance_memo` | string | Human-readable governance documentation for committee review |
| `red_team_summary` | object | Crowding assessment, false diversification flags, skill inflation checks, cost underestimation risks |
| `provenance` | object | Input sources (ValuationPacket IDs, UniverseCard IDs, WorldStateSnapshot ID, EvidenceBundle IDs) |
| `timestamp` | datetime | Construction timestamp |

### Consumed

| Object | Source | Used For |
| ------ | ------ | -------- |
| **ValuationPacket** | Investment Valuation | Fair-value ranges as expected-return inputs; scenario-conditioned assumptions for signal calibration; peer-set context for relative positioning |
| **UniverseCard / ScenarioRegistry** | Multiverse | Probability-weighted futures for scenario-based construction; regime assumptions for approach selection; triggers/falsifiers for monitoring hooks |
| **WorldStateSnapshot** | Market Matrix | Live dependency graphs for risk model calibration; crowding signals for constraint and risk-budget decisions; exposure maps for overlap analysis |
| **EvidenceBundle** | Deep Research | Evidence for governance assumptions; causal structure for signal quality assessment; contradictions for red-teaming |

---

## Integration Architecture

### Integration with Investment Valuation

Investment Valuation produces ValuationPackets. Portfolio Management consumes them as **expected-return inputs**:

```text
ValuationPacket.fair_value_range -> alpha-signal-calibration-skill (expected return input)
ValuationPacket.scenario_assumptions -> scenario-vs-mean-variance-skill (approach conditioning)
ValuationPacket.peer_sets -> benchmark-efficiency-test-skill (relative positioning context)
ValuationPacket.strategic_risk_factors -> pm-red-team-skill (crowding/reflexivity inputs)
```

Portfolio Management does not reprice assets. It consumes prices and constructs portfolios.

### Integration with Multiverse

Multiverse produces UniverseCards and ScenarioRegistries. Portfolio Management consumes them for **scenario-conditioned construction**:

```text
UniverseCard.probability -> scenario-process-controller-skill (probability-weighted construction)
UniverseCard.triggers -> monitoring_hooks (rebalance triggers)
UniverseCard.falsifiers -> monitoring_hooks (thesis invalidation alerts)
ScenarioRegistry.regime_assumptions -> pm-approach-selector-skill (mean-variance vs scenario routing)
```

Portfolio Management does not generate scenarios. It constructs portfolios conditioned on scenarios.

### Integration with Market Matrix

Market Matrix produces WorldStateSnapshots. Portfolio Management consumes them for **live risk calibration**:

```text
WorldStateSnapshot.exposure_maps -> risk-budget-router-skill (current exposure context)
WorldStateSnapshot.crowding_signals -> common-factor-crowding-check-skill (crowding risk input)
WorldStateSnapshot.dependency_graphs -> multi-manager-overlap-risk-skill (hidden correlation detection)
WorldStateSnapshot.event_state -> portfolio-update-loop-skill (dynamic adjustment triggers)
```

Portfolio Management does not ingest market data. It consumes structured state for risk calibration.

### Integration with UATS (Downstream)

Portfolio Management produces PortfolioConstructionPacks. UATS consumes them for **execution**:

```text
PortfolioConstructionPack.position_sizes -> UATS position targeting
PortfolioConstructionPack.rebalance_rules -> UATS trade scheduling
PortfolioConstructionPack.risk_budget -> UATS risk monitoring
PortfolioConstructionPack.turnover_policy -> UATS trade-rate control
PortfolioConstructionPack.monitoring_hooks -> UATS alert and trigger system
PortfolioConstructionPack.constraint_spec -> UATS constraint enforcement
```

UATS does not decide what to own or how much risk to take. It executes the portfolio plan.

---

## Orchestration Flow

The full orchestration flow from mandate to PortfolioConstructionPack:

### Phase 1: Mandate Intake

```text
Input: Mandate document, benchmark, constraints, operator intent
    |
    v
portfolio-management-intake-skill
    -> Classify mandate type, benchmark, universe, horizon,
       turnover tolerance, leverage policy, decision problem
    |
    v
benchmark-and-mandate-reader-skill
    -> Extract true benchmark, active universe, responsibility split,
       policy constraints
    |
    v
risk-budget-router-skill
    -> Translate objectives into tracking-error, active-risk,
       leverage, capacity, signal-budget targets
```

### Phase 2: Approach Selection

```text
pm-approach-selector-skill
    -> Route to: alpha research | dynamic implementation |
       attribution | smart beta | portfolio construction
    |
    v
market-efficiency-beliefs-check-skill (if applicable)
    -> Map required belief set for chosen approach
    |
    v
active-passive-smart-beta-router-skill (if applicable)
    -> Recommend passive, active, smart-beta, or blend
```

### Phase 3: Signal Calibration

```text
Consume ValuationPackets as expected-return inputs
    |
    v
alpha-signal-calibration-skill
    -> Convert raw signals to standardized alphas
    |
    v
fundamental-law-diagnostics-skill
    -> Decompose expected IR into skill, breadth, efficiency
    |
    v
breadth-skill-time-skill
    -> Estimate breadth and skill-time dynamics
    |
    v
signal-decay-estimator-skill
    -> Estimate half-life and aging per signal family
    |
    v
data-mining-sanity-check-skill
    -> Red-team signals for false discovery
```

### Phase 4: Risk Budgeting

```text
Consume WorldStateSnapshots for live risk calibration
    |
    v
risk-framework-skill
    -> Decompose risk: priced vs unpriced, systematic vs idiosyncratic
    |
    v
benchmark-efficiency-test-skill
    -> Test benchmark efficiency relative to opportunity set
    |
    v
constraint-and-cost-impact-skill
    -> Quantify friction impact on expected IR
    |
    v
downside-risk-lab-skill
    -> Compare variance-based vs downside/scenario-based risk
    |
    v
tail-risk-concentration-check-skill
    -> Identify hidden tail-risk concentration
    |
    v
diversification-danger-check-skill + common-factor-crowding-check-skill
    -> Detect false diversification and factor crowding
```

### Phase 5: Portfolio Construction

```text
Consume UniverseCards for scenario conditioning
    |
    v
scenario-vs-mean-variance-skill
    -> Select construction approach (MV vs scenario-based)
    |
    v
signal-weighting-skill
    -> Allocate risk across signal families
    |
    v
dynamic-portfolio-analysis-skill
    -> Model portfolio as moving object with signal/trade speed
    |
    v
optimal-gearing-skill (if leverage allowed)
    -> Analyze leverage choices and target risk interaction
    |
    v
scenario-process-controller-skill (if scenario-based)
    -> Enforce governance on scenario selection
```

### Phase 6: Implementation Optimization

```text
trade-rate-optimizer-skill
    -> Choose cost-aware trading pace
    |
    v
linear-trading-rules-skill / nonlinear-trading-rules-skill
    -> Build rebalance rules appropriate to friction environment
    |
    v
turnover-policy-designer-skill
    -> Design turnover policy aligned with decay and cost
    |
    v
implementation-efficiency-audit-skill
    -> Break shortfall into cost, opportunity loss, backlog, transfer
    |
    v
transfer-coefficient-diagnostics-skill
    -> Measure paper-to-real portfolio degradation
    |
    v
transaction-cost-opportunity-loss-skill
    -> Attribute implementation drag
```

### Phase 7: Attribution and Diagnostics

```text
attribution-engine-skill
    -> Perform ex ante / ex post attribution
    |
    v
portfolio-description-skill
    -> Describe portfolio through exposures and mimicking portfolios
    |
    v
ex-ante-ex-post-attribution-check-skill
    -> Compare predicted vs realized attribution
    |
    v
holdings-flow-diagnostics-skill
    -> Explain exposure changes in stock and flow terms
    |
    v
opportunity-loss-attribution-skill
    -> Explain opportunity loss by source and stage
```

### Phase 8: Governance and Red Team

```text
pm-assumption-checker-skill
    -> Audit load-bearing assumptions
    |
    v
pm-bias-check-skill
    -> Detect overconfidence, data mining, stale beliefs
    |
    v
pm-model-auditor-skill
    -> Test internal coherence across all components
    |
    v
pm-red-team-skill
    -> Attack from breadth, crowding, cost, benchmark, diversification angles
    |
    v
portfolio-review-memo-writer-skill
    -> Produce standardized review memo
    |
    v
investment-committee-pack-skill
    -> Assemble committee-ready governance pack
    |
    v
OUTPUT: PortfolioConstructionPack
```

---

## Key Architectural Decisions

### 1. Skill-centric, not optimizer-centric

The system's intelligence lives in 53+ discrete skills, not in a single optimization engine. The orchestrator navigates the skill graph adaptively. This enables mandate-specific methodology while keeping each skill independently testable and improvable.

### 2. Separate from Investment Valuation

Portfolio Management is a distinct subsystem, not a module within Investment Valuation. Investment Valuation prices assets. Portfolio Management constructs portfolios. The boundary is clean: ValuationPackets flow from IV to PM as expected-return inputs.

### 3. Separate from UATS

Portfolio Management produces the portfolio plan. UATS executes it. Portfolio Management does not concern itself with order routing, market microstructure, or strategy runtime. It produces PortfolioConstructionPacks that UATS consumes.

### 4. Implementation is first-class

Unlike many PM frameworks that optimize in theory and hand-wave implementation, this system treats costs, constraints, signal decay, and transfer coefficient as first-class components with dedicated skills. The implementation layer is not an afterthought - it is roughly a quarter of the skill set.

### 5. Governance is built-in

Red-teaming, bias checking, assumption auditing, and committee documentation are not optional add-ons. They are built into the orchestration flow as mandatory steps. Every PortfolioConstructionPack carries a governance memo and red-team summary.

### 6. Game-theoretic awareness is structural

Crowding checks, capacity-alpha modeling, multi-manager overlap analysis, and fee normalization are dedicated skills - not post-hoc adjustments. The system is designed to reason about adversarial and strategic dynamics as part of the core construction flow.

### 7. Grinold & Kahn as the theoretical backbone

The skill library is grounded in *Advances in Active Portfolio Management*. Chapter references are preserved in each skill's metadata. This provides institutional credibility, methodological rigor, and a shared vocabulary for portfolio construction concepts.

---

## Design Principles Summary

| Principle | Implementation |
| --------- | -------------- |
| Classify before constructing | Intake and approach-selector skills run first |
| Methodology fits the mandate | Multiple construction approaches, adaptively selected |
| Risk budgets are explicit | Risk-budget-router and constraint skills produce transparent allocations |
| Implementation is modeled | Trade-rate, turnover, transfer-coefficient skills are mandatory |
| Attribution closes the loop | Ex ante and ex post attribution with reconciliation |
| Governance is enforced | Red-team, bias-check, and auditor skills are mandatory |
| Game theory is structural | Crowding, capacity, overlap, and fee skills are first-class |
| Outputs are machine-readable | PortfolioConstructionPack is structured for UATS consumption |
| Provenance is preserved | Every output traces to upstream inputs (ValuationPacket IDs, UniverseCard IDs, etc.) |
