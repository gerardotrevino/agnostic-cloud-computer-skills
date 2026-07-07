# Pre-Built Tokenomics Models for REX Protocol

## Model 1: Metal-Buying Flywheel Dynamics

### Problem
Model the self-reinforcing loop: $REX purchase → metal buying → commission → staking yield → buyback → price support → more $REX demand.

### Mathematical Formulation

Let:
- R(t) = cumulative raise at time t
- M(t) = metal AUM at time t = R(t) × leverage (10x)
- C(t) = commission revenue = M(t) × commission_rate (10%)
- S(t) = staking yield = C(t) × 0.30
- B(t) = buyback amount = C(t) × 0.10
- P(t) = token price
- D(t) = demand for $REX

System of ODEs:
```
dM/dt = R'(t) × leverage
dC/dt = dM/dt × commission_rate
dP/dt = f(S(t), B(t), D(t))  # price as function of yield, buyback, demand
dD/dt = g(S(t)/P(t))  # demand increases as yield/price ratio increases
```

### Key Insight
The flywheel has a **positive feedback loop**: lower price → higher yield ratio → more demand → higher price. This creates a natural price floor.

## Model 2: Airdrop Rate Optimization

### Problem
Find the optimal airdrop rate (α) that maximizes token distribution while keeping the airdrop "incidental" for regulatory purposes.

### Variables
- α = airdrop rate (% of metal purchase value given as $REX)
- V(t) = metal purchase volume at time t
- T_dist(t) = tokens distributed via airdrop = V(t) × α / P_rex
- T_pool = total tokens available for airdrop (100M from Community/Airdrop)
- t_exhaust = time to exhaust airdrop pool

### Constraints
- α must be small enough to be "incidental" (regulatory): α ≤ 1%
- α must be large enough to be meaningful incentive: α ≥ 0.1%
- Distribution must last long enough: t_exhaust ≥ 24 months
- Must not create excessive sell pressure: daily distribution < 0.1% of circulating supply

### Optimization
```
maximize: total_unique_holders(α)
subject to:
  0.001 ≤ α ≤ 0.01
  t_exhaust(α) ≥ 24 months
  daily_distribution(α) < 0.001 × circulating_supply
```

## Model 3: Price Floor Stability Analysis

### Problem
Analyze whether the three price floors (NAV, yield, buyback) create a stable equilibrium or can be broken.

### NAV Floor
P_nav = AUM / circulating_supply
- If P < P_nav: tokens trade below the value of their backing → arbitrage opportunity
- Stability: STRONG — backed by physical metals in vaults

### Yield Floor
P_yield = annual_staking_yield / target_yield_rate
- If P drops: yield_rate = staking_yield / P increases → attracts buyers
- Stability: MODERATE — depends on continued protocol revenue

### Buyback Floor
P_buyback = f(quarterly_buyback_amount, market_depth)
- Protocol buys back and burns tokens quarterly
- Stability: MODERATE — depends on revenue continuation

### Combined Stability
The three floors create a **triple safety net**:
1. If P drops below yield floor → yield seekers buy
2. If P drops further below buyback floor → protocol buys
3. If P drops to NAV floor → arbitrageurs buy (tokens worth less than metal backing)

### Stress Test Scenarios
- 50% AUM decline: NAV floor drops 50%, yield floor drops 50%
- Zero new mints for 6 months: buyback continues from existing revenue
- Whale dump (10% of supply): price recovers if fundamentals intact

## Model 4: Dual-Track Distribution Equilibrium

### Problem
Model the Nash equilibrium between Track 1 (direct presale) and Track 2 (metal-purchase airdrop) participants.

### Players
- Type A: Investors (want $REX exposure, may or may not want metals)
- Type B: Metal buyers (want metals, $REX is bonus)
- Type C: Arbitrageurs (exploit price differences between tracks)

### Payoff Functions
- Type A payoff: (TGE_price - purchase_price) × tokens_bought
- Type B payoff: metal_appreciation + α × metal_value × (TGE_price / airdrop_price)
- Type C payoff: |Track1_effective_price - Track2_effective_price| × volume

### Equilibrium Condition
In equilibrium, the effective price of $REX should be approximately equal across both tracks. If Track 2 gives cheaper $REX, arbitrageurs buy metals just for the airdrop. If Track 1 is cheaper, metal buyers switch to direct purchase.

## Model 5: Presale Window Game Theory

### Problem
Model investor timing decisions across 48-hour windows.

### The Timing Dilemma
- Earlier window = lower price = higher return
- But: earlier = higher risk (fewer milestones achieved)
- Each window has limited time (48 hours) creating urgency

### Nash Equilibrium Analysis
Rational investor buys in window W* where:
marginal_return(W*) = marginal_risk(W*)

The risk-adjusted return should be equal across all windows in equilibrium. If one window offers better risk-adjusted returns, rational investors pile in, which is self-correcting (window fills faster → next window opens).

## Python Script Templates

### Monte Carlo Simulation Template
```python
import numpy as np

def monte_carlo_tokenomics(n_simulations=10000):
    results = []
    for _ in range(n_simulations):
        # Sample uncertain parameters
        raise_amount = np.random.lognormal(mean=np.log(95e6), sigma=0.5)
        commission_rate = np.random.uniform(0.08, 0.12)
        aum_multiplier = np.random.uniform(8, 12)
        
        # Calculate outcomes
        metal_aum = raise_amount * aum_multiplier
        commission = metal_aum * commission_rate
        staking_yield = commission * 0.30
        buyback = commission * 0.10
        implied_fdv = staking_yield * 30  # 30x yield multiple
        
        results.append({
            'raise': raise_amount,
            'aum': metal_aum,
            'commission': commission,
            'fdv': implied_fdv
        })
    
    return pd.DataFrame(results)
```

### Sensitivity Analysis Template
```python
def sensitivity_analysis(base_params, param_name, range_pct=0.5, n_points=20):
    base_value = base_params[param_name]
    values = np.linspace(base_value * (1 - range_pct), base_value * (1 + range_pct), n_points)
    outputs = []
    for v in values:
        params = base_params.copy()
        params[param_name] = v
        outputs.append(calculate_model(params))
    return values, outputs
```
