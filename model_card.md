# Model Card — D2C 60-Day Churn Predictor

## Intended use
Rank active customers by their probability of **not purchasing in the next 60 days** so the retention team can
prioritise outreach. Decision-support only — it informs *who* to contact, not an automated action. Audience:
product, marketing, and CRM leadership.

## Data
- Source: `rfm_modeling_snapshot.csv`, a leakage-safe per-customer feature table as of **2025-09-30**.
- 25 features spanning RFM (recency/frequency/monetary), returns, support sentiment, web/app
  activity, discounts, tenure, and profile attributes. No post-snapshot or target-derived columns are used
  (verified by an explicit leakage assertion in the notebook).
- Provided train/validation/test split (1728/336/336); churn base rate ~47%.

## Model approach
- **Baseline:** class-balanced Logistic Regression (validation ROC-AUC 0.883).
- **Production model:** XGBoost, 600 shallow (depth-3) trees, lr 0.02, with subsampling and L2 regularisation
  to limit overfitting. Preprocessing (impute + scale + one-hot) is bundled in the saved pipeline.
- **Decision threshold:** 0.330, chosen from the validation PR curve, recall-leaning.

## Performance (held-out test set)
| Metric | Value |
|---|---|
| ROC-AUC | 0.8755 |
| PR-AUC | 0.8392 |
| Precision | 0.7463 |
| Recall | 0.9107 |
| F1 | 0.8204 |
| Accuracy | 0.8006 |
Confusion matrix: TN=116, FP=52, FN=15, TP=153. XGBoost beats the baseline on validation ROC-AUC.

## Top drivers
recency_days, last_visit_days_ago, category_diversity_180d, frequency_180d, monetary_180d, return_rate_180d,
negative_ticket_rate_90d. Disengagement and narrow purchase breadth dominate; dissatisfaction signals add lift.

## Limitations
- Trained on a single 2025-09-30 snapshot; seasonality and trend shifts are not captured.
- Misses churners whose drivers are outside the data (competitor offers, delivery experience, unlogged
  dissatisfaction) — see `error_analysis.md` false negatives.
- Probabilities are **relative risk scores**, not calibrated long-run frequencies; calibrate before any
  expected-value/—-budget automation.
- Small test set (n=336); treat third-decimal metric differences as noise.

## Ethical risks
- **Self-fulfilling neglect:** if low scorers are systematically ignored, the brand may abandon recoverable
  customers. Keep a low-cost baseline touch for everyone.
- **Fairness:** demographic fields (age_group, city_tier) are inputs — monitor that outreach/offers do not
  systematically disadvantage a protected group. Prefer behaviour-driven actions.
- **Do not** use churn scores to degrade service (e.g., deprioritise support for "likely-to-leave" customers).

## Monitoring
- Feature drift (PSI) on top drivers; prediction-distribution drift; live precision/recall once outcomes land;
  alert on input-schema or null-rate changes. (Full plan in Part 4 `monitoring_plan.md`.)

## Retraining
- Refresh the snapshot and retrain at least quarterly, or when drift alerts fire, or after major changes to
  pricing, catalogue, or the loyalty programme.

## When NOT to use
- For brand-new customers with almost no history (cold start — use onboarding heuristics instead).
- As an automated cancel/deny mechanism, or as the sole basis for any customer-facing decision.
- Outside the D2C personal-care context it was trained on.
