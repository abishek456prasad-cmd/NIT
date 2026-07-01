# JFDS Experiment — 5-Notebook Series

This replaces the original single 83-cell `Experiment_on_jfds_updated.ipynb` with five focused
notebooks, restructured against the publishability priorities discussed:

| # | Notebook | Priority tier | Depends on |
|---|---|---|---|
| 1 | `01_theory_and_hypothesis.ipynb` | Tier 1 | none — standalone |
| 2 | `02_main_experiments.ipynb` | — (core pipeline) | MovieLens 1M at `../dataset/movie_lens` |
| 3 | `03_ablation_and_complexity.ipynb` | Tier 2 | `jfds_artifacts.pkl` from `02` |
| 4 | `04_significance_and_robustness.ipynb` | Tier 1/2 (core claim) | `jfds_artifacts.pkl` from `02` |
| 5 | `05_appendix_supplementary.ipynb` | Tier 3 | `jfds_artifacts.pkl` from `02` |

## Run order

1. **Run `02_main_experiments.ipynb` first, start to finish.** It trains SVD and BPR, runs the
   Optuna λ-search, generates Top-K / JFDS / MMR / Fairness-only recommendation lists, and — as its
   last cell — writes `jfds_artifacts.pkl` (a few hundred MB) into the working directory.
2. **`03`, `04`, and `05` can then be run in any order** (or in parallel) — each loads
   `jfds_artifacts.pkl` in its own "Setup" cell instead of re-running training / re-ranking.
3. **`01` has no dependency on any other notebook** and can be read/run first or last.

If you don't have `jfds_artifacts.pkl` yet, run `02` once. If you re-run `02` (e.g. after changing
`RANDOM_SEED` or a hyperparameter), re-run `03`–`05` afterward so their pickled inputs stay in sync.

## Why the split

- **`01`** exists so the JFDS equation, the squared-gap fairness term, and the Adaptive-λ schedule
  are *derived*, not just implemented — and states three falsifiable hypotheses (H1, H2, H3) that
  the rest of the series tests.
- **`02`** is the minimal pipeline needed to reproduce the headline "does JFDS work" result.
- **`03`** is new content: an ablation of JFDS's own internal components (fairness term, diversity
  term, gap exponent) — distinct from the MMR/Fairness-only *baseline* comparison in `02` — plus a
  Big-O and empirical runtime analysis across every re-ranking strategy used in the project.
- **`04`** carries every result a reviewer would call load-bearing: paired significance tests with
  multiple-comparison correction, bootstrap CIs, group-fairness by demographic, aggregate-diversity
  / exposure-equity analysis, the Adaptive-λ variant, and a 30-resample robustness check — plus an
  expanded conclusion (limitations, practical implications, future work).
- **`05`** holds the exploratory analyses (Pareto frontier, scaling law, regime analysis,
  conservation invariant, coupling coefficient, information-theoretic checks, schedule-shape
  sensitivity) that are interesting context but not part of the confirmatory claim.

## A note on the code

`03`–`05` re-declare the core scoring/metric functions (`greedy_rerank`, `jfds_score`,
`mmr_score`, `ild`, `precision_recall_ndcg`, etc.) in their own "Setup" cells rather than importing
them from `02`, so each notebook is independently runnable given only `jfds_artifacts.pkl`. These
re-declarations are kept byte-identical to their originals in `02` — if you modify a scoring
function, update it in all notebooks that redeclare it (or better, factor it out into a shared
`jfds_common.py` module if you extend this project further).
