# Pre-Immune Window of Monolayer Neoplasia — code and data

Simulation code and complete run data for *The Pre-Immune Window of Monolayer Neoplasia*
(Preface, Section 0, Parts I–II, Chapters 1–4). Everything reported in Chapters 1, 2 and 4
of the paper can be reproduced from this repository.

## What is here

```
code/     the simulation engine, the batch runners, and the interactive bench
data/     the five CSV files behind every figure and table in Chapters 1–2 and 4
figures/  the figures as published, regenerated from data/ by code/make_figures.py
```

## The engine

The engine is an agent-based model of a monolayer epithelium on a hexagonal lattice of
76 × 79 = 6004 nodes. A cell owns 1–4 nodes; tension (the mean territory size of a cell
and its neighbours) governs the awakening of reserve cells and division; glucose diffuses
across the sheet and is consumed in proportion to each cell's metabolic state; division is
asymmetric. Two parameters distinguish the reverted lineage:

| Parameter | Meaning |
|---|---|
| `revStore` | ceiling of the reverted cell's glycogen store, as a fraction of a differentiated cell's |
| `oxyRev` | probability that a newborn reverted cell survives the one-time oxygen filter at birth; `1.0` disables the filter |

The engine is byte-for-byte identical in all three runner files, including the order of
random-number-generator calls, so runs are comparable across series. A second implementation
of the same model, in JavaScript, is included as an interactive bench — see below. The bench
is not required to reproduce anything here, and its geometry of cell occupancy differs, so it
is not a step-for-step replica of the Python runs.

## The runners

| File | Series it produced |
|---|---|
| `code/run_batch_oxygrid.py` | oxygen grid, no wound: `oxyRev` ∈ {0.30 … 0.50}, `revStore` = 0.33, 500 runs |
| `code/run_batch_oxygrid_wound.py` | oxygen grid under wounding: `oxyRev` ∈ {0.35 … 0.50} × {bounded, chronic}, 800 runs |
| `code/run_batch_missing.py` | runner for series S0–S3, described in `code/SERIES_S0_S3.md` and in Appendix A of the paper; adds 24-hour trajectory logging and a paired wound interval. These series were not run; the paper does not depend on them. |

The bench and its driver are listed separately under *The interactive bench* below.

Each runner resumes from its own CSV, so an interrupted series can be continued by
launching the same command again.

```bash
pip install numba          # optional but strongly recommended; the pure-Python fallback is very slow
python code/run_batch_oxygrid.py
```

## The interactive bench

`code/epithelium_bench.html` is a single self-contained HTML file — no build step, no
dependencies. Open it in a browser and the sheet runs: pause, change any parameter, inflict
a wound, watch the reverted clone spread or fail. `code/bench_headless.js` drives the same
file in headless Chromium for a full 20,000 h horizon in about seven minutes.

```bash
npm i playwright
node code/bench_headless.js --store 0.33 --oxy 0.40 --hours 20000
```

`code/BENCH.md` documents the controls, the parameters used in the paper, how the bench
relates to the Python engine, and which passages of the notes inside the page are stale.

## The data

| File | Runs | revStore | oxyRev | Scenarios |
|---|---|---|---|---|
| `runs_series_baseline_033_2026-09.csv` | 300 | 0.33 | 1.0 (off) | none, bounded, chronic |
| `runs_series_glycogen_grid_none_2026-09.csv` | 300 | 0.20, 0.25, 0.30 | 1.0 (off) | none |
| `runs_series_glycogen_grid_wound_2026-09.csv` | 600 | 0.20, 0.25, 0.30 | 1.0 (off) | bounded, chronic |
| `runs_series_oxygrid_2026-08-29.csv` | 500 | 0.33 | 0.30–0.50 | none |
| `runs_series_oxygrid_wound_2026-09.csv` | 800 | 0.33 | 0.35–0.50 | bounded, chronic |

2500 runs in total. One row per run. Common settings across every series: horizon 20,000 h,
establishment threshold 500 cells in the largest connected reverted cluster checked every
24 h, `pRevert` = 1/1000 per division, 100 seeds (0–99) per parameter combination. Wound
protocol where applicable: first wound at 1000 h, defect 200 cells, interval drawn per run
from {12, 24, 48} h; `bounded` stops wounding at the first threshold crossing, `chronic`
continues to the end of the horizon.

Columns worth noting: `final_clone_size` (largest connected reverted clone at the end),
`clone_established_at_h` (empty if the threshold was never crossed), `rev_total` (reversion
events), `rev_oxy_death` (would-be reverted cells culled by the oxygen filter at birth),
`rev_lost` (reverted cells lost through loss of niche), `final_density` (live cells / 6004).

## Figures

```bash
pip install pandas numpy matplotlib scipy
python code/make_figures.py
```

Regenerates all eleven figures of the paper into `figures/` — `fig1_1` … `fig1_5`, `fig2_1` …
`fig2_5` and `fig4_1` — directly from `data/`. The script also prints the two statistics quoted
in Chapter 4: niche-loss mortality per surviving birth, 10.4% in runs where no clone became
established against 2.1% where one did, and a Spearman rank correlation of −0.55 between that
quantity and clone size. Reproducing those three numbers is the quickest check that the data
and the code in this repository belong together.

## Scope

The parameter grid answers the questions posed in Chapters 1 and 2 and is not a complete
factorial exploration. In particular, full glycogen (`revStore` = 1.0) with an active oxygen
filter (`oxyRev` < 1) was not run; see the paper's Afterword and Appendix A. Numerical values
from these runs are coordinates within this parameterization, not calibrated quantities — the
model is intended to fix signs and orderings, and the paper states this explicitly (Chapter 4,
Rule 7).

## Authors

Dmitrochenko Y., M.D. — model, simulations, paper.
Demi S. L., Data Scientist — batch runs, data handling.

## Citation

See `CITATION.cff`. If you use the data or the engine, please cite the paper and this
repository.

## License

Code: MIT (see `LICENSE`). Data (`data/`) and figures: CC BY 4.0.
