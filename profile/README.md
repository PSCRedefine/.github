<div align="center">

<img src="https://raw.githubusercontent.com/PSCRedefine/.github/main/assets/banner.svg" alt="PSC Redefine — calibrated engagement prediction for short-form video" width="100%"/>

<br/>

![Python](https://img.shields.io/badge/Python-3.11%E2%80%933.13-3776AB?logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-two--tower-EE4C2C?logo=pytorch&logoColor=white)
![LightGBM](https://img.shields.io/badge/LightGBM-ranking-9ACD32)
![FastAPI](https://img.shields.io/badge/FastAPI-serving-009688?logo=fastapi&logoColor=white)
![DuckDB](https://img.shields.io/badge/DuckDB-322M%20rows-FFF000?logo=duckdb&logoColor=black)
![Tests](https://img.shields.io/badge/pytest-292%20tests%20%C2%B7%20CI-6C9)
[![Hiring](https://img.shields.io/badge/%F0%9F%9A%80_We're_hiring-Founding_Engineer-8b7bff)](https://github.com/PSCRedefine/.github/blob/main/careers/founding-engineer.md)

</div>

## What we're building

Short-form feeds are ranked by models nobody can audit, validated offline on logs shaped by the very policy being replaced. **PSC Redefine** builds engagement prediction you can reason about: calibrated probabilities instead of opaque scores, evaluation that separates real signal from exposure bias, and serving with monitoring built in — not bolted on.

Our work runs on real production logs at scale: **322M interactions, ROC-AUC 0.9147**, every figure reproduced from a committed artifact.

## Research: [RankShift / GroundTruth](https://github.com/PSCRedefine/rankshift)

KuaiRand's uniformly-random exposure window gives a genuinely missing-at-random test set — a rare gold standard. Using it, we isolate what survives a policy change and what silently breaks:

- **Ranking transfers; calibration does not.** Top-of-traffic lift holds across exposure policies (3.43× → 3.36×), but predicted probabilities run **2.07× too high** on randomized traffic — a model that looks fine offline ships probabilities wrong by a factor of two.
- **Off-policy estimators graded against ground truth.** IPS, SNIPS, DM and DR all miss the uniform policy's true value (3.0–3.9× off) — item-level propensities can't express the user–video matching that drives most of the exposure bias.
- **Findings replicate at 322M rows**, evaluated in full, out of core (DuckDB), where the calibration break grows to 2.29×.

## The serving stack: Cognitive Shorts

| Service | What it does |
|---|---|
| **[SinglePrediction](https://github.com/PSCRedefine/SinglePrediction)** | Train, select, and serve — real-time scoring API, ~1 ms per prediction |
| **[BatchPrediction](https://github.com/PSCRedefine/BatchPrediction)** | Bulk scoring with per-row fault isolation |
| **[ModelInfo](https://github.com/PSCRedefine/ModelInfo)** | Model artifact introspection API + health dashboard |
| **[AnalyticsDashboard](https://github.com/PSCRedefine/AnalyticsDashboard)** | Request logging, drift alerts, live monitoring |

FastAPI + Streamlit per service, Docker Compose, leakage blacklists enforced by tests, chronological splits, paired-bootstrap model selection, operating points set against a traffic budget. 292 tests, CI on Python 3.11–3.13, MIT — and each repo states its known gaps in `docs/PRODUCTION_READINESS.md`.

## How we work

- **Calibration is a feature.** A probability should mean what it says — we measure calibration error and publish it.
- **Evidence prunes everything.** Features, models, even our own earlier conclusions: the same screening rule that kept 2 of 25 features on synthetic data keeps 24 of 37 on real logs, and we documented why.
- **Reproducible by default.** Every reported figure regenerates from a committed JSON artifact and a fetch script.
- **Monitoring ships with the model.** Error-rate, latency, and output-distribution alerts live beside serving, not after it.

## 🚀 We're hiring

Founder-led, early-stage, and looking for our first hire: a **Founding Engineer** to own major parts of the platform end to end.

**→ [Read the role & apply](https://github.com/PSCRedefine/.github/blob/main/careers/founding-engineer.md)**

## Founder

**Mengyun Wang** — ML/AI infrastructure engineer; previously training-data infrastructure at Meta and GPU/training software at Oracle. [GitHub](https://github.com/xiyiji) · [Site](https://xiyiji.github.io) · mengyun_wang_ai@outlook.com

<div align="center">
<sub>PSC Redefine · building feeds you can reason about</sub>
</div>
