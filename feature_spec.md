# Feature Specification — Hosted DB Edition (Mathematical Definition)

> **No prebuilt views:** There is **no** helper view/materialized view. You must write the JSON expansion and
> aggregations yourself (e.g., CTE + `jsonb_array_elements`). Do **not** rely on server-side objects in the hosted DB.

This document defines each feature **mathematically** so you can implement it in **any** engine (SQL, Spark, Python, etc.).
Tables already exist on the hosted PostgreSQL and are indexed on `(customer_id, snapshot_date)`.

- Training features for model fit: `data/features_training.csv`
- Hosted tables (read-only):
  - `bank_transactions(customer_id, snapshot_date, transactions JSONB, label INT)`
  - `bank_transactions_scoring(customer_id, snapshot_date, transactions JSONB)`

---

## 1) Notation & windows

For a given row (customer) with snapshot date **S** and a multiset of transactions **T** = `{t₁, t₂, …}` extracted from
`transactions` (an array of objects). Each transaction **t** has fields:
- `ts(t) ∈ ℝ` — timestamp (UTC)
- `amt(t) ∈ ℝ₊` — absolute amount (strictly non‑negative number as stored)
- `type(t) ∈ {credit, debit}`
- `cat(t) ∈ Σ` — category, e.g., `salary`, `fees`, `gambling`, `cash_withdrawal`, …
- `merchant(t) ∈ Σ`
- `status(t) ∈ Σ` — e.g., `success`, `reversed` (we **do not** filter reversals)

Define time windows relative to **S**:

- **W₃₀** = `{ t ∈ T : ts(t) ≥ S − 30 days }`
- **W₉₀** = `{ t ∈ T : ts(t) ≥ S − 90 days }`

We treat missing sets as empty. Sums over empty sets are `0`. Means over empty sets are `0`. When a ratio denominator is
`0`, the feature value is defined as `0` unless stated otherwise.

Let indicator function `1[·]` return `1` if the predicate holds, else `0`.

---

## 2) Feature definitions (one vector per (customer_id, S))

1. **income_inflow_30d**
	note: sum of credit amounts in the last 30 days.
\[
\mathrm{income\_inflow\_{30d}}(S) = \sum_{t \in W_{30}} 1[\mathrm{type}(t)=\mathrm{credit}] \cdot \mathrm{amt}(t)
\]

2. **spend_outflow_30d**
	note: sum of debit amounts in the last 30 days.
\[
\mathrm{spend\_outflow\_{30d}}(S) = \sum_{t \in W_{30}} 1[\mathrm{type}(t)=\mathrm{debit}] \cdot \mathrm{amt}(t)
\]

3. **pct_gambling_spend_90d**
	note: fraction of debit spend that is gambling in last 90 days. If total debit = 0, return 0.
\[
\mathrm{pct\_gambling\_spend\_{90d}}(S) =
\frac{\sum_{t \in W_{90}} 1[\mathrm{type}(t)=\mathrm{debit} \land \mathrm{cat}(t)=\mathrm{gambling}] \cdot \mathrm{amt}(t)}
     {\max\left(\sum_{t \in W_{90}} 1[\mathrm{type}(t)=\mathrm{debit}] \cdot \mathrm{amt}(t),\ 0\right) + \varepsilon}
\]
with post‑processing: if denominator=0 then set value to `0`. Use `ε=1e−6` only to avoid division by zero.

4. **merchant_diversity_90d**
	note: number of distinct merchants in last 90 days.
\[
\mathrm{merchant\_diversity\_{90d}}(S) = \left| \{ \mathrm{merchant}(t) : t \in W_{90} \} \right|
\]

5. **avg_debit_amt_30d**
	note: arithmetic mean of debit amounts in last 30 days; `0` if no such transactions.
\[
\mathrm{avg\_debit\_amt\_{30d}}(S) =
\begin{cases}
0, & \text{if } \sum_{t \in W_{30}} 1[\mathrm{type}(t)=\mathrm{debit}] = 0 \\
\frac{\sum_{t \in W_{30}} 1[\mathrm{type}(t)=\mathrm{debit}] \cdot \mathrm{amt}(t)}
     {\sum_{t \in W_{30}} 1[\mathrm{type}(t)=\mathrm{debit}]}, & \text{otherwise}
\end{cases}
\]

6. **num_big_txn_30d**
	note: count of transactions with amount ≥ 500 (any type) in last 30 days.
