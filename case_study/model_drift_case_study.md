# Case Study — Production Accuracy Dropped ~20% in 3 Weeks (Hosted DB Context)

## Background
You are the on-call MLOps/ML Engineer for a **credit-risk** model that scores customers using features derived from
nested bank-transaction JSON (see `feature_spec.md`). The model was trained on features calculated from historical
data similar to `bank_transactions` and served in production against `bank_transactions_scoring`.
- Model family: Gradient-boosted trees (XGBoost)
- Training period: **up to 2025‑08‑31**
- Production scoring: **from 2025‑09‑01 onward**
- Labels (defaults) arrive with a **60–90 day delay**
- Serving stack: batch scoring → persisted predictions → downstream decisioning
- Feature contract: **13 features** in the **exact** order in `model/feature_order.json`

### Incident
After a stable first week, business reports a **~20% relative drop** in accuracy (AUC and precision at business
threshold) over the following two weeks.

## Your Task
Produce a written, technical briefing that covers the following. You may include code/SQL blocks.
Timebox guidance: **90–120 minutes** of reasoning and outline depth; no need to fully implement code.

### 1) Rapid triage & hypotheses
- Enumerate **top 5 hypotheses**, prioritize, and explain *why* (impact × likelihood × time-to-check).
- Include a **stakeholder comms plan** (what to say in the first hour vs. day 1).

### 2) Data integrity checks (hosted Postgres)
Write **SQL queries** (read‑only) to check for:
- Record volume anomalies (by day), per `(customer_id, snapshot_date)`.
- Time windows / timezone issues (e.g., `ts` near window boundaries, unexpected local-time shifts).
- Schema/semantics (new or missing `category` values; null/NaN `amount`; weird `status` ratios).
- Duplicates or late-arriving transactions.
- Currency/scale issues (orders of magnitude).

> Use the lateral JSON expansion pattern from `feature_spec.md` in your queries.

### 3) Feature drift analysis
- Define **metrics** to detect drift (e.g., PSI for each of the 13 features, KL/Wasserstein optional).
- Provide **pseudo-code** or code snippets to compute PSI from two windows (train vs. recent prod).
- Propose **alert thresholds** and what actions they trigger.

### 4) Concept drift & label delay
- Explain how you can detect *concept* drift given delayed labels (e.g., delayed back-tests, weak-proxy outcomes).
- Show how you would **segment** performance (income bands, merchant diversity, high-gambling cohort) to localize issues.
- Recommend **temporary mitigations** (threshold nudges, guard-rails) until retraining completes.

### 5) Serving parity & feature contract
- Describe tests to ensure **parity** between offline feature code and prod SQL (window bounds, casting, reversals).
- Verify **feature order** is enforced at serve time (read `feature_order.json` and reorder columns).

### 6) Remediation & retraining plan
- **Data windows** for retraining; whether to include most recent 2–4 weeks.
- **Validation & back-test** plan (rolling windows, out-of-time splits).
- **Deployment**: champion–challenger with canary, rollback plan.
- **Monitoring**: dashboards (data quality + drift + performance once labels land).

### 7) Risks & controls
- Compliance/privacy considerations.
- Change management, versioning of **data**, **features**, **code**, and **artifacts**.

## Deliverables for this case study
- A 1–3 page briefing that follows the sections above.
- A small appendix with **SQL snippets** for your top checks and a diagram or bullet list of the **monitoring plan**.

## Evaluation
We will assess clarity, prioritization, depth of reasoning, understanding of drift types, practicality of the plan,
and alignment with the feature contract and hosted-DB constraints.

---

## Reference: Recent Prod Feature Snapshot (anonymized)

To anchor your PSI example, we provide a **tiny** sample of recent production feature stats
covering **2025-09-01 → 2025-09-14** (aggregated; anonymized). Use these as *illustrative*
targets to compare against your **training** distributions (which you should compute from `data/features_training.csv`).
Do **not** assume these reflect the exact DB; they are deliberately approximate to avoid giving away answers.

A CSV with the sample is included at: `case_study/recent_prod_feature_stats_sample.csv`

| feature                   | mean   | std    | p10   | p50   | p90   | n  |
|--------------------------|--------|--------|-------|-------|-------|----|
| income_inflow_30d        | 23500  | 12000  | 8000  | 22000 | 40000 | 160 |
| spend_outflow_30d        | 21000  | 11500  | 7000  | 20000 | 38000 | 160 |
| debit_credit_ratio_90d   | 1.35   | 0.80   | 0.40  | 1.20  | 2.80  | 160 |
| merchant_diversity_90d   | 24     | 10     | 10    | 23    | 40    | 160 |
| pct_gambling_spend_90d   | 0.06   | 0.09   | 0.00  | 0.02  | 0.22  | 160 |

### How to use this in your PSI example
1. Compute **training** quantile bins (e.g., deciles) for each feature using `features_training.csv`.
2. Place the **recent prod** values into those bins and compute PSI per feature.
3. Explain your **thresholds** (e.g., PSI > 0.2 moderate drift, > 0.3 large) and prioritization for investigation.
4. Show a short code or pseudo-code snippet for one feature.

> You may choose different binning strategies (e.g., fixed bins from domain knowledge); be explicit and consistent.