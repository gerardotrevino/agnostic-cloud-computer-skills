---
name: mathematical-modeling
description: Scientific mathematical modeling framework adapted from the MM-Agent (NeurIPS 2025) methodology. Provides a structured four-stage workflow for translating real-world problems into rigorous mathematical models, with a hierarchical method library (HMML) for selecting appropriate techniques. Use when working on tokenomics design, pricing models, economic simulations, sensitivity analysis, game theory, optimization, or any task requiring quantitative rigor. Especially important for REX Protocol tokenomics discussions.
---

# Mathematical Modeling Framework

Adapted from "MM-Agent: LLM as Agents for Real-world Mathematical Modeling Problem" (NeurIPS 2025, HKUST). The original framework achieved 11.88% improvement over human expert solutions and helped teams reach top 2.0% of 27,456 teams in MCM/ICM 2025.

## Four-Stage Workflow

Every mathematical modeling task MUST follow these four stages in order:

### Stage 1: Problem Analysis
- **Identify** the real-world problem, objectives, and constraints
- **Decompose** into subtasks (what needs to be modeled separately)
- **Catalog** available data, assumptions, and unknowns
- **Define** success criteria (what makes a good solution)
- **Output:** Clear problem statement with variables, constraints, and objectives

### Stage 2: Mathematical Modeling
- **Retrieve** appropriate methods from the HMML (see below)
- **Formulate** the mathematical model with explicit assumptions
- **Justify** why each assumption is reasonable
- **Define** all variables, parameters, and their domains
- **Write** the model in formal mathematical notation
- **Output:** Complete mathematical formulation ready for computation

### Stage 3: Computational Solving
- **Implement** the model in Python (NumPy, SciPy, pandas, matplotlib)
- **Validate** with sanity checks and edge cases
- **Iterate** using MLE-Solver approach: run → analyze errors → improve → re-run
- **Perform** sensitivity analysis on key parameters
- **Run** Monte Carlo simulations where uncertainty exists
- **Output:** Numerical results with confidence intervals

### Stage 4: Solution Reporting
- **Summarize** the modeling process and key decisions
- **Interpret** results in real-world terms (not just numbers)
- **Visualize** with clear charts and tables
- **Identify** limitations and areas of uncertainty
- **Recommend** actions based on findings
- **Output:** Structured report with findings, tables, and visualizations

## Hierarchical Mathematical Modeling Library (HMML)

### Domain 1: Optimization
Use when: finding the best allocation, schedule, or configuration.

| Method | When to Use | REX Application |
|---|---|---|
| Linear Programming | Constraints are linear, single objective | Token allocation across pools |
| Nonlinear Programming | Nonlinear relationships | Bonding curve design |
| Multi-Objective Optimization | Competing goals | Maximize distribution vs minimize dilution |
| Integer Programming | Discrete decisions | Number of presale windows |
| Constrained Optimization | Hard limits exist | Supply caps, vesting schedules |
| Convex Optimization | Provably optimal solutions needed | Fee structure optimization |

### Domain 2: Simulation
Use when: system is too complex for closed-form solutions, or uncertainty is high.

| Method | When to Use | REX Application |
|---|---|---|
| Monte Carlo Simulation | Probabilistic outcomes, many variables | Price scenarios, raise projections |
| Agent-Based Modeling | Multiple interacting participants | Market participant behavior, whale analysis |
| Discrete Event Simulation | Sequential events with timing | Presale window mechanics, 48-hour cycles |
| System Dynamics | Feedback loops, stocks and flows | Flywheel modeling, AUM growth |
| Bootstrapping | Estimate confidence from limited data | Revenue projection confidence intervals |

### Domain 3: Game Theory
Use when: multiple strategic actors with competing interests.

| Method | When to Use | REX Application |
|---|---|---|
| Mechanism Design | Designing incentive systems | Dual-track (presale + airdrop) incentives |
| Nash Equilibrium | Finding stable strategies | Investor timing decisions (which window) |
| Auction Theory | Price discovery mechanisms | Presale window pricing |
| Signaling Games | Information asymmetry | Tier gating by milestones |
| Cooperative Game Theory | Coalition formation | Staking pool dynamics |

