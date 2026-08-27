## Cognitive Shorts

Four repositories that form one prediction system for a short-video platform, and
the measurements behind it. Read in order, they follow a single question as it
moves out of a notebook and into production.

| | Repository | The question it answers |
|---|---|---|
| 1 | [Single Prediction](https://github.com/PSCRedefine/SinglePrediction) | Is the model right? |
| 2 | [Batch Prediction](https://github.com/PSCRedefine/BatchPrediction) | Does it hold up at volume? |
| 3 | [Model Info](https://github.com/PSCRedefine/ModelInfo) | Is the artefact in memory the one that was shipped? |
| 4 | [Analytics Dashboard](https://github.com/PSCRedefine/AnalyticsDashboard) | Is it still healthy today? |

**Start with Single Prediction.** It carries the modelling work: a leakage-safe
feature table, four candidate models separated first by a paired bootstrap and
then by operating cost, isotonic calibration applied because it measurably
helped, and a decision threshold chosen against a budget rather than against F1.

**The headline is a negative result.** On this data only watch behaviour predicts
engagement, so any truthful model is bounded near 0.60 ROC-AUC. Twenty-three of
twenty-five candidate features are statistically indistinguishable from noise.
The shipped model uses two of them and is 3 KB. Knowing which features are not
worth building a pipeline for is the most reusable thing here.

Each repository states what is known to be wrong or missing in its
**Limitations** section, and what it would need before carrying real traffic in
**`docs/PRODUCTION_READINESS.md`**.

All four are MIT licensed, tagged `v1.0.0`, and run their test suites on Python
3.11, 3.12 and 3.13 in CI.
