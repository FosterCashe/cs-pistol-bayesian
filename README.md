# Does Winning the Pistol Round Predict Winning the Map?

A Bayesian logistic regression analysis of professional Counter-Strike matches. Built for ISyE 6420 (Bayesian Statistics) at Georgia Tech.

## The question

Counter-Strike rounds aren't all worth the same. The two **pistol rounds** (round 1 and round 16, the first round of each half) are played on equal, minimal economy — no rifles, no saved money — and the winner gets a significant economic head start on the rounds that follow. Community wisdom holds that pistol rounds are disproportionately important. This project quantifies that belief: **how much does winning a pistol round actually change a team's probability of winning the map**, and does the effect hold up once you control for skill difference and side?

## Approach

The analysis fits three models on the same data so they can be compared directly:

1. **Bayesian logistic regression with informative priors** — priors encode the community belief that pistol wins matter (centered on an odds ratio of ~2.7) and that the higher-ranked team has an edge, while staying skeptical about a CT-side advantage
2. **Bayesian logistic regression with vague priors** — a sensitivity check to see how much the priors actually drive the result
3. **Frequentist logistic regression** — a maximum-likelihood baseline for comparison

Outcome: did team 1 win the map. Predictors: won pistol 1, won pistol 2, standardized rank difference, started on CT side.

To isolate the pistol effect from raw skill mismatch, the data is subset to **closely-ranked matches (rank gap < 15)** — roughly 10,600 maps. If only blowouts were included, the better team would win the pistols *and* the map, and we couldn't separate the two.

## Data

Public HLTV professional match dataset ([Kaggle: CS:GO Professional Matches](https://www.kaggle.com/datasets/mateusdmachado/csgo-professional-matches)). Two files are merged on `match_id` + `map`: `results.csv` (match outcomes, rankings, side) and `economy.csv` (per-round winners, used to identify pistol-round winners).

The full dataset is ~28MB, so this repo includes **sampled extracts** (`data/*_sample.csv`) for inspection. Download the full files from the Kaggle link above to reproduce the complete analysis.

## Key findings

Descriptively, the pattern is stark — among closely-ranked teams:

| Situation | Map win rate |
|---|---|
| Won pistol 1 | 61.3% |
| Lost pistol 1 | 41.1% |
| Won **both** pistols | 70.7% |
| Lost **both** pistols | 31.4% |

The Bayesian model (informative priors) translates this into odds ratios:

| Predictor | Odds Ratio | 95% HDI |
|---|---|---|
| Won pistol round 1 | **2.36** | 2.18 – 2.55 |
| Won pistol round 2 | **2.26** | 2.09 – 2.45 |
| Rank difference (per SD) | 1.26 | 1.21 – 1.31 |
| Started CT side | 1.01 | 0.93 – 1.09 |

Takeaways:

- **Each pistol win roughly doubles the odds of winning the map** (~2.3x), even between evenly-matched teams. The two pistols matter about equally.
- **CT-side advantage effectively vanishes** at the map level (OR 1.01, HDI straddles 1.0). The skeptical prior was justified — whatever round-level CT edge exists doesn't carry to map outcomes in this era of the data.
- **Priors barely moved the result.** Informative and vague priors produced nearly identical posteriors (e.g. pistol-1 coefficient 0.857 vs 0.856), and both matched the frequentist estimate. With ~10k observations the data dominates — which is itself the correct Bayesian conclusion to report: the likelihood swamps the prior at this sample size.

All models sampled cleanly (R-hat = 1.00, high effective sample sizes, no divergences).

## Files

| File | Description |
|---|---|
| `Bayesian-CounterStrike-Project.ipynb` | Full analysis notebook with saved outputs |
| `Bayesian-CounterStrike-Project.html` | Rendered notebook (view without running) |
| `data/results_sample.csv` | Sample of match results data |
| `data/economy_sample.csv` | Sample of per-round economy data |
| `requirements.txt` | Dependencies |

## Running it

```
pip install -r requirements.txt
```

Download the full `results.csv` and `economy.csv` from Kaggle into `data/`, then run the notebook top to bottom. Sampling takes ~30 seconds per Bayesian model.

## Method notes

- Standardizing `rank_diff` makes the prior scale interpretable (coefficient is per standard deviation of rank gap)
- 7-day style attribution doesn't apply here; each map is one observation with a clean binary outcome
- The closely-ranked subset is the key design choice — it's what lets the pistol effect be read as something beyond "better team wins"
