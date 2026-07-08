# Community Detection in the European Football Transfer Network

*Utrecht University — Network Science 2026 — Topic 6*

This project builds a network from real player-transfer data between European football
clubs and studies its **community structure**: do clubs naturally cluster by national
league, or do transfer flows cut across borders? A full pipeline detects communities with
several algorithms, compares them against the "ground truth" of national leagues, tests
how stable the structure is, and validates the methods on synthetic benchmarks.

All code lives in [`leagues1.ipynb`](leagues1.ipynb).

## The data

Transfer records come from the [ewenme/transfers](https://github.com/ewenme/transfers)
dataset (one CSV per league, stored in [`data/`](data/)). Each row is a single transfer
with fields such as `club_name`, `club_involved_name` (the other club), `fee_cleaned`,
`transfer_movement` (`in`/`out`), `season`, and `country`.

The analysis covers the **2015–2022** seasons across eight leagues:

| Country | League |
|---|---|
| England | Premier League, Championship |
| Spain | Primera División |
| Italy | Serie A |
| Germany | 1. Bundesliga |
| France | Ligue 1 |
| Portugal | Liga NOS |
| Netherlands | Eredivisie |

## The network

- **Nodes** = clubs
- **Edges** = a transfer relationship between two clubs
- **Edge weight** = number of players transferred between the two clubs

The graph is **undirected and weighted**. To avoid double counting, only inbound
transfers are used, self-loops and missing counterparts are dropped, isolated nodes are
removed, and the analysis runs on the largest connected component.

## What the pipeline does

The notebook is organised into self-contained sections:

1. **Data loading & network construction** — load and filter the CSVs, build the weighted club graph, attach league/country metadata.
2. **Community detection** — run and compare five algorithms:
   - Louvain
   - Greedy modularity (Clauset–Newman–Moore)
   - Label propagation
   - Kernighan–Lin bisection
   - Girvan–Newman (on small enough graphs)
3. **Ground-truth comparison** — score detected communities against national leagues using **NMI** and **ARI**, with a per-community league breakdown.
4. **Robustness analysis** — measure how community structure holds up under:
   - removal of hub clubs (top transfer nodes),
   - random edge noise (5–30% edges removed, repeated over many trials),
   - edge-weight thresholding.
5. **Random baseline** — compare the real modularity against Erdős–Rényi null-model graphs and report a Z-score for significance.
6. **Temporal analysis** — sliding 3-season windows to track how modularity and community membership evolve over time.
7. **LFR benchmark** — validate the algorithms on synthetic graphs with a known ground truth and a tunable mixing parameter μ.
8. **Visualisation** — network plots coloured by community, an algorithm-agreement heatmap, temporal-evolution charts, and LFR performance curves (saved to [`figures/`](figures/)).

## Key metrics used

- **Modularity** — quality of a partition (higher = stronger community structure).
- **NMI** (Normalized Mutual Information) and **ARI** (Adjusted Rand Index) — agreement between two partitions (detected vs. ground truth, or across algorithms/trials).

## Running it

1. Download the dataset from [ewenme/transfers](https://github.com/ewenme/transfers) and place the CSVs in `data/` (already included here).
2. Install dependencies:
   ```bash
   pip install networkx python-louvain matplotlib numpy pandas scikit-learn
   ```
3. Open `leagues1.ipynb` in Jupyter and run all cells (the first cell installs `python-louvain` automatically). Generated figures are written to `figures/`.

Configuration (leagues, season range, edge-weight threshold, output paths) is set at the
top of the notebook and can be edited before running.