### Domain 4: Financial Mathematics
Use when: valuation, pricing, or risk analysis.

| Method | When to Use | REX Application |
|---|---|---|
| Discounted Cash Flow | Valuing future cash flows | Token valuation from protocol revenue |
| Options Pricing | Asymmetric payoffs | SAFT as call option on $REX |
| Bonding Curves | Automated price discovery | Continuous token pricing |
| Risk-Neutral Pricing | Fair value under uncertainty | TGE price justification |
| Stochastic Processes | Price evolution over time | $REX price trajectory modeling |
| Yield Curve Analysis | Term structure of returns | Vesting schedule optimization |

### Domain 5: Dynamical Systems
Use when: modeling how systems evolve over time with feedback.

| Method | When to Use | REX Application |
|---|---|---|
| Ordinary Differential Equations | Continuous dynamics | Token supply/demand evolution |
| Feedback Loop Analysis | Self-reinforcing or self-correcting | Metal-buying flywheel stability |
| Stability Analysis | Will the system converge or diverge? | Price floor effectiveness |
| Phase Portraits | Qualitative behavior mapping | Market regime identification |
| Lotka-Volterra | Predator-prey dynamics | Buyer/seller ecosystem balance |

### Domain 6: Statistical Analysis
Use when: analyzing data, estimating parameters, or quantifying uncertainty.

| Method | When to Use | REX Application |
|---|---|---|
| Sensitivity Analysis | Which parameters matter most? | Commission rate, airdrop %, AUM growth |
| Regression Analysis | Relationships between variables | Price-volume, AUM-revenue |
| Bayesian Inference | Updating beliefs with evidence | Parameter estimation from market data |
| Time Series Analysis | Temporal patterns | Metal price forecasting |
| Hypothesis Testing | Comparing alternatives | Dual-track vs single-track performance |

### Domain 7: Network & Graph Theory
Use when: analyzing relationships, flows, or connectivity.

| Method | When to Use | REX Application |
|---|---|---|
| Flow Networks | Resource allocation through networks | Token flow between entities |
| Centrality Analysis | Identifying key nodes | Whale concentration risk |
| Community Detection | Finding clusters | Investor segmentation |

## Method Selection Process

When faced with a modeling task:

1. **Classify the problem type:** Is it optimization? Simulation? Valuation? Game theory?
2. **Check constraints:** Are there hard limits? Uncertainty? Multiple actors?
3. **Select primary method:** Choose the most appropriate from HMML
4. **Select supporting methods:** Often need 2-3 methods combined
5. **Validate choice:** Does this method handle the problem's key features?

## Python Implementation Standards

When implementing models computationally:

```python
# Required imports for mathematical modeling
import numpy as np
import pandas as pd
from scipy import optimize, stats, integrate
import matplotlib.pyplot as plt
from dataclasses import dataclass
from typing import Dict, List, Tuple

# Always define parameters as dataclasses for clarity
@dataclass
class ModelParameters:
    """All model parameters with units and valid ranges."""
    param_name: float  # units, valid range [min, max]

# Always include:
# 1. Parameter validation
# 2. Sanity checks on outputs
# 3. Sensitivity analysis on key inputs
# 4. Monte Carlo for uncertainty quantification
# 5. Clear visualization of results
```

## Key Principles

1. **Assumptions must be explicit and justified** — Never hide an assumption. State it, justify it, and test what happens if it's wrong.
2. **Models should be falsifiable** — Define what evidence would prove the model wrong.
3. **Sensitivity analysis is mandatory** — Always test: which parameter, if changed by 10%, changes the output the most?
4. **Monte Carlo for uncertainty** — When in doubt, simulate 10,000 scenarios.
5. **Interpret, don't just compute** — Every number must have a real-world meaning.
6. **Start simple, add complexity** — Begin with the simplest model that captures the key dynamics. Add complexity only when the simple model fails.
7. **Validate against known cases** — Test the model against scenarios where you know the answer.

## Reference Files

Load `references/tokenomics-models.md` for specific mathematical models pre-built for REX tokenomics analysis (flywheel dynamics, airdrop optimization, price floor stability).
