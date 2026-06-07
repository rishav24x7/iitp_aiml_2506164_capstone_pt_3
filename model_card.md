# Model Card — D2C 60-Day Churn Predictor

## What it's for
It scores each active customer on how likely they are to make **no purchase in the next 60 days**, so the
retention team can decide who to reach out to first. It's a decision-support tool — it suggests *who* to look
at, it doesn't take any action on its own. The people who'd read its output are product, marketing, and CRM.

## What it learned from
- The source is `rfm_modeling_snapshot.csv`, a per-customer feature table built as of **2025-09-30** and
  certified leakage-free.
- 25 features in all: RFM (recency, frequency, monetary), returns, support sentiment, web and app
  activity, discount behaviour, tenure, and basic profile fields. Nothing from after the snapshot and nothing
  derived from the label gets in — the notebook asserts this explicitly.
- I used the provided train/validation/test split (1728/336/336); churn runs at about
  47% of customers.

## How it's built
- **Baseline:** a class-balanced Logistic Regression (validation ROC-AUC 0.883) — simple and honest.
- **The model I'd ship:** XGBoost with 600 shallow (depth-3) trees at a 0.02 learning rate, plus subsampling
  and L2 regularisation to keep it from overfitting. All the preprocessing lives inside the saved pipeline, so
  serving applies the exact same transforms as training.
- **Decision threshold:** 0.349, read off the validation precision-recall curve and tuned
  toward recall on purpose (see below).

## How well it does (held-out test set)
| Metric | Value |
|---|---|
| ROC-AUC | 0.8712 |
| PR-AUC | 0.8351 |
| Precision | 0.7626 |
| Recall | 0.8988 |
| F1 | 0.8251 |
| Accuracy | 0.8095 |

Confusion matrix: TN=121, FP=47, FN=17, TP=151. XGBoost edges past the baseline on validation ROC-AUC.
I deliberately favoured recall: in retention, missing a churner (losing the customer outright) costs far more
than a false alarm, which is just one cheap, unnecessary outreach.

## What's driving the predictions
recency_days, last_visit_days_ago, category_diversity_180d, frequency_180d, monetary_180d, return_rate_180d
and negative_ticket_rate_90d do most of the work. In plain terms: how recently and how often someone engages,
how broadly they shop, and whether they've been unhappy. That lines up with the Part 1 EDA, which is
reassuring — the model is picking up the same story the data told by eye.

## Where it falls short
- It's trained on one snapshot in time, so it doesn't know about seasonality or longer trends.
- It misses churners whose real reasons live outside the data — competitor offers, a bad delivery, frustration
  we never recorded. The false negatives in `error_analysis.md` are exactly these.
- The probabilities are **relative risk scores**, not calibrated odds. Calibrate them before wiring the model
  into anything that does expected-value or rupee-budget maths automatically.
- The test set is small (n=336), so I wouldn't read too much into third-decimal differences.

## Things to be careful about
- **Don't let it become a self-fulfilling prophecy.** If we quietly ignore everyone it scores low, we'll
  abandon customers we could have kept. Everyone should still get a baseline level of care.
- **Watch fairness.** Age group and city tier are inputs, so I'd monitor that outreach and offers don't end up
  systematically favouring or penalising any group — and lean on behavioural signals over demographic ones.
- **Never use it to make service worse.** A high churn score is not a reason to deprioritise someone's support.

## Keeping an eye on it
Track drift on the top features (PSI), watch the spread of predicted probabilities, and measure real
precision/recall once the 60-day outcomes actually land. Alert if the input schema or null rates shift. The
full plan is in Part 4's `monitoring_plan.md`.

## When to retrain
Refresh the snapshot and retrain at least once a quarter — sooner if drift alerts fire or if something big
changes in pricing, the catalogue, or the loyalty programme.

## When NOT to use it
- For brand-new customers with almost no history — that's a cold-start problem, handle it with onboarding rules.
- As an automatic cancel/deny switch, or as the only basis for anything a customer actually sees.
- Outside the D2C personal-care setting it was trained on.