\[
\mathrm{num\_big\_txn\_{30d}}(S) = \sum_{t \in W_{30}} 1[\mathrm{amt}(t) \ge 500]
\]

7. **days_since_last_salary**
	note: days since most recent `salary` transaction (any type) as of S; `999` if none exists.
\[
\mathrm{days\_since\_last\_salary}(S) =
\begin{cases}
999, & \text{if } \max\{\mathrm{ts}(t): \mathrm{cat}(t)=\mathrm{salary}\} \text{ does not exist} \\
(S - \max\{\mathrm{ts}(t): \mathrm{cat}(t)=\mathrm{salary}\}) \text{ in days}, & \text{otherwise}
\end{cases}
\]

8. **debit_credit_ratio_90d**
	note: debit sum divided by credit sum in last 90 days; if credit sum = 0, divide by `ε` (and/or map to large number),
then clamp to non‑negative reals.
\[
\mathrm{debit\_credit\_ratio\_{90d}}(S) =
\frac{\sum_{t \in W_{90}} 1[\mathrm{type}(t)=\mathrm{debit}] \cdot \mathrm{amt}(t)}
     {\sum_{t \in W_{90}} 1[\mathrm{type}(t)=\mathrm{credit}] \cdot \mathrm{amt}(t) + \varepsilon}
\]

9. **late_fee_count_90d**
	note: count of fee transactions in last 90 days.
\[
\mathrm{late\_fee\_count\_{90d}}(S) = \sum_{t \in W_{90}} 1[\mathrm{cat}(t)=\mathrm{fees}]
\]

10. **debit_txn_count_30d**
\[
\mathrm{debit\_txn\_count\_{30d}}(S) = \sum_{t \in W_{30}} 1[\mathrm{type}(t)=\mathrm{debit}]
\]

11. **credit_txn_count_30d**
\[
\mathrm{credit\_txn\_count\_{30d}}(S) = \sum_{t \in W_{30}} 1[\mathrm{type}(t)=\mathrm{credit}]
\]

12. **net_cash_flow_30d**
	note: signed net flow where credits are +amt and debits are −amt in last 30 days.
\[
\mathrm{net\_cash\_flow\_{30d}}(S) =
\sum_{t \in W_{30}} \left( 1[\mathrm{type}(t)=\mathrm{credit}] - 1[\mathrm{type}(t)=\mathrm{debit}] \right) \cdot \mathrm{amt}(t)
\]

13. **cash_withdrawal_90d**
\[
\mathrm{cash\_withdrawal\_{90d}}(S) = \sum_{t \in W_{90}} 1[\mathrm{cat}(t)=\mathrm{cash\_withdrawal}] \cdot \mathrm{amt}(t)
\]

Use `ε = 10^{-6}` wherever a denominator may be zero.

---

## 3) Implementation guidance (engine‑agnostic)

- **JSON extraction:** Expand the transaction array into rows; each element yields one transaction `t`.
- **Timestamps:** Parse ISO‑8601 `Z` as UTC and compare to **S**. Treat **S** as midnight (00:00) in UTC unless your engine
  uses date arithmetic natively; keep consistency across training vs. scoring.
- **Reversed transactions:** Do **not** exclude; treat them as regular rows.
- **Empty windows:** When no qualifying rows exist, use `0` for sums/means/counts and `999` for the salary gap metric.
- **Ratios:** Protect denominators with `max(den, ε)` and apply the stated post‑processing rules.
- **Performance:** Compute all features in a **single pass** over expanded transactions when possible.

---

## 4) Feature order contract (must match model)

Vectors must follow exactly this order (also saved in `model/feature_order.json`):

```
income_inflow_30d,
spend_outflow_30d,
pct_gambling_spend_90d,
merchant_diversity_90d,
avg_debit_amt_30d,
num_big_txn_30d,
days_since_last_salary,
debit_credit_ratio_90d,
late_fee_count_90d,
debit_txn_count_30d,
credit_txn_count_30d,
net_cash_flow_30d,
cash_withdrawal_90d
```

---

## 5) Validation & parity checks

- Recompute the vector for a sample customer from the hosted **training** table and compare to the row in
  `data/features_training.csv` (tolerances: exact for counts, ≤ 1e-6 for ratios).
- Ensure inference uses the same **window bounds** and timestamp handling as training.
- Confirm your final matrix columns are in the exact order listed above.