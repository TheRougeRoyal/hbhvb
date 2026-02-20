# Entropy-Regularized Shortest Paths on DAGs

This repository studies entropy-regularized shortest paths on directed acyclic graphs (DAGs), with theory-backed bounds and reproducible experiments.

For source `s`, sink `t`, path cost `C(π)`, and temperature `T > 0`, the soft value is:

`d_T(s) = -T log( sum_{π:s->t} exp(-C(π)/T) )`

As `T -> 0+`, `d_T(s)` approaches the classical shortest-path cost `d*(s)`.

## What This Project Contains

- Core implementations for classical shortest paths (`d*`).
- Soft shortest paths (`d_T`) via stable log-sum-exp dynamic programming on DAGs.
- Theorem-based soft-hard gap upper bound: `T log(1 + N_sub exp(-Δ/T))`.
- Deterministic experiments producing plots and CSVs.
- Unit tests validating equivalence and bound behavior.

## Repository Structure

- `src/graph.py`: DAG wrapper + JSON loader.
- `src/classical_shortest_path.py`: Dijkstra/Bellman-Ford wrappers + classical cost helper.
- `src/entropy_regularized.py`: soft shortest-path routines on DAGs.
- `src/bounds.py`: path statistics, path-cost enumeration, Theorem III.1 bound utility.
- `experiments/temperature_analysis.py`: gap vs temperature, exponential convergence, node-level classical vs soft comparison.
- `experiments/cost_margin.py`: effect of increasing cost margin `Δ`.
- `experiments/path_multiplicity.py`: effect of increasing number of paths.
- `run_all_experiments.py`: runs all experiments and prints CSV summaries.
- `tests/`: theorem and numerical-validation tests.
- `data/sample_dag.json`: sample DAG.
- `results/`: generated plots/CSVs.
- `notebooks/`: optional exploratory notebooks (no separate README).

## Installation

Use Python 3.10+ (3.11/3.12 recommended).

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

## Quick Start

Run all experiments:

```bash
python run_all_experiments.py
```

This regenerates artifacts in `results/`, including:

- `temperature_gap.csv`, `temperature_gap.png`
- `exponential_convergence.csv`, `exponential_convergence.png`
- `classical_vs_soft.csv`, `classical_vs_soft.png`
- `cost_margin.csv`, `cost_margin.png`
- `path_multiplicity.csv`, `path_multiplicity.png`

## Testing

Run the test suite:

```bash
python -m pytest -q
```

The tests verify:

- DP soft value equals brute-force partition-function computation on small DAGs.
- Theorem III.1 bound upper-bounds `d*(s) - d_T(s)`.
- Small-temperature convergence to classical shortest-path cost.
- Random DAG consistency checks against classical shortest-path implementations.

## Minimal API Example

```python
import networkx as nx
from src.classical_shortest_path import shortest_path_cost
from src.entropy_regularized import soft_shortest_path
from src.bounds import soft_hard_gap_bound, compute_path_stats

g = nx.DiGraph()
g.add_edge("s", "a", weight=1.0)
g.add_edge("a", "t", weight=1.0)
g.add_edge("s", "b", weight=1.1)
g.add_edge("b", "t", weight=1.0)

d_star = shortest_path_cost(g, "s", "t")
d_T, _ = soft_shortest_path(g, "s", "t", temperature=0.5)
stats = compute_path_stats(g, "s", "t")
bound = soft_hard_gap_bound(delta=stats["delta"], n_sub=stats["n_sub"], temperature=0.5)

print(d_star, d_T, d_star - d_T, bound)
```

## Input Format for JSON DAGs

Expected JSON fields:

- `nodes`: list of node IDs
- `edges`: list of `[u, v, weight]`
- `source`: source node ID
- `sink`: sink node ID

Example: `data/sample_dag.json`.

## Assumptions and Limits

- Soft shortest-path implementation expects a DAG.
- Path-statistics helpers use full path enumeration, which can become expensive for large DAGs with many paths.
- Edge weights default to `1.0` if missing.

## Reproducibility Notes

- Scripts set NumPy and Python `random` seeds for deterministic experiment generation.
- Numerical behavior depends on package versions pinned in `requirements.txt`.

## Citation

See `CITATION.cff` for citation metadata.
