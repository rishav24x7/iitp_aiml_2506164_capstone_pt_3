# Part 3 — Churn Prediction Model & Model Card

**D2C Customer Churn Intelligence & Retention** · Capstone Part 3 of 4

This repository builds a model that predicts whether a customer will **churn in the next 60 days**
(`churn_next_60d` = no purchase in `2025-10-01` → `2025-11-29`). It trains a baseline and a stronger model,
evaluates them with churn-appropriate metrics, selects a business-justified threshold, analyses errors on real
customers, and ships a model card.

## Repository structure

```
iitp_aiml_2506164_capstone_pt_3/
├── churn_model.ipynb       # Main notebook (run top-to-bottom, outputs included)
├── model.pkl               # Generated: fitted pipeline + threshold + feature list
├── metrics.json            # Generated: validation/test metrics, threshold, confusion matrix
├── error_analysis.md       # Generated: FP/FN analysis with 12 real customer examples
├── model_card.md           # Generated: intended use, limits, ethics, monitoring
├── evaluation_curves.png   # Generated: ROC, PR, confusion matrix
├── feature_importance.png
├── data/                   # The 7 source CSVs + DATA_DICTIONARY.md
├── requirements.txt
└── README.md
```

## Setup & run

```bash
python -m venv .venv && source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace churn_model.ipynb
```

Reads from the relative `data/` folder. Running it regenerates `model.pkl`, `metrics.json`, both markdown
reports, and the charts. The saved `model.pkl` is consumed directly by the Part 4 scoring API.

## Approach

1. **Leakage-safe features** from `rfm_modeling_snapshot.csv`, with an **explicit assertion** that the target,
   `split`, and `snapshot_date` never enter the feature set.
2. **Provided train/validation/test split** (1728 / 336 / 336) for reproducible evaluation.
3. **Preprocessing pipeline** — median-impute + scale numerics, constant-impute + one-hot encode categoricals
   (null `loyalty_tier` → an informative `"Missing"` level). Bundled so serving uses the identical transform.
4. **Baseline:** class-balanced Logistic Regression. **Stronger:** tuned XGBoost (600 depth-3 trees, lr 0.02, L2).
5. **Threshold** chosen from the validation PR curve, **recall-leaning** (a missed churner costs more than a
   cheap wasted touch).

## Results (held-out test set)

| Metric | XGBoost |
|---|---|
| ROC-AUC | 0.871 |
| PR-AUC | 0.835 |
| Precision | 0.763 |
| Recall | 0.899 |
| F1 | 0.825 |
| Accuracy | 0.810 |

Confusion matrix @ threshold 0.349: TN=121, FP=47, FN=17, TP=151. **XGBoost beats the baseline on validation
ROC-AUC (0.8855 vs 0.8826).** Top drivers: `recency_days`, `last_visit_days_ago`, `category_diversity_180d`,
`frequency_180d`, `monetary_180d` — disengagement dominates, matching the Part 1 hypotheses.

See `model_card.md` for intended use, limitations, ethics, and monitoring, and `error_analysis.md` for the
false-positive / false-negative breakdown with real customer IDs.
