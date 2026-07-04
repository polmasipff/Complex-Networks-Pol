# Structure of a Culinary Ingredient Network and the SIS Phase Transition on Random Graphs

Final project for the *Complex Networks* course. The work has two parts:

1. **Part I — Structure.** Characterization of the flavor network of Ahn *et al.* (ingredients linked when they share a flavor compound) and comparison with a degree-exact null model.
2. **Part II — Dynamics.** Characterization of the SIS non-equilibrium phase transition on configuration-model networks using the lifespan method (Mata *et al.*) and an exact Gillespie algorithm.

---

## Repository structure

```
submission_Masip/
├── README.md
├── notebooks/
│   ├── network1.ipynb                # Part I: structure + null-model comparison
│   ├── SIS_lifespan_ADAPTIVE.ipynb   # Part II: SIS data generation (Gillespie)
│   └── SIS_lifespan_FINAL.ipynb      # Part II: analysis, fits and final figures
├── data/
│   ├── network_gcc.gexf              # giant connected component of the flavor network
│   └── cm_doubleswap_results.npz     # saved output of the Part I null model (100 realizations)
├── figures/                          # figures used in the report
└── report/
    └── main_Masip.tex                # LaTeX source of the report
```

---

## Part I — Structure (`network1.ipynb`)

Reads the flavor network, restricts it to the giant connected component (`data/network_gcc.gexf`) and stores it in an adjacency-array structure (degree vector `D`, neighbour vector `V`, and per-node pointers). On this graph it computes:

- the degree distribution `P(k)` and its complementary cumulative `ccP(k)`;
- the average nearest-neighbor degree `k_nn(k)`, normalized by `kappa = <k^2>/<k>`;
- the clustering coefficient by degree `c(k)` and the global clustering;
- the community structure with the Louvain algorithm (modularity `Q`).

It then builds the **degree-exact null model** with the **double-edge-swap** method: starting from the real graph, pairs of edges are repeatedly swapped (rejecting any swap that would create a self-loop or a repeated edge), so every realization keeps the exact degree sequence, hubs included. Because the network is very dense the acceptance rate is low and only about 30% of the links can be rewired; `k_nn(k)`, `c(k)` and the modularity are averaged over 100 independent realizations and compared with the real values.

**Outputs:** the structural figures (`distribution.png`, `structural_descriptors.png`, `community_structure_double_column.png`, `clustering lin.png`) and the null-model results (`cm_doubleswap_results.npz`).

---

## Part II — SIS dynamics

The dynamical part is split into two notebooks: one that **generates** the simulation data and one that **analyzes** it and produces the final figures. The data is generated once and saved to disk, so the (heavy) simulations do not have to be repeated for every plot.

### 1. Data generation — `SIS_lifespan_ADAPTIVE.ipynb`

- Builds configuration-model networks with a power-law degree sequence `P(k) ∝ k^{-γ}`, `k_min = 4`, for `γ = 2.5` and `3.5` and sizes `N = 10^4 … 10^6`.
- Simulates the SIS dynamics with an **exact continuous-time Gillespie algorithm**, accelerated with a Numba kernel that keeps `O(1)`-update lists of infected nodes and active edges.
- Uses the **lifespan method**: each run starts from a single infected node of minimum degree and is classified as *endemic* when its coverage reaches `N/2`; otherwise its lifespan `τ` is recorded. For each `(γ, N)` it measures `P_end(λ, N)` and `⟨τ(λ, N)⟩`.
- Scans `λ` on an **adaptive grid** clustered around the peak of `⟨τ⟩`, and **saves each scan incrementally** to `sis_results/` (so an interrupted run can resume without losing work).

The production run used in the report (`adaptive_boost`, 50 000 runs per `λ` point, all seven sizes for both `γ`) was executed on a laptop; the notebook regenerates the same data if run with the production settings.

### 2. Analysis and figures — `SIS_lifespan_FINAL.ipynb`

- Loads and **pools** all the saved scans (reconstructing `⟨τ²⟩` where needed) with exact weighted averages.
- Locates the peak `λ_p(N)` of each lifespan curve and fits the drift `λ_p(N) = λ_c + a N^{-1/ν}` to obtain `λ_c` and `1/ν`.
- Extracts the peak-height exponents `γ_1/ν`, `γ_2/ν` and the order-parameter exponent `β/ν`, and produces the finite-size **data collapses**.
- Generates the final report figures (`sis_3x2_raw_and_fit.pdf`, `sis_point45_combined.pdf`, `sis_collapse_grid.pdf`) and the exponent tables.

---

## How to run

**Requirements:** Python 3, and `numpy`, `scipy`, `pandas`, `matplotlib`, `networkx`, and `numba` (for the Gillespie kernel).

```bash
pip install numpy scipy pandas matplotlib networkx numba
```

**Order:**

1. `notebooks/network1.ipynb` — Part I. Needs `data/network_gcc.gexf`. Produces the structural figures and the null-model comparison. (The 100 double-edge-swap realizations on this dense network take on the order of an hour; the result is also provided in `data/cm_doubleswap_results.npz` so the plots can be reproduced without rerunning.)
2. `notebooks/SIS_lifespan_ADAPTIVE.ipynb` — Part II data generation. Creates a `sis_results/` folder with the graphs and the per-`(γ, N)` scans. The full production run is long; the notebook saves incrementally and can be resumed.
3. `notebooks/SIS_lifespan_FINAL.ipynb` — Part II analysis. Reads `sis_results/`, does all the fits and writes the final figures and tables.

---

## Notes

- The SIS simulations are the computationally heavy step; the notebooks are designed so the data is generated once and saved, and the analysis/figures are then produced from the saved files.
- All figures in `figures/` are the ones referenced in `report/main_Masip.tex`.
