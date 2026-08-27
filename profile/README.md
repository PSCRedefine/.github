<h1 align="center">Cognitive Shorts</h1>

<p align="center">
  <em>An engagement prediction system for a short-video platform &mdash;<br>
  built to be measured, operated, and argued with.</em>
</p>

<p align="center">
  <a href="https://github.com/PSCRedefine/SinglePrediction/actions/workflows/tests.yml"><img alt="Single Prediction" src="https://github.com/PSCRedefine/SinglePrediction/actions/workflows/tests.yml/badge.svg"></a>
  <a href="https://github.com/PSCRedefine/BatchPrediction/actions/workflows/tests.yml"><img alt="Batch Prediction" src="https://github.com/PSCRedefine/BatchPrediction/actions/workflows/tests.yml/badge.svg"></a>
  <a href="https://github.com/PSCRedefine/ModelInfo/actions/workflows/tests.yml"><img alt="Model Info" src="https://github.com/PSCRedefine/ModelInfo/actions/workflows/tests.yml/badge.svg"></a>
  <a href="https://github.com/PSCRedefine/AnalyticsDashboard/actions/workflows/tests.yml"><img alt="Analytics Dashboard" src="https://github.com/PSCRedefine/AnalyticsDashboard/actions/workflows/tests.yml/badge.svg"></a>
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/python-3.11%20%7C%203.12%20%7C%203.13-blue">
  <img alt="Tests" src="https://img.shields.io/badge/tests-292-brightgreen">
  <img alt="Licence" src="https://img.shields.io/badge/licence-MIT-lightgrey">
</p>

---

I build machine learning systems, and I care most about the part that usually
gets skipped: knowing whether the thing actually works, and saying so plainly
when it does not.

These four repositories are **one system, not four projects**. Read in order,
they follow a single question as it moves out of a notebook and into production.

