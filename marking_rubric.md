# Marking Rubric (100 pts)
## Stage 1 — Data handling (35)
- JSONB expansion/windowing correctness (15)
- Feature calculations match spec (15)
- Query performance & clarity (5)

## Stage 2 — Model & deployment (45)
- XGBoost training reproducibility (10)
- Artifacts (model.xgb.json + feature_order.json) (10)
- Lambda/Docker correctness & I/O schema (15)
- Local test / example invocation (10)

## Stage 3 — Case study (20)
- Drift diagnosis depth (10)
- Monitoring/retrain strategy (10)

Bonus up to +10 for CI, types, robust errors, repo quality.


**Rule:** No prebuilt/assessor-provided views or flattened tables. Candidates must demonstrate JSONB handling and correct feature aggregation in SQL.
