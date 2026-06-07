# Error Analysis (Part 3)

Model: XGBoost @ threshold 0.330. Test set n=336. Confusion matrix: TN=116, FP=52, FN=15, TP=153.

## Error rates & business risk

- **False positives (52)**: model flags a customer as churn-risk who actually stays. Business cost = a wasted (and deliberately cheap) retention touch. Low risk, acceptable in volume.
- **False negatives (15)**: model misses a real churner. Business cost = lost customer with no intervention — the expensive error. We chose a recall-leaning threshold precisely to keep this small.

## False positives — predicted churn, actually stayed (top 6 by confidence)

| customer_id | churn_proba | recency_days | last_visit_days_ago | frequency_180d | monetary_180d | interpretation |
|---|---|---|---|---|---|---|
| CUST01325 | 0.94 | 186 | 43 | 0 | 0 | Long gap since last order/visit, so the model expected churn — but the customer returned. These are recoverable lapsers; a retention touch here is cheap and often unnecessary. |
| CUST01017 | 0.94 | 133 | 13 | 2 | 1,167 | Long gap since last order/visit, so the model expected churn — but the customer returned. These are recoverable lapsers; a retention touch here is cheap and often unnecessary. |
| CUST00437 | 0.94 | 151 | 33 | 1 | 729 | Long gap since last order/visit, so the model expected churn — but the customer returned. These are recoverable lapsers; a retention touch here is cheap and often unnecessary. |
| CUST01370 | 0.93 | 161 | 35 | 2 | 1,246 | Long gap since last order/visit, so the model expected churn — but the customer returned. These are recoverable lapsers; a retention touch here is cheap and often unnecessary. |
| CUST01405 | 0.91 | 140 | 20 | 1 | 1,013 | Long gap since last order/visit, so the model expected churn — but the customer returned. These are recoverable lapsers; a retention touch here is cheap and often unnecessary. |
| CUST01614 | 0.90 | 103 | 4 | 2 | 1,352 | Long gap since last order/visit, so the model expected churn — but the customer returned. These are recoverable lapsers; a retention touch here is cheap and often unnecessary. |

## False negatives — predicted stay, actually churned (top 6 by confidence)

| customer_id | churn_proba | recency_days | last_visit_days_ago | frequency_180d | monetary_180d | interpretation |
|---|---|---|---|---|---|---|
| CUST00184 | 0.03 | 14 | 6 | 3 | 2,457 | Recent, active, often high-spend — looked healthy at snapshot but left anyway. Likely driven by factors outside the feature set (competitor, one-off need met, unlogged dissatisfaction). These are the costly misses to study for new features. |
| CUST01990 | 0.05 | 59 | 7 | 4 | 3,878 | Recent, active, often high-spend — looked healthy at snapshot but left anyway. Likely driven by factors outside the feature set (competitor, one-off need met, unlogged dissatisfaction). These are the costly misses to study for new features. |
| CUST01655 | 0.08 | 13 | 7 | 2 | 1,359 | Recent, active, often high-spend — looked healthy at snapshot but left anyway. Likely driven by factors outside the feature set (competitor, one-off need met, unlogged dissatisfaction). These are the costly misses to study for new features. |
| CUST02072 | 0.12 | 35 | 1 | 7 | 4,340 | Recent, active, often high-spend — looked healthy at snapshot but left anyway. Likely driven by factors outside the feature set (competitor, one-off need met, unlogged dissatisfaction). These are the costly misses to study for new features. |
| CUST01303 | 0.13 | 20 | 0 | 1 | 845 | Recent, active, often high-spend — looked healthy at snapshot but left anyway. Likely driven by factors outside the feature set (competitor, one-off need met, unlogged dissatisfaction). These are the costly misses to study for new features. |
| CUST00592 | 0.13 | 20 | 1 | 1 | 627 | Recent, active, often high-spend — looked healthy at snapshot but left anyway. Likely driven by factors outside the feature set (competitor, one-off need met, unlogged dissatisfaction). These are the costly misses to study for new features. |

## Takeaways
- The model's confident FPs are **dormant-but-loyal** customers; the cheap-touch retention policy absorbs them.
- The FNs are **engaged customers who left unexpectedly** — the signal isn't in the current features. Candidate new features: competitor pricing, delivery-experience trends, recent sentiment deltas, life-stage changes.
- Net: recall on churners is prioritised, FNs are few, and FPs are low-cost — appropriate for retention.