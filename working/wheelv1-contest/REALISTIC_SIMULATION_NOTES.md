# Realistic Contest Simulation Notes

## What changed

The simulation tab now uses a **Monte Carlo model on top of the real `spin()` engine** instead of a pool-level EV shortcut.

It now accounts for:

- prize `qty` depletion;
- tier progression (`bronze/silver/gold`);
- per-player `max_return_pct`;
- `drop_after` / `drop_before`;
- hard budget guardrail via `min_margin`;
- actual prize selection weights;
- realistic interleaving of many players instead of one synthetic player.

## Player model

Default assumptions:

- **3000 simulated players**;
- heavy-tail purchase/spin distribution (many small buyers, fewer big buyers);
- player sessions are split into multiple bursts and shuffled to mimic real interleaving;
- defaults can be overridden through `/simulation?simulated_players=...&monte_carlo_runs=...`.

## Important finding from the current prize table

If all visible RNG prizes and current `qty` limits are used as-is, the total stock payout ceiling is:

- **4,022 total prize units**
- **£27,883 total max payout**

So for a large contest window, especially around **100,000 spins**, the stock cap dominates the economics.

Example with spin price **£3**:

- revenue at 100,000 spins = **£300,000**
- target payout at 70% RTP = **£210,000**
- actual max payout from current stock = **£27,883**

That means a 70% payout profile is **mathematically impossible** without increasing prize quantities or replenishing stock.

## Files touched

- `engine.py`
- `models.py`
- `database.py`
- `main.py`
- `admin.html`
- `simulate_realistic.py`
