# GOAL

## Mission

Build an **AI-native adaptive portfolio management system** - the **strategic portfolio brain** of Sigmance.

Portfolio Management is the system that translates intelligence into implementable portfolio decisions. Given views on assets, probability-weighted futures, and live market state, it constructs optimal portfolios with explicit risk budgets, signal weights, implementation efficiency targets, and governance frameworks. It is grounded in institutional portfolio theory (Grinold & Kahn, *Advances in Active Portfolio Management*) and built as a composable skill engine that adapts its methodology to each mandate.

---

## What It Is

A **skill-based portfolio management engine** with 53+ specialized skills spanning the full institutional PM workflow: mandate intake, alpha signal calibration, risk budgeting, dynamic portfolio construction, implementation optimization, performance attribution, smart beta analysis, and governance.

Each skill encodes a discrete portfolio management capability - from the Fundamental Law of Active Management diagnostics, to signal decay estimation, to nonlinear trading rule design, to multi-manager overlap risk analysis. An LLM orchestrator navigates the skill graph, adaptively selecting the right construction methodology, risk framework, and governance structure for each mandate based on its characteristics.

This is not an optimizer. This is the encoded portfolio judgment of an entire institutional investment team - knowing when to use mean-variance vs scenario-based construction, when factor tilts are better than pure alpha, when apparent diversification is illusory, when signal decay demands faster turnover, and when costs make that turnover destructive.

---

## Core Thesis

Portfolio management methodology is never one-size-fits-all. The same principle that drives Investment Valuation (the right pricing methodology depends on the asset) applies to portfolio construction: the right construction methodology depends on the mandate.

A long-only equity fund with a tracking-error target needs different PM technology than a leveraged multi-strategy allocation. A factor-tilt smart-beta product needs different governance than a concentrated alpha portfolio. A scenario-driven tactical overlay needs different signal weighting than a systematic quantitative strategy.

The system must:

- **Classify before constructing** - understand the mandate before choosing a methodology.
- **Adapt methodology to mandate** - mean-variance, scenario-based, factor-tilt, risk-parity, or blended approaches, each applied where they fit.
- **Budget risk explicitly** - tracking error, active risk, signal allocation, leverage, and capacity constraints must be transparent and governed.
- **Account for implementation** - paper portfolios are not real portfolios; costs, constraints, and transfer coefficient degradation must be modeled.
- **Red-team every construction** - crowding, false diversification, skill inflation, cost underestimation, and benchmark weakness must be challenged.
- **Govern the process** - scenario selection, signal weighting, and rebalance decisions must follow auditable governance rules.

---

## North Star

Given views on assets and scenarios, construct optimal portfolios with explicit risk budgets, signal weights, and implementation efficiency - bridging the gap between "what things are worth" and "what the portfolio should look like."

At maturity, Portfolio Management should support a full institutional workflow where:

- mandates are classified and their constraints, benchmarks, and policy parameters extracted,
- alpha signals are calibrated with breadth, skill, and decay awareness,
- risk is budgeted across signals, factors, and implementation stages,
- portfolios are constructed using the methodology best suited to the mandate,
- implementation is optimized with cost-aware trading rules and turnover policies,
- performance is attributed ex ante and ex post with gap analysis,
- governance is enforced through assumption audits, bias checks, red-teaming, and committee-ready documentation,
- the entire process improves as the machine learns from attribution outcomes and adapts.

---

## Role in the Sigmance Operating Loop

Portfolio Management sits at **step 5.5** in the Sigmance operating loop - between pricing and execution:

```text
event -> evidence -> present-state ripple map -> probabilistic future universes
    -> valuation / underwriting -> PORTFOLIO CONSTRUCTION -> portfolio action
        -> monitoring -> adaptation -> improved future decisions
```

- **Investment Valuation** answers: "What is this asset worth under different realities?"
- **Portfolio Management** answers: "Given what things are worth, what should the portfolio look like?"
- **UATS** answers: "How do we execute, maintain, and adapt that portfolio?"

