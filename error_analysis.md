# Error Analysis (Part 3)

Model: XGBoost at a 0.349 threshold, scored on the held-out test set (n=336). The confusion matrix was TN=121, FP=47, FN=17, TP=151. Below I look at the mistakes it made most confidently, because the two error types cost the business very different amounts.

## The two ways to be wrong

- **False positives (47):** we flag someone as churn-risk and they stay. The cost is a single retention touch we didn't strictly need — and we deliberately keep those cheap, so this is an acceptable error to make.
- **False negatives (17):** we miss a real churner and do nothing. That's a customer lost with no attempt to save them — the expensive mistake. Keeping this number low is exactly why I leaned the threshold toward recall.

## False positives — we said churn, they stayed (6 most confident)

| customer_id | churn_proba | recency_days | last_visit_days_ago | frequency_180d | monetary_180d | what happened |
|---|---|---|---|---|---|---|
| CUST01017 | 0.94 | 133 | 13 | 2 | 1,167 | Long quiet stretch since the last order and visit, so the model reasonably bet on churn — but they came back. These are lapsing-but-loyal customers; a nudge here is cheap and mostly harmless. |
| CUST00437 | 0.94 | 151 | 33 | 1 | 729 | Long quiet stretch since the last order and visit, so the model reasonably bet on churn — but they came back. These are lapsing-but-loyal customers; a nudge here is cheap and mostly harmless. |
| CUST01325 | 0.93 | 186 | 43 | 0 | 0 | Long quiet stretch since the last order and visit, so the model reasonably bet on churn — but they came back. These are lapsing-but-loyal customers; a nudge here is cheap and mostly harmless. |
| CUST01370 | 0.93 | 161 | 35 | 2 | 1,246 | Long quiet stretch since the last order and visit, so the model reasonably bet on churn — but they came back. These are lapsing-but-loyal customers; a nudge here is cheap and mostly harmless. |
| CUST01405 | 0.92 | 140 | 20 | 1 | 1,013 | Long quiet stretch since the last order and visit, so the model reasonably bet on churn — but they came back. These are lapsing-but-loyal customers; a nudge here is cheap and mostly harmless. |
| CUST01614 | 0.91 | 103 | 4 | 2 | 1,352 | Long quiet stretch since the last order and visit, so the model reasonably bet on churn — but they came back. These are lapsing-but-loyal customers; a nudge here is cheap and mostly harmless. |

## False negatives — we said stay, they churned (6 most confident)

| customer_id | churn_proba | recency_days | last_visit_days_ago | frequency_180d | monetary_180d | what happened |
|---|---|---|---|---|---|---|
| CUST00184 | 0.02 | 14 | 6 | 3 | 2,457 | Recent, active, often a strong spender — looked perfectly healthy at the snapshot and left anyway. The reason almost certainly sits outside our data: a competitor, a one-off need that's now met, or frustration we never logged. These are the misses worth studying. |
| CUST01990 | 0.05 | 59 | 7 | 4 | 3,878 | Recent, active, often a strong spender — looked perfectly healthy at the snapshot and left anyway. The reason almost certainly sits outside our data: a competitor, a one-off need that's now met, or frustration we never logged. These are the misses worth studying. |
| CUST01655 | 0.07 | 13 | 7 | 2 | 1,359 | Recent, active, often a strong spender — looked perfectly healthy at the snapshot and left anyway. The reason almost certainly sits outside our data: a competitor, a one-off need that's now met, or frustration we never logged. These are the misses worth studying. |
| CUST01303 | 0.12 | 20 | 0 | 1 | 845 | Recent, active, often a strong spender — looked perfectly healthy at the snapshot and left anyway. The reason almost certainly sits outside our data: a competitor, a one-off need that's now met, or frustration we never logged. These are the misses worth studying. |
| CUST02072 | 0.13 | 35 | 1 | 7 | 4,340 | Recent, active, often a strong spender — looked perfectly healthy at the snapshot and left anyway. The reason almost certainly sits outside our data: a competitor, a one-off need that's now met, or frustration we never logged. These are the misses worth studying. |
| CUST00592 | 0.13 | 20 | 1 | 1 | 627 | Recent, active, often a strong spender — looked perfectly healthy at the snapshot and left anyway. The reason almost certainly sits outside our data: a competitor, a one-off need that's now met, or frustration we never logged. These are the misses worth studying. |

## What I take from this
- The confident false positives are basically **dormant-but-loyal** customers, and our cheap-touch policy absorbs them without much waste.
- The false negatives are **engaged customers who left out of nowhere** — the current features simply don't see why. If I wanted to catch them, I'd go looking for new signals: competitor pricing, a worsening delivery experience, recent shifts in sentiment, or life-stage changes.
- On balance the trade-off is the right one for retention: we catch most churners, the misses are few, and the false alarms are cheap.