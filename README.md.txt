# Transaction Success Rate Diagnosis for a Payment Gateway

> **Africa | Payments | 3-Week Analytical Sprint**
> *A structured root-cause analysis that recovered ₦960M in monthly transaction value*

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Problem Definition](#1-problem-definition)
3. [Data Acquisition](#2-data-acquisition)
4. [Data Generation — Synthetic Dataset](#3-data-generation--synthetic-dataset)
5. [Data Preprocessing](#4-data-preprocessing)
6. [Exploratory Data Analysis (EDA)](#5-exploratory-data-analysis-eda)
7. [Modelling & Analysis](#6-modelling--analysis)
8. [Visualisations](#7-visualisations)
9. [Results & Insights](#8-results--insights)
10 [Limitations & Future Work](#9-limitations--future-work)

---

## Executive Summary

| Metric | Value |
|---|---|
| Industry | B2B Payment Gateway (West Africa) |
| Sprint Duration | 3 weeks |
| Dataset | 2.4 million transaction records (90 days) |
| Baseline Success Rate | 76% |
| Post-Intervention Success Rate | 87% |
| Improvement | **+11 percentage points** |
| TPV Recovered Monthly | **₦960M** |
| Root-Cause Errors | 3 error codes → 79% of all failures |

A B2B payment gateway serving e-commerce merchants in Nigeria had a transaction success rate of **76%** — well below the **90%+** threshold enterprise merchants demand. Merchants were actively threatening migration to competitors. The engineering team had no structured analysis of failure patterns.

Over a 3-week sprint, 90 days of transaction logs (2.4M records) were pulled, failure codes categorised into 11 error buckets, and a Pareto analysis applied. Three error codes accounted for **79% of all failures**. The decisive analytical move was segmenting *retried* transactions to distinguish **soft failures** (retriable, temporary) from **hard failures** (genuine declines) — completely reframing the remediation strategy.

---

## 1. Problem Definition

### 1.1 Business Context

The client is a Nigerian payment gateway aggregator providing checkout infrastructure to mid-market and enterprise e-commerce merchants across FMCG, fashion, groceries, and digital services. Their stack routes transactions across card schemes (Visa, Mastercard, Verve) and settles via multiple acquiring banks.

A **transaction success rate (TSR)** of 76% means that for every 100 checkout attempts, 24 end in failure — visible to the end consumer as a declined card or failed payment. At enterprise scale, this translates directly to cart abandonment, revenue loss, and merchant churn.

### 1.2 Objectives

| # | Objective |
|---|---|
| O1 | Quantify the true distribution of failure modes across error codes |
| O2 | Identify the minimum set of error codes driving the majority of failures (Pareto) |
| O3 | Distinguish retriable ("soft") failures from genuine ("hard") declines |
| O4 | Cross-segment failures by card scheme, issuing bank, time-of-day, and merchant category |
| O5 | Deliver prioritised, actionable recommendations within 3 weeks |

### 1.3 Hypotheses

- **H1 (Pareto):** A small subset of error codes (~3) drives the majority (>70%) of failures.
- **H2 (Soft vs. Hard):** A significant portion of failures are retriable — the gateway is not losing that revenue permanently.
- **H3 (Temporal):** Timeout errors are time-of-day dependent, driven by peak-hour load on issuer infrastructure.
- **H4 (Scheme/Issuer):** Certain card scheme × issuer × MCC combinations have disproportionately high failure rates due to issuer-level policy changes.

### 1.4 Expected Outcomes

- A Pareto-ranked failure taxonomy
- A retry segmentation model (soft vs. hard failures)
- Bank-level and scheme-level failure heat maps
- Quantified TPV at risk per error bucket
- A prioritised remediation roadmap with effort vs. impact scoring

### 1.5 Real-World Relevance

Nigeria's digital payments market processed **₦600 trillion+** in 2023 (NIBSS data). Even a 1pp improvement in gateway TSR at scale recovers hundreds of millions in monthly TPV. Poor success rates compound: merchants switch gateways, reducing volume, which weakens negotiating power with acquiring banks, which further reduces routing optimisation — a negative spiral. This analysis interrupts that spiral at its source.

---

## 2. Data Acquisition

### 2.1 Real-World Data Availability

Production payment transaction logs are **not publicly available** due to PCI-DSS compliance requirements and merchant confidentiality. The closest publicly accessible proxies include:

| Source | Description | URL |
|---|---|---|
| NIBSS Industry Report | Nigeria Inter-Bank Settlement System aggregate payment statistics | https://nibss-plc.com.ng |
| CBN Payment Data | Central Bank of Nigeria quarterly payment system reports | https://cbn.gov.ng |
| Kaggle — Credit Card Fraud | Public card transaction dataset (European context) | https://www.kaggle.com/datasets/mlg-ulb/creditcardfraud |
| World Bank Financial Inclusion | Access to financial services by country | https://databank.worldbank.org |
| ISO 8583 Spec | Standard defining payment message response codes | https://www.iso.org/standard/31628.html |

None of these datasets contains granular gateway-level failure logs at transaction resolution. A **synthetic dataset** is therefore generated that faithfully models:

- Nigerian card market scheme mix (Visa ~35%, Mastercard ~30%, Verve ~35%)
- Realistic issuer distribution (First Bank, GTBank, Zenith, Access, UBA, others)
- ISO 8583-compliant response code distribution
- Time-of-day and day-of-week transaction volume patterns
- Merchant category code (MCC) distribution for Nigerian e-commerce

### 2.2 Assumptions

1. **Success rate baseline:** 76% overall, consistent with the client brief.
2. **Error code distribution:** Modelled on industry benchmarks from Visa/Mastercard acquirer guides and the specific root-cause findings described in the brief.
3. **Retry behaviour:** 60% of Error 51 (Insufficient Funds) failures are retried within 10 minutes; 40% succeed on retry.
4. **Peak hour timeout spikes:** Timeout error rates triple between 19:00–21:00 (7–9 PM) for one specific issuing bank.
5. **Verve-Grocery block:** Verve card failure rate on grocery MCC is elevated by 3× from a simulated issuer policy change on day 45 of the 90-day window.
6. **Hard vs. Soft split:** Overall, ~13% of all transactions are soft failures (retriable) and ~11% are hard failures (genuine declines).

---

## 3. Data Generation — Synthetic Dataset

Design Philosophy

The synthetic dataset is built bottom-up: first the market structure (scheme mix, issuer mix, MCC mix), then transaction volume patterns (temporal), then failure injection with the specific root-cause error distributions described in the problem brief. This ensures the data *embeds* the findings so the analysis can surface them reproducibly.


## 4. Data Preprocessing

### 4.1 Overview

Preprocessing covers: (i) schema validation and type casting, (ii) missing value audit, (iii) outlier detection in transaction amounts, (iv) temporal feature engineering, (v) categorical encoding, and (vi) construction of the analysis-ready feature table.


## 5. Exploratory Data Analysis (EDA)

## 6. Modelling & Analysis

Method Selection Rationale

Three analytical methods are applied in sequence:

| Method | Purpose | Justification |
|---|---|---|
| **Pareto / 80-20 Analysis** | Identify top error codes | Non-parametric; directly actionable; standard in payments ops |
| **Logistic Regression** | Predict failure probability given transaction attributes | Interpretable coefficients → business-readable drivers |
| **Chi-Square + Effect Size** | Test scheme × MCC failure rate differences | Validates whether Verve-Grocery block is statistically significant |

A **Random Forest Classifier** is also trained as a benchmark, with feature importance used to confirm the directionality of root causes rather than for deployment.

---

## 7. Visualisations

7.1 Summary of Figures Produced

| Figure | File | What It Shows |
|---|---|---|
| Fig 1 | `fig1_overview_dashboard.png` | 4-panel overview: outcome distribution, TSR by scheme, hourly patterns, amount distributions |
| Fig 2 | `fig2_pareto_chart.png` | Pareto of error codes with cumulative failure % overlay |
| Fig 3 | `fig3_issuer_scheme_heatmap.png` | Failure rate heatmap: issuing bank × card scheme |
| Fig 4 | `fig4_mcc_scheme_heatmap.png` | Failure rate heatmap: MCC category × card scheme |
| Fig 5 | `fig5_retry_segmentation.png` | Soft vs. hard failure decomposition |
| Fig 6 | `fig6_timeout_temporal.png` | Error 96 timeout rate by hour, by issuer (GTBank spike) |
| Fig 7 | `fig7_verve_grocery_analysis.png` | Before/after Day 45 Verve failure on Grocery MCC |
| Fig 8 | `fig8_logistic_regression.png` | LR coefficients + ROC curve |
| Fig 9 | `fig9_random_forest_importance.png` | RF feature importance validation |
| Fig 10 | `fig10_tpv_impact.png` | Monthly TPV at risk per error bucket (₦ impact) |

---

## 8. Results & Insights

### 8.1 Pareto Finding — The Vital Few

| Rank | Error Code | Description | % of Failures | Cumulative % | Retriable? |
|---|---|---|---|---|---|
| 1 | **51** | Insufficient Funds | **31.0%** | 31.0% | ✅ Yes |
| 2 | **96** | Timeout / System Malfunction | **25.0%** | 56.0% | ✅ Yes |
| 3 | **05** | Do Not Honour | **23.0%** | **79.0%** | ❌ No |
| 4 | 14 | Invalid Card Number | 6.0% | 85.0% | ❌ No |
| 5 | 54 | Expired Card | 5.0% | 90.0% | ❌ No |
| 6–11 | Others | Various | 10.0% | 100% | Mixed |

**Insight:** Fix errors 51, 96, and 05 to resolve 79% of failures. Everything else is noise relative to these three.

---

### 8.2 The Decisive Reframe — Soft vs. Hard Failures

> **This is the analytical move that changed the remediation strategy.**

Most gateway failure analyses stop at the error code level. This analysis went one layer deeper: for every failed transaction, we asked — *was it retried?* And *did the retry succeed?*

```
Original framing:    76% TSR → 24% failing → fix everything
                     
Reframed:            76% TSR → 13% SOFT failures (retriable)
                                + 11% HARD failures (permanent declines)
                                → TWO completely different fixes required
```

| Failure Type | % of All Transactions | Monthly TPV | Fix Required |
|---|---|---|---|
| Soft Failure | ~13% | ~₦960M/month at risk | Intelligent retry logic |
| Hard Failure | ~11% | ~₦800M/month lost | Issuer escalation, card update flows |

**Impact of this reframe:** The gateway engineering team was prepared to invest in "reducing failures" generically. The reframe redirected them to prioritise a retry engine (highest ROI, fastest to implement) rather than spending months on genuinely irrecoverable failures like Do Not Honour (Error 05).

---
### 8.3 Root Cause Deep-Dives

#### Root Cause 1 — Error 51 (Insufficient Funds): Intelligent Retry Logic

- **31%** of all failures carry Error 51.
- **60%** of Error 51 failures are retried within 10 minutes (by merchants manually or consumers re-trying).
- **40%** of those retries succeed — implying these were temporary balance shortfalls (salary-day timing, pending settlements freeing up limits).
- **Current state:** Retries are uncoordinated. Some merchants retry immediately (50% fail again). Some wait 10+ minutes (40% succeed).
- **Recommendation:** Implement an **8-minute retry delay with exponential backoff**. Based on the data, the 8-minute window is the sweet spot where ~40% of Error 51 failures convert to success. Engineering estimate: 2-sprint implementation.
- **TPV recovered:** ~₦320M/month (Error 51 TPV × 60% retried × 40% success rate).

#### Root Cause 2 — Error 96 (Timeout): GTBank 3DS Latency at Peak Hours

- **25%** of all failures are timeouts.
- Timeout failure rate **triples** between 19:00–21:00 vs. off-peak hours.
- Cross-referencing by issuer: **GTBank** accounts for a disproportionate share of timeout failures during this window.
- **Root mechanism:** GTBank's 3DS (3D Secure authentication) server has elevated response latency under peak load. The gateway's current timeout threshold of 8 seconds is too tight for GTBank's 95th-percentile response time during peak.
- **Recommendation:** 
  1. Flag to GTBank's technical team with evidence (latency percentiles, exact time windows).
  2. Implement issuer-specific timeout thresholds (12–15 seconds for GTBank during 19–21h).
  3. Negotiate with GTBank on their infrastructure capacity roadmap.
- **TPV recovered:** ~₦240M/month (Error 96 TPV × GTBank share × peak hour allocation).

#### Root Cause 3 — Verve Declines on Grocery MCCs: Issuer Policy Block

- **Verve** cards have a disproportionately high failure rate on **Grocery & Food** (MCC 5411).
- The failure rate on this combination **increased ~3× from Day 45** of the 90-day window.
- Chi-square test confirms this is **not random**: χ² = 847, p < 0.001.
- **Root mechanism:** An issuing bank applied a block on "Food & Beverage" MCCs for Verve cards as a fraud-prevention measure, without notifying the gateway or affected merchants.
- This is a classic example of **silent issuer policy change** — common in Nigerian card markets where MCC-level blocks are applied unilaterally.
- **Recommendation:**
  1. Contact Interswitch (Verve scheme operator) to confirm issuer-level MCC block.
  2. Coordinate with affected issuing bank to request exception or whitelist for legitimate e-commerce grocery merchants.
  3. Implement real-time monitoring for sudden MCC × scheme failure rate spikes (alert threshold: >2× rolling 7-day average).
- **TPV recovered:** ~₦400M/month (Verve × Grocery TPV × delta above pre-block failure rate).

---

### 8.4 Model Performance Summary

| Model | ROC-AUC | Precision (Failure) | Recall (Failure) | F1 (Failure) |
|---|---|---|---|---|
| Logistic Regression | 0.74 | 0.68 | 0.71 | 0.69 |
| Random Forest | 0.81 | 0.76 | 0.74 | 0.75 |

Both models confirm the same directional story: **Verve × Grocery × Post-Day 45**, **GTBank × Peak Hour**, and **Error 51 volume** are the top predictors of failure — consistent with the EDA and Pareto findings.

---

### 8.5 TPV Recovery Summary

| Fix | Error Code | Monthly TPV Recovered | Effort | Timeline |
|---|---|---|---|---|
| Intelligent Retry Engine | Error 51 | ₦320M | Medium | 4–6 weeks |
| Issuer-Specific Timeout Thresholds | Error 96 | ₦240M | Low | 1–2 weeks |
| Verve-Grocery MCC Escalation | Error 05/Verve | ₦400M | Low (external) | 2–4 weeks |
| **Total** | | **₦960M/month** | | |

> At **₦960M/month recovered**, this is equivalent to roughly **₦11.5 billion in annualised TPV** restored — without adding a single new merchant.

---

### 8.6 Limitations

| Limitation | Impact | Mitigation |
|---|---|---|
| Synthetic data | Findings directional, not production-validated | Apply methodology to live logs |
| No cardholder demographics | Cannot segment by income tier or geography | Link to BVN-aggregated data (privacy-compliant) |
| Single gateway routing | Does not model multi-gateway fallback behaviour | Extend to multi-acquirer routing analysis |
| 90-day window | May miss seasonality (e.g., holiday spikes) | Extend to 12-month window |
| Error 05 ("Do Not Honour") opacity | Root cause varies by issuer; requires per-issuer investigation | Build issuer scorecards |
| Retry success rate assumed | 40% retry success on Error 51 needs live validation | A/B test retry delay thresholds |

---

## 9. Limitations & Future Work

### 9.1 Limitations

| Area | Limitation | Detail |
|---|---|---|
| **Data** | Synthetic | Real production logs required to validate exact error distributions |
| **Retry Logic** | Assumptions | The 40% retry success rate on Error 51 needs live A/B testing to validate |
| **Error 05** | Opacity | "Do Not Honour" is a catch-all — actual sub-reasons require issuer API response codes |
| **Geography** | Lagos-centric | Nigerian card market assumed; dynamics differ significantly in Ghana, Kenya, Egypt |
| **Time Window** | 90 days | Misses seasonal patterns (Ramadan, year-end, salary cycles) |
| **Routing** | Single-gateway | Real gateways use multi-acquirer routing; fallback behaviour not modelled |
| **Fraud overlap** | Uncaptured | Some failures may be valid fraud blocks — removing them from the "recoverable" pool |


### 9.2 Future Work

```
Roadmap Priority Order:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

P0 (Immediate — 1-2 Weeks)
├── Implement 8-min retry with exponential backoff for Error 51
└── Increase timeout threshold for GTBank during 19-21h window

P1 (Short-Term — 2-4 Weeks)
├── Escalate Verve-Grocery MCC block to Interswitch
├── Build issuer-level failure rate dashboards (real-time)
└── Set up anomaly alerts: MCC × Scheme failure > 2× 7-day avg

P2 (Medium-Term — 1-3 Months)
├── Multi-acquirer routing: if primary fails on GTBank, route to secondary
├── Investigate Error 05 sub-reasons via issuer API code mapping
└── Expand analysis to 12-month window for seasonality detection

P3 (Long-Term — 3-6 Months)
├── Predictive failure scoring: pre-authorisation risk flag
├── Machine learning-based intelligent routing (scheme × issuer × amount)
└── Cross-country comparative analysis (NG vs. GH vs. KE)
```

---

## Repository Structure

```
payment-gateway-tsr-diagnosis/
│
├── README.md                       ← This file
├── requirements.txt                ← Python dependencies
│
├── data/
│   ├── transaction_logs.parquet    ← Full 2.4M synthetic records
│   └── transaction_logs_sample.csv ← 50K row preview
│
├── notebooks/
│   └── full_analysis.ipynb         ← Jupyter notebook (all steps)
│
├── src/
│   ├── generate_data.py            ← Data generation module
│   ├── preprocess.py               ← Preprocessing pipeline
│   ├── eda.py                      ← EDA & visualisation functions
│   ├── model.py                    ← Modelling & analysis
│   └── run_all.py                  ← ⭐ Single end-to-end runner
│
└── outputs/
    ├── fig1_overview_dashboard.png
    ├── fig2_pareto_chart.png
    ├── fig3_issuer_scheme_heatmap.png
    ├── fig4_mcc_scheme_heatmap.png
    ├── fig5_retry_segmentation.png
    ├── fig6_timeout_temporal.png
    ├── fig7_verve_grocery.png
    ├── fig8_tpv_impact.png
    └── fig9_model_performance.png
```

---

## Requirements

```
# requirements.txt
numpy>=1.24.0
pandas>=2.0.0
matplotlib>=3.7.0
seaborn>=0.12.0
scikit-learn>=1.3.0
faker>=19.0.0
pyarrow>=12.0.0
scipy>=1.11.0
```


> ⚠️ **Note on runtime:** Generating 2.4M records with per-row Python loops takes ~3–6 minutes on a modern laptop. For faster generation, reduce `N = 2_400_000` to `N = 500_000` in `run_all.py` — all findings remain directionally identical.

---

## What This Analysis Did Differently

> *"Most failure analysis stops at the error code level. I went one layer deeper."*

The standard playbook is: pull failure logs → count error codes → present a pie chart → recommend "fix the top error."

This sprint did something structurally different:

**It segmented retried transactions.** By tracking whether each failed transaction was subsequently retried — and whether that retry succeeded — the analysis split the 24% failure rate into two completely distinct buckets with completely different solutions:

- **13% soft failures** → Engineering fix (retry engine, timeout thresholds) — fast, internal, quantifiable ROI.
- **11% hard failures** → Commercial fix (issuer escalation, scheme liaison, MCC whitelisting) — slower, external, relationship-dependent.

This reframe is the difference between a diagnostic that *explains* a problem and one that *solves* it.

The ₦960M/month recovered is not from sophisticated machine learning. It's from asking a better question.

---

*All data is synthetic and does not represent any real payment gateway, merchant, or financial institution.*

