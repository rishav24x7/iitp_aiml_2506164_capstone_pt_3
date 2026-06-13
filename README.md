# Part 3 — Churn Prediction Model & Model Card

**D2C Customer Churn Intelligence & Retention** · Capstone Part 3 of 4

This is where I actually build the model that predicts whether a customer will **churn in the next 60 days**
(`churn_next_60d` = no purchase between `2025-10-01` and `2025-11-29`). I train a simple baseline and a
stronger model, judge them on metrics that actually suit churn (not just accuracy), pick a threshold that
makes business sense, dig into the model's mistakes on real customers, and write up a model card.

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
| ROC-AUC | 0.8755 |
| PR-AUC | 0.8392 |
| Precision | 0.7463 |
| Recall | 0.9107 |
| F1 | 0.8204 |
| Accuracy | 0.8006 |

At the 0.3304 threshold the confusion matrix is TN=116, FP=52, FN=15, TP=153. **XGBoost comes out ahead of the
baseline on validation ROC-AUC (0.8832 vs 0.8826).** The features doing the heavy lifting are `recency_days`,
`last_visit_days_ago`, `category_diversity_180d`, `frequency_180d` and `monetary_180d` — so disengagement
dominates here too, which is exactly what the Part 1 EDA predicted.

For the full picture, `model_card.md` covers intended use, limitations, ethics and monitoring, and
`error_analysis.md` walks through the false positives and negatives with real customer IDs.
