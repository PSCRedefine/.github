# PSCRedefine

Two related projects on short-video engagement prediction. The first is an
off-policy evaluation study on real logs; the second is the serving stack it
grew out of.

## [GroundTruth](https://github.com/PSCRedefine/groundtruth) — unbiased offline evaluation on real logs

[KuaiRand](https://kuairand.com/) contains two weeks in which Kuaishou inserted
uniformly random videos into live feeds, giving a genuinely missing-at-random
test set. Training on algorithmic exposure and testing on both mechanisms over
the identical calendar window isolates exposure bias from temporal drift.

Findings so far:

- **Ranking transfers across exposure policies; calibration does not.**
  Lift at the top 23.9% of traffic: 3.428x on algorithmic exposure, 3.363x on
  random (−2.0%). Mean predicted ÷ actual probability: 0.96x on algorithmic,
  **2.07x** on random. A team validating offline would watch the ranking hold
  and ship probabilities that are wrong by a factor of two. Both findings
  replicate at KuaiRand-27K scale — **322M interactions**, evaluated in full,
  out of core — where the calibration break grows to 2.29x.
- **Off-policy estimators graded against the gold standard.** With item-level
  propensities, naive, IPS, SNIPS, direct method and doubly robust all miss
  the uniform policy's true value (3.0–3.9x off, tight intervals). The reverse
  direction decomposes the 3.8x exposure bias: 1.25x from which videos the
  policy picks, **3.04x from user-video matching** — the part item-level
  propensities cannot express. Unlogged propensities cannot be reconstructed.
- **A PyTorch two-tower ties LightGBM at feature parity** (AUC 0.8815 vs
  0.8811 on 1M+ rows); the pure retrieval tower reaches 0.8246 with no watch
  signal, so personalization alone carries most of the ranking power.
- **No single calibrator fixes both exposure policies.** A calibrator fitted
  on randomized logs is correct there and off on served traffic by roughly the
  same factor in the other direction.
- **The upstream project's "signal ceiling" was an artifact of its data.**
  The same feature-screening rule that kept 2 of 25 features on the synthetic
  set keeps 24 of 37 on real logs; `watch_ratio` alone reaches 0.7486 ROC-AUC.

`ROC-AUC 0.9147 at 322M rows` · six experiments, every figure reproduced from a
committed JSON artifact · `./scripts/get_data.sh` fetches the dataset

## Cognitive Shorts — the serving stack

Four services around one model: [SinglePrediction](https://github.com/PSCRedefine/SinglePrediction)
(train, select, serve), [BatchPrediction](https://github.com/PSCRedefine/BatchPrediction)
(per-row fault isolation at volume), [ModelInfo](https://github.com/PSCRedefine/ModelInfo)
(artifact introspection), [AnalyticsDashboard](https://github.com/PSCRedefine/AnalyticsDashboard)
(request logging and drift monitoring).

Built on a synthetic course dataset, which is where its discipline came from:
a leakage blacklist enforced by tests, chronological splits, paired-bootstrap
model selection with ties broken by serving cost, and an operating point set
against a traffic budget after finding the default 0.5 threshold degenerate.
When the data itself turned out to be corrupted — snapshot joins disagreeing
with logs on 97% of rows — measuring that honestly became the motivation for
GroundTruth.

`292 tests` · `CI on Python 3.11 / 3.12 / 3.13` · `MIT` · each repo states its
known gaps in `docs/PRODUCTION_READINESS.md`
