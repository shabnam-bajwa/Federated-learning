# Verifying Data Locality in Federated Health Analytics over Solid Pods

A decentralised, privacy-aware machine-learning pipeline for **meal-centric glucose prediction**, in which raw health data never leaves each participant's [Solid](https://solidproject.org/) Pod. The system trains a shared model with **federated averaging (FedAvg)**, keeps feature engineering, inference and recommendation **inside each Pod**, and — crucially — **verifies** the data-locality property with an access-control verification (ACV) procedure rather than assuming it.

> MSc Artificial Intelligence dissertation · Birmingham City University
> Author: **Dev Rabari** · Supervisor: **Dr Mohamed Ragab**

---

## What this project does

Conventional health analytics pools raw records into a central store, which removes user control and concentrates risk. This project takes the opposite approach: **bring the model to the data.** Each participant has a Solid Pod; a Pod-authorised local agent builds features, trains locally, predicts, and writes recommendations back to the same Pod. Only model parameters cross the boundary.

Three properties are implemented genuinely, not simulated:

1. **In-Pod feature engineering** — each agent reads its participant's raw files *from that participant's Pod* and builds features locally.
2. **Recommendations written back to the Pod** — guidance is generated in-Pod and written to the owner's Pod output directory.
3. **Aggregate-only federated standardisation** — each client sends only `{count, sum, sum-of-squares}` per feature; the aggregator computes a global mean/std without ever seeing raw values.

The privacy boundary is enforced at the **application level** in this implementation (an ownership check plus an audit of every transmitted update). A production system would enforce it with native Solid Web Access Control (WAC) and WebID — noted as future work.

---

## Key results

Evaluated on the AI4FoodDB cohort (96 participants → 95 eligible → 90 contributing trained updates), meal-centric `glucose_60m` prediction on a shared held-out set:

| Model | R² | MAE | RMSE |
|---|---|---|---|
| Federated (multi-round FedAvg, 30 rounds) | **0.191** | 15.62 | 20.51 |
| Federated (single-round, reference) | 0.118 | 16.44 | 21.40 |
| Centralised (pooled, same features) | 0.205 | 15.50 | 20.33 |
| Centralised (XGBoost, full features)\* | 0.58 | 6.42 | 12.30 |

Baselines (RQ3): mean-prediction ≈ **0.000**, local-only (no collaboration) ≈ **−0.913** — federation clearly beats non-collaborative learning. Multi-seed (5 seeds): federated **0.166** (SD 0.006), centralised **0.179** (SD 0.009), gap **0.013**.

**Access-control verification:** own-Pod reads allowed; cross-Pod and aggregator reads denied; no raw records in any of the 90 audited update files — **93/93 checks passed.**

**In-Pod recommendations:** across 8,468 meals — 92.2% routine, 7.8% moderate, 0% high; 85 of 95 clients received at least one flag.

\*The XGBoost row is a **centralised, non-private reference only.** It uses a richer time-series feature set and a temporal split, so it is an *indicative* best-achievable benchmark, **not** a like-for-like comparison with the federated model.

---

## Repository structure

```
.
├── decentralised_pipeline.ipynb          # MAIN SYSTEM (reported in the dissertation)
│                                          # Pod-local FedAvg: in-Pod features, agents read/write
│                                          # their own Pod, aggregate-only standardisation, ACV,
│                                          # in-Pod recommendations, multi-seed, figures.
│
├── 05_Regression_Models_TIME_SERIES.ipynb # Centralised (non-private) REFERENCE models.
│                                          # Source of the XGBoost R² 0.58 benchmark.
│                                          # NOT part of the decentralised system.
│
├── <feature_engineering>.ipynb            # Builds the meal-centric dataset from raw data.
│
├── figures/                               # Generated result figures (PNG).
├── reports/                               # Generated CSV result tables.
│
└── archive/                               # Earlier / exploratory notebooks — NOT part of the
                                           # final evaluation (kept for project history only).
```

> Rename `<feature_engineering>` to your actual filename before publishing.

---

## Data availability

**The AI4FoodDB dataset is not included in this repository.** It is licensed third-party research data containing sensitive health records, and must not be redistributed. Access it from the original providers. The `logical_pods/` and any raw `data/` folders are excluded via `.gitignore` and should never be committed.

---

## Requirements

- Python 3.10+
- `numpy`, `pandas`, `scikit-learn`, `matplotlib`
- `xgboost` (only for the centralised reference notebook)
- [Community Solid Server](https://github.com/CommunitySolidServer/CommunitySolidServer) (for the physical-Pod access-control demonstration)

```bash
pip install numpy pandas scikit-learn matplotlib xgboost
```

---

## How to run

1. Obtain the AI4FoodDB data and place it where the config cell expects it.
2. Open `decentralised_pipeline.ipynb`, set the two paths in the **Config** cell to your machine, and **Run All** from the top.
   - It provisions a logical Pod per eligible client, runs federated training, performs the access-control verification, generates in-Pod recommendations, and saves the figures.
   - The final federated R² should print **0.1909**.
3. For the centralised XGBoost reference, run `05_Regression_Models_TIME_SERIES.ipynb`.

---

## Limitations

- Linear federated model — chosen so parameters can be averaged and audited; bounds accuracy.
- Small, single cohort with sparse per-participant data (per-client R² varies widely; median ≈ −0.13).
- Access control is demonstrated at the application level, not enforced by native Solid WAC/WebID.
- No secure aggregation or differential privacy — parameter sharing still carries residual inference risk.

## Future work

Non-linear yet averageable models; native Solid WAC/WebID enforcement; secure aggregation and differential privacy; larger and more diverse cohorts; extension to other conditions.

---

## Acknowledgements

Supervised by Dr Mohamed Ragab, Birmingham City University. Uses the AI4FoodDB research database and the Solid / Community Solid Server ecosystem.