| | Repository | The question it answers | Tests |
|---|---|---|---:|
| 1 | [**Single Prediction**](https://github.com/PSCRedefine/SinglePrediction) | Is the model right? | 43 |
| 2 | [**Batch Prediction**](https://github.com/PSCRedefine/BatchPrediction) | Does it hold up at volume? | 78 |
| 3 | [**Model Info**](https://github.com/PSCRedefine/ModelInfo) | Is the artefact in memory the one that was shipped? | 70 |
| 4 | [**Analytics Dashboard**](https://github.com/PSCRedefine/AnalyticsDashboard) | Is it still healthy today? | 101 |

The task is predicting whether someone engages with a short video. The model is
not the interesting part &mdash; it is a two-feature logistic regression that
serialises to **3 KB**. Everything around the decision to ship it is.

---

## Four results worth stating plainly

**The signal is weak, and the first screen says so.** Only watch behaviour
predicts engagement in this data, so any truthful model is bounded near **0.60
ROC-AUC**. That ceiling was measured before the modelling started, not discovered
after it. Twenty-three of twenty-five candidate features are statistically
indistinguishable from noise, and each would have cost a pipeline, a backfill and
a monitoring surface to serve. *Measuring which features are not worth building
is the most reusable result here.*

**Four candidate models came out tied, so cost broke the tie.** The per-model
standard error (~0.0045) was larger than the entire spread between the four
candidates (0.0031). Ranking them by validation score would have been ranking
sampling noise, and the order flips on a different seed. A paired bootstrap
established the tie; operating cost chose the winner &mdash; **1,958x smaller and
34x faster** than the runner-up, with no measurable loss.

**The default threshold was wrong, and that shipped once.** The calibrated
model's maximum output is 0.389, so at the conventional 0.5 it flagged nothing at
all and F1 was 0.0. That was never a broken model; it was a broken threshold. The
fix was not a better constant but an operating-point sweep against a traffic
budget, with the chosen threshold published in the model metadata and read by the
service rather than hard-coded.

**Probabilities are rates, not ranks.** Calibration error is **0.0035**, down
from 0.0273. A row scoring 0.38 means roughly 38 in 100 such rows engage, so the
output can be budgeted against and summed across a batch rather than merely
sorted. Isotonic calibration was applied because Brier improved &mdash; a strictly
proper scoring rule &mdash; not because ECE looked better, since ECE can be driven
toward zero by collapsing every prediction to the base rate.

---

## What this exercises

| Area | What it demanded |
|---|---|
| **Leakage control** | `engagement_score` scores 0.836 ROC-AUC alone because it is computed from the outcome. An explicit blocklist excludes it and the five action columns, and a test asserts the blocklist never intersects the feature set. |
| **Statistical inference** | Paired bootstrap confidence intervals on AUC gaps; univariate screening with bootstrap intervals; permutation importance. The one-standard-error rule applied to operating cost. |
| **Probability calibration** | Isotonic regression fitted on validation, gated on Brier, with its quantisation to eight discrete outputs documented rather than hidden. |
| **Decision theory** | Operating-point sweeps, lift by decile, threshold selection under a traffic budget rather than F1 &mdash; which on a weak-signal problem degenerates into "treat everybody". |
| **Data forensics** | The three source files share identifier spaces but not attribute values: the same `user_id` disagrees on age across files on **97.1%** of rows, correlation 0.0014. Found by measurement, not assumption. |
| **API design** | FastAPI services with Pydantic contracts, per-row fault isolation, sanitised error surfaces, and a deliberate separation between the request contract and the feature set. |
| **Observability** | Request-logging middleware that cannot break a request, bounded ring-buffer storage, window aggregation with honest null semantics, and drift signals the transport layer cannot see. |
| **Release engineering** | Docker images with non-root users and profile-gated trainers, CI across three interpreters, traceability documents mapping every requirement to its implementation and evidence. |

---

<details>
<summary><b>The hard parts</b></summary>

<br>

**Knowing when a difference is not a difference.** The instinct is to rank four
models by score and ship the top one. The discipline is to notice the standard
error exceeds the spread and conclude that the ranking is noise. That single
observation changed the shipped artefact from a 3.9 MB random forest to a 3 KB
linear model with identical measured performance.

**Distinguishing a broken model from a broken threshold.** An F1 of 0.0 looks
like catastrophic failure. It was a correctly-calibrated model whose output range
never reached an arbitrary constant. Diagnosing that required looking at the
output distribution rather than at the metric.

**Refusing to use available data.** Twenty-three features were available, joinable
and plausible. Establishing that they were noise &mdash; and that the joins
themselves were meaningless because the files describe different universes &mdash;
required measurement that produces a negative result and no model improvement.

**Making failure legible.** A service that dies loudly is easy. A service that
reports `degraded` accurately, keeps `/health` serviceable when the model is
missing, isolates one bad row without failing a batch, and reports an empty
window as null rather than zero, is a series of deliberate decisions, each of
which has a wrong answer that looks fine in a demo.

</details>

<details>
<summary><b>Putting it in production</b></summary>

<br>

**Real use cases for a calibrated engagement score.** Because the output is a
rate rather than a rank, it can be spent against a budget:

- **Push notification targeting** &mdash; notify the top slice instead of everyone.
  At the recommended operating point the model reaches 23.9% of traffic and finds
  engaged users 1.40x more often than random, capturing a third of all engagement.
- **Candidate pre-filtering** &mdash; cheaply prune a large candidate pool before a
  heavier ranker runs, where a 3 KB dot product is effectively free.
- **Promoted-slot and ad-load decisions** &mdash; allocate paid inventory where
  predicted engagement justifies it.
- **Creator payout and boost eligibility** &mdash; any per-impression cost decision.

**The path to production**, in the order it has to happen:

1. **Fix training/serving skew first.** `watch_ratio` is derived from a different
   duration at serving time than at training time. This is the only defect that
   makes predictions *wrong* rather than merely unmonitored, and it is named first
   in the readiness review.
2. **Make the artefact trustworthy.** `joblib.load` executes what it deserialises.
   Artefacts need an immutable store, a content hash verified before load, and
   recorded provenance &mdash; training run, data snapshot, code revision, library
   versions.
3. **Split liveness from readiness**, so an orchestrator restarts a dead process
   and de-registers an unready one rather than confusing the two.
4. **Close the outcome loop.** Nothing currently compares predictions to the
   engagement that actually happened, so calibration drift &mdash; the failure that
   costs money &mdash; is invisible. A delayed-label job recomputing Brier and ECE
   is the single most valuable missing measurement.
5. **Ship behind a canary** with a registry, `model_version` on every prediction,
   and rollback as a pointer change rather than a redeploy.

**Scaling it.** Inference is not the bottleneck and never will be &mdash; the model
is a dot product over two features. The costs live elsewhere:

| Bottleneck | What it needs |
|---|---|
| **Feature retrieval** | The identifier tables are loaded into process memory from CSV at start-up. At real cardinality this becomes a feature store with caching, and staleness becomes a metric on `/health`. |
| **Batch throughput** | Synchronous HTTP capped at 100 rows suits interactive use. Millions of rows need an asynchronous job API writing to object storage, or an offline pipeline &mdash; with idempotency keys so retries do not double-score. |
| **Horizontal scale** | The services are stateless apart from the analytics log, which is an in-memory deque and therefore pins the deployment to one worker. Exporting metrics to a time-series database instead removes that ceiling. |
| **Load characteristics** | The recorded latencies come from sequential calls on a development machine. Real capacity planning needs a stated objective, an error budget, and a load test that finds the knee. |

</details>

<details>
<summary><b>What would make it better</b></summary>

<br>

**The ceiling here is set by the data, not by the modelling.** Four model families
converging within noise is itself the evidence: more capacity cannot extract
signal that is not present. Improvement has to come from better inputs.

**What is missing from this dataset**

- **Real user history.** No sequence of what a user watched before this
  impression. Sequential models are the single largest known gain in this domain
  and cannot be attempted here.
- **Content representation.** No video embeddings, audio, thumbnails or captions
  &mdash; only categorical metadata that turned out to be uncorrelated with the log.
- **Context.** No session position, no scroll depth, no device or network
  conditions.
- **Coherent joins.** The user and video tables share identifiers with the
  interaction log but not attribute values, so they cannot be used at all.

**Public datasets that carry what this one lacks**

| Dataset | Why it fits |
|---|---|
| [KuaiRec](https://kuairec.com/) | Kuaishou short-video logs carrying play duration and **watch ratio** &mdash; the two features that actually work here &mdash; over 12.5M interactions, with a small matrix at 99.6% density. A fully-observed matrix removes the missing-not-at-random problem that quietly distorts offline evaluation. |
| [KuaiRand](https://kuairand.com/) | Sequential Kuaishou data whose 12 feedback signals include likes, follows, comments and forwards &mdash; the exact actions this project's label is built from &mdash; plus millions of *randomly exposed* items, giving an unbiased slice instead of only logged-policy traffic. |
| [Tenrec](https://tenrec0.github.io/) | Large multipurpose benchmark from Tencent feeds, built for transfer, cold-start and multi-task setups across several scenarios. |
| [Taobao UserBehavior](https://tianchi.aliyun.com/dataset/649?lang=en-us) | ~100M behaviour records with genuine user sequences, for sequential and session-based modelling at scale. |
| [RecSysDatasets](https://github.com/RUCAIBox/RecSysDatasets) | A maintained index of public recommendation datasets with conversion tooling, for choosing by property rather than by familiarity. |

**What I would do with better data.** Keep the discipline and change the inputs:
the same leakage blocklist, chronological split, tie test, calibration gate and
budgeted operating point &mdash; applied to sequence features and content
embeddings, evaluated against the same honest ceiling analysis. The pipeline is
the reusable artefact; the 0.58 is a property of this dataset.

</details>

---

Each repository documents what is known to be wrong or incomplete in its
**Limitations** section, and what it would need before carrying real traffic in
**`docs/PRODUCTION_READINESS.md`** &mdash; ordered by risk, with the cost of each
remedy.