Portfolio Management consumes:
- **ValuationPackets** (fair-value ranges, scenario-conditioned assumptions) as expected-return inputs
- **UniverseCards** (probability-weighted futures, triggers, falsifiers) for scenario-conditioned construction
- **WorldStateSnapshots** (live state, crowding signals, exposure maps) for risk model calibration
- **EvidenceBundles** (evidence, claims, contradictions) for governance grounding

Portfolio Management produces:
- **PortfolioConstructionPacks** (position sizes, risk budgets, signal weights, turnover policies, rebalance rules, constraint specs, governance memos, monitoring hooks) consumed by UATS

### C2 Role: Portfolio Strategist and Chief Investment Officer

Under C2 command, Portfolio Management is the **Portfolio Strategist and CIO** - the officer who translates the commander's strategic intent into portfolio architecture. "Go defensive with 3% tracking error" becomes a concrete construction with position sizes, signal weights, risk budgets, and turnover rules. "Show me where our diversification is illusory" triggers the crowding, overlap, and common-factor diagnostic stack.

### Game-Theoretic Dimension

Portfolio Management operates with awareness that:
- **Crowding compresses alpha** - if every fund owns the same value tilt, the tilt is not alpha, it is crowded beta.
- **Common factors hide correlation** - sleeves that look diversified may share hidden factor exposures that correlate in crises.
- **Capacity constrains scale** - strategy alpha decays as assets grow; risk budgets must reflect capacity-adjusted, not historical, alpha.
- **Fee structures obscure economics** - marketing-friendly fees can mask prohibitive effective costs on the active component.
- **Implementation creates market impact** - large positions create reflexive dynamics; the portfolio's own rebalancing generates signals for adversaries.
- **Edge decays** - signals that work get discovered and arbitraged; the construction must account for edge lifecycle.

---

## What Success Looks Like

Portfolio Management is succeeding when:

- every mandate gets the right construction methodology, not a forced one-size-fits-all optimizer,
- risk budgets are explicit, transparent, and traceable to mandate-level objectives,
- signal weighting accounts for decay, breadth, skill, and cost - not just historical Sharpe,
- implementation efficiency is modeled and attributed, not assumed away,
- red-teaming surfaces crowding, false diversification, and skill inflation before they cause losses,
- PortfolioConstructionPacks are structured, machine-readable, and consumed directly by UATS,
- attribution closes the loop between what the model predicted and what actually happened,
- governance documentation is committee-ready without manual assembly,
- the system compounds institutional knowledge rather than scattering it.

A strong success test: given a mandate, a set of ValuationPackets, a ScenarioRegistry, and a WorldStateSnapshot, the system can produce a fully governed PortfolioConstructionPack that UATS can execute - with explicit risk budgets, signal weights, turnover policies, rebalance rules, monitoring hooks, and a governance memo that a human investment committee would accept.

---

## What It Should NOT Become

Portfolio Management should not become:

- a **valuation or pricing engine** (that is Investment Valuation),
- a **scenario generator** (that is Multiverse),
- a **market data ingestion system** (that is Market Matrix),
- a **trade execution or strategy runtime** (that is UATS),
- a **black-box optimizer** that cannot explain its construction decisions,
- a **one-size-fits-all mean-variance machine** that ignores mandate-specific methodology,
- a system that **assumes away implementation** (costs, constraints, and transfer coefficient are first-class concerns, not afterthoughts),
- a system where **risk budgets are implicit** (every allocation must be explicitly justified and governed),
- a system that **ignores game-theoretic dynamics** (crowding, capacity decay, hidden overlap, and adversarial fee structures are real portfolio risks, not academic curiosities).

The goal is a **governed, adaptive, institutional-grade portfolio construction engine** that bridges intelligence into implementable capital decisions.
