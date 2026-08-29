# Cognitive Shorts

An engagement prediction system for short video, taken from a notebook to four
deployable services &mdash; and a second project that went back and tested what it
claimed.

## The system &mdash; four services

| | Repository | The question it answers | Tests |
|---|---|---|---:|
| 1 | [Single Prediction](https://github.com/PSCRedefine/SinglePrediction) | Is the model right? | 43 |
| 2 | [Batch Prediction](https://github.com/PSCRedefine/BatchPrediction) | Does it hold up at volume? | 78 |
| 3 | [Model Info](https://github.com/PSCRedefine/ModelInfo) | Is the artefact in memory the one that shipped? | 70 |
| 4 | [Analytics Dashboard](https://github.com/PSCRedefine/AnalyticsDashboard) | Is it still healthy today? | 101 |

A leakage-safe feature pipeline, cost-aware model selection under a paired-bootstrap
tie test, an operating point chosen against a traffic budget rather than by maximising
F1, then model introspection and drift monitoring around it.

Four candidate models finished statistically tied &mdash; every gap's confidence interval
included zero &mdash; so operating cost broke the tie. The shipped artefact is **1,958x
smaller** and 34x faster than the runner-up, with no measurable loss.

`292 tests` · `CI on Python 3.11 / 3.12 / 3.13` · `MIT`

## The validation &mdash; [GroundTruth](https://github.com/PSCRedefine/groundtruth)

Cognitive Shorts shipped with a stated limitation: its offline lift had never been
validated against a randomized holdout. [KuaiRand](https://kuairand.com/) supplies one
&mdash; Kuaishou injected uniformly random videos into live recommendation feeds for two
weeks, so testing both mechanisms across the identical calendar window isolates
exposure bias from temporal drift.

**The ceiling was the dataset, not the problem.** 2 of 25 request-time features carried
signal on the synthetic data; **24 of 37** do on real logs, and a single feature reaches
**0.7486**. The 0.58&ndash;0.60 ROC-AUC ceiling was a property of the synthetic set's
corrupted joins, not of engagement prediction.

**Ranking transfers. Calibration does not.**

| Metric | Algorithmic | Random | |
|---|---:|---:|---|
| Lift @ top 23.9% | 3.428x | 3.363x | −2.0% — transfers |
| Mean predicted ÷ actual | 0.96x | **2.07x** | **breaks** |

A team validating offline would watch the ranking hold, conclude the model transfers,
and ship probabilities that are silently wrong.

**Calibration is a property of the exposure policy, not of the model.** A calibrator
fitted on randomized logs ships there and is rejected on served traffic, by almost
exactly the same factor in the other direction. The Brier gate &mdash; adopted in Cognitive
Shorts for an unrelated reason, to stop ECE being gamed by variance collapse &mdash; catches
this on its own, with no special-casing.

`ROC-AUC 0.8811` · `2.6M interactions` · `every figure reproduces from a committed JSON artefact`

---

Each repository documents what is known to be wrong with it before it documents what
works, and what it would need before carrying real traffic in `docs/PRODUCTION_READINESS.md`.
