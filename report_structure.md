# Report Structure — Community Detection in the European Football Transfer Market

> Utrecht University — Network Science 2026

---

## 1. Introduction *(~0.5–1 page)*

### 1.1 Motivation & Research Context
- Football transfers as a real-world economic network: clubs = nodes, transfers = weighted edges
- Why community structure is expected: clubs transfer predominantly within national markets
- Why deviations are interesting: cross-national corridors, shared-ownership clusters, geographical/cultural proximity

### 1.2 Central Hypothesis
> Community detection algorithms applied to the transfer network should recover the national league structure, since clubs from the same domestic league trade players more frequently among themselves than with clubs from other countries. However, meaningful deviations — cross-national corridors, shared-ownership clusters, and cultural proximity effects — should create communities that do not align perfectly with league boundaries.

### 1.3 Research Questions
1. **Algorithm comparison**: How do Louvain, Greedy Modularity (CNM), Label Propagation, Kernighan-Lin, and Girvan-Newman compare on modularity, number of communities, and agreement with ground truth?
2. **Robustness**: How robust is the community structure to edge removal (random noise), node removal (iterative removal of top-k hub clubs), and threshold variation (minimum transfer count)?

---

## 2. Data & Network Construction *(~1–1.5 pages)*

### 2.1 Data Source & Scope
- [ewenme/transfers](https://github.com/ewenme/transfers) repository — 9 European leagues
- Leagues: Premier League, Championship, La Liga, Serie A, Bundesliga, Ligue 1, Liga NOS, Eredivisie, Premier Liga
- Season window: 2015–2022 (~53k transfer records from ~162k total)

### 2.2 Data Cleaning
- **Duplicate records**: each transfer appears in both buyer's and seller's league file → filter to `transfer_movement == "in"` only
- **Club name normalization**: fuzzy matching (`rapidfuzz`) + manual CSV mapping to unify naming variants (e.g. "Wolves" ↔ "Wolverhampton Wanderers")
- **Self-loops**: removed after normalization
- **Youth/reserve teams**: retained (separate transfer market entities)

### 2.3 Network Model
- **Undirected**: a transfer between A and B represents a bilateral relationship; standard community detection operates on undirected graphs
- **Weighted**: edge weight = total number of transfers between two clubs
- **Single connected component**: largest CC used for analysis

### 2.4 Ground Truth
- Each club assigned to its **home league** based on the dataset it originates from
- External clubs (appearing only as `club_involved_name`) labeled **"Other"**
- 9 ground-truth labels: 8 leagues + "Other"
- **Limitation**: "Other" is a catch-all (~90% of nodes), which dilutes NMI/ARI scores

---

## 3. Basic Network Characterization *(~0.5 page)*

- Nodes, edges, density, clustering coefficient, connected components
- Degree distribution (plot) — heavy-tailed?
- Top hub clubs by weighted degree — which clubs dominate the transfer market?
- What the basic structure suggests about community presence before running detection algorithms

---

## 4. Community Detection: Algorithm Comparison *(~2–2.5 pages)*

**RQ1**: *How do different community detection algorithms compare on this network?*

### 4.1 Why Disjoint Communities?
- Each club belongs to exactly one league → disjoint partitioning is the natural model
- All five algorithms produce disjoint partitions

### 4.2 Algorithms & Motivation

| Algorithm | Approach | Null Model | Motivation |
|---|---|---|---|
| **Louvain** | Modularity optimization (multi-level greedy) | Configuration model | Gold standard for large networks; fast; hierarchical resolution |
| **Greedy Modularity (CNM)** | Modularity optimization (agglomerative) | Configuration model | Alternative optimizer; different greedy strategy → different local optima? |
| **Label Propagation** | Local heuristic (label spreading) | None | No modularity optimization → finds structure from a fundamentally different angle |
| **Kernighan-Lin** | Graph bisection | Min-cut heuristic | Produces exactly 2 communities → baseline/sanity check |
| **Girvan-Newman** | Edge betweenness removal (divisive) | Betweenness-based | Classic hierarchical approach; produces dendrogram; limited to smaller graphs |

### 4.3 Null Model Discussion
- Louvain and CNM optimize modularity under the **configuration model**: $Q = \frac{1}{2m}\sum_{ij}\left[A_{ij} - \frac{k_i k_j}{2m}\right]\delta(c_i, c_j)$
- Sensible for the transfer network: high-degree hubs naturally attract more connections, and the null model accounts for this
- **Resolution limit** (Fortunato & Barthélemy, 2007): small communities may be merged — relevant given that some leagues have fewer clubs in the network

### 4.4 Results

#### Summary Table
| Algorithm | Communities | Modularity | NMI (all) | ARI (all) | NMI (league-only) | ARI (league-only) |
|---|---|---|---|---|---|---|
| Louvain | ... | ... | ... | ... | ... | ... |
| Greedy (CNM) | ... | ... | ... | ... | ... | ... |
| Label Propagation | ... | ... | ... | ... | ... | ... |
| Kernighan-Lin | ... | ... | ... | ... | ... | ... |
| Girvan-Newman | ... | ... | ... | ... | ... | ... |

#### Visualizations
- **Grouped bar chart**: Modularity, NMI, ARI across all 5 algorithms
- **Radar chart**: algorithm profiles at a glance
- **NMI heatmap**: pairwise agreement between algorithms — do they find the same structure?
- **Network plots**: colored by each algorithm's partition (at least for Louvain)

#### Discussion Points
- Which algorithms agree? (Louvain ≈ CNM? Label Propagation finds something different?)
- Why Label Propagation may collapse to 1 community (lack of resolution mechanism)
- Kernighan-Lin as a lower bound (forced bisection)
- Girvan-Newman: which hierarchy level maximizes modularity?

### 4.5 Community Composition
- For the best-performing algorithm (Louvain): breakdown of each community by league
- Does each community correspond to one dominant league?
- Which leagues get merged? Which get split?
- Bar charts showing league composition per community

---

## 5. Robustness & Sensitivity Analysis *(~2–2.5 pages)*

**RQ2**: *How robust is the community structure?*

> [!IMPORTANT]
> This section addresses the core concern from Good et al. (2010): modularity optimization can find many near-optimal but structurally diverse solutions. We test whether the community structure is a stable feature of the network or an artifact of a specific algorithm run.

### 5.1 Edge Removal — Random Noise
- **Method**: Randomly remove 0–50% of edges, re-run Louvain, measure NMI and ARI vs. the baseline (unperturbed) partition. 20 trials per noise level.
- **Plots** (2×3 grid, row 1):
  - Modularity vs. % edges removed (with error bars)
  - NMI & ARI vs. baseline partition (stability)
  - Number of communities vs. % removed
- **Discussion**: At what noise level does the partition break down? Is there a threshold effect or gradual degradation? What does this tell us about the strength of the community signal?

### 5.2 Iterative Hub Removal — Node Removal
- **Method**: Rank clubs by weighted degree. Remove the top hub, re-run Louvain, record metrics. Repeat up to k = 50.
- **Plots** (2×3 grid, row 2):
  - Modularity, NMI, ARI vs. k (hubs removed)
  - Community count evolution
- **Discussion**: Do a few mega-hubs (e.g., Chelsea, Juventus, Benfica) hold the community structure together, or is it distributed across many clubs? Does removing English hubs specifically affect the English community? List the top-10 hubs and their leagues.

### 5.3 Edge-Weight Threshold Sweep
- **Method**: Keep only edges with weight ≥ t (t = 1, 2, 3, …, 20). Re-run Louvain, compute Modularity, NMI(GT), ARI(GT).
- **Plot**: Metrics vs. threshold + nodes remaining (secondary axis)
- **Discussion**: Do weak edges (single transfers) add noise or signal? Is there an optimal threshold that maximizes ground-truth agreement? How does the network's size and density change?

### 5.4 Summary of Robustness Findings
- Synthesize: is the community structure a **robust, persistent feature** of the transfer network, or is it fragile?
- Connect to Good et al.: modularity landscape degeneracy — does this affect our findings?

---

## 6. Discussion & Interpretation *(~1.5 pages)*

### 6.1 Do Transfer Communities Match Leagues?
- The central hypothesis: **partially confirmed**
- Each Louvain community is dominated by one national league
- But communities also absorb external clubs (satellite clubs, feeder clubs, frequent trading partners)
- Communities reveal **"national transfer ecosystems"** rather than strict league boundaries

### 6.2 Meaningful Deviations from League Structure
*(These are the deviations your proposal predicted — discuss which ones you actually found)*

- **Cross-national transfer corridors**: e.g., Portuguese clubs clustering with Brazilian clubs (linguistic/colonial ties), Dutch clubs linking with Belgian clubs
- **Shared-ownership / multi-club groups**: e.g., Red Bull network (Salzburg, Leipzig), City Football Group — do these clubs appear in unexpected communities?
- **Geographical/cultural proximity**: e.g., Scandinavian clubs, Balkan clubs — do they form sub-communities within the "Other" group?

### 6.3 Why NMI/ARI Are Low (and Why That's Interesting)
- The "Other" category (~90% of nodes) creates a massive ground-truth group that no algorithm can meaningfully partition
- When restricted to **league clubs only**, NMI/ARI improve significantly — the algorithms *do* recover league structure for known clubs
- The low full-network scores are not a failure — they reflect the **richness of the community structure beyond national boundaries**

### 6.4 Algorithm Comparison: Key Takeaways
- Louvain and CNM find similar structure (high pairwise NMI) → modularity-based methods are consistent
- Label Propagation struggles without a resolution parameter → sensitive to initialization
- Girvan-Newman provides a hierarchical view but is computationally limited
- Kernighan-Lin's forced bisection reveals the dominant east-west or north-south divide

---

## 7. Conclusion *(~0.25 page)*

- The transfer network exhibits **robust, meaningful community structure** aligned with national leagues
- Deviations from league boundaries reveal cross-national transfer corridors and cultural proximity effects
- Louvain provides the best balance of quality, speed, and robustness
- The community structure is stable under moderate edge noise and hub removal, indicating it is a genuine feature of the transfer market rather than a methodological artifact

---

## References

- Blondel, V. D. et al. (2008). *Fast unfolding of communities in large networks* — Louvain
- Clauset, A., Newman, M. E. J., & Moore, C. (2004). *Finding community structure in very large networks* — CNM
- Raghavan, U. N., Albert, R., & Kumara, S. (2007). *Near linear time algorithm to detect community structure* — Label Propagation
- Girvan, M. & Newman, M. E. J. (2002). *Community structure in social and biological networks* — Girvan-Newman
- Kernighan, B. W. & Lin, S. (1970). *An efficient heuristic procedure for partitioning graphs* — KL Bisection
- Good, B. H. et al. (2010). *Performance of modularity maximization in practical contexts*
- Karrer, B. et al. (2008). *Robustness of community structure in networks*
- Rosvall, M. & Bergstrom, C. T. (2008). *Mapping Change in Large Networks*
- Fortunato, S. & Barthélemy, M. (2007). *Resolution limit in community detection*
- Reijnders, D. et al. — *Ocean Surface Connectivity in the Arctic* (multi-solution overlap)

---

## Appendix: Checklist — Proposal ↔ Report Mapping

| Proposal Element | Section |
|---|---|
| Central hypothesis (leagues as communities) | §1.2, §6.1 |
| Expected deviations (corridors, shared-ownership, cultural proximity) | §1.2, §6.2 |
| RQ1: Algorithm comparison (Louvain, CNM, LPA, KL, GN) | §4 |
| Metrics: modularity, # communities, ground-truth agreement | §4.4 |
| RQ2: Robustness — edge removal | §5.1 |
| RQ2: Robustness — iterative hub removal (top-k) | §5.2 |
| RQ2: Robustness — threshold variation | §5.3 |
| Data cleaning, network construction | §2 |
| Ground truth definition & limitations | §2.4, §6.3 |
