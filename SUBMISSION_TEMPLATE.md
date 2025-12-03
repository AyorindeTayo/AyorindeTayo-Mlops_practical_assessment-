# Candidate Submission Template — MLE Assessment (Hosted DB)

> **Instructions**: Duplicate this template and fill in every section. Keep it concise but complete.
> Where code is requested, include *executable* snippets or point to files in your repo.

---

## 0) Checklist (tick before submission)
- [ ] Trained model on `data/features_training.csv`
- [ ] Artifacts saved: `model/model.xgb.json`, `model/feature_order.json`
- [ ] Scored hosted `bank_transactions_scoring` (read-only feature SQL)
- [ ] Output file with `customer_id, snapshot_date, probability, model_version`
- [ ] Lambda/Docker runnable (local) with example invocation
- [ ] Case study briefing completed (see `case_study/model_drift_case_study.md`)
- [ ] README with exact steps to reproduce

---


## 1) Repository layout
```
.
├─ data/
│  ├─ features_training.csv
│  └─ DB_CONNECTIONS.md   # no secrets in repo
├─ model/
│  ├─ model.xgb.json
│  └─ feature_order.json
├─ notebooks/
│  └─ train_credit_risk_model.ipynb
├─ sql/
│  └─ scoring_features.sql   # your read-only SELECT/CTE computing 13 features
├─ starter/
│  ├─ lambda_handler.py
│  ├─ Dockerfile
│  └─ requirements.txt
├─ scripts/                  # optional utilities
│  └─ <anything helpful>
├─ outputs/
│  └─ predictions.jsonl or predictions.csv
└─ README.md
```

---

## 2) How to reproduce (commands)
**Training**
```bash
# Train and save artifacts
# (Describe exact cells/commands to run the notebook or provide a CLI script here)
```

**DB Connection**
```bash
# How you set PGURL locally (example only; do not commit secrets)
export PGURL="postgres://<USER>:<PASS>@<HOST>:<PORT>/<DB>?sslmode=require"
```

**Feature SQL (hosted DB)**
```sql
-- paste your final read-only SELECT/CTE here that computes all 13 features
```

**Scoring**
```bash
# Show the commands or code you used to fetch features, apply model, and write outputs
```

**Lambda / Docker**
```bash
# Build and run; show a sample event and the returned probability
docker build -t credit-risk-lambda ./starter
docker run --rm -e AWS_DEFAULT_REGION=eu-west-1 credit-risk-lambda
# Example local invoke:
python -c 'import json,requests; print("see README for curl sample")'
```

---

## 3) Model details
- XGBoost parameters:
  - `n_estimators=`
  - `max_depth=`
  - `learning_rate=`
  - `subsample=`
  - `colsample_bytree=`
  - any others...
- Training metrics: e.g., `AUC=`, `PR AUC=`, calibration notes.
- Feature importance summary (optional).

---

## 4) Artifacts
- `model/model.xgb.json` — SHA256: `________`
- `model/feature_order.json` — SHA256: `________`

(Provide a short code snippet or shell one-liner you used to compute hashes.)

---

## 5) Outputs
- Path: `outputs/predictions.(csv|jsonl)`
- Schema: `customer_id, snapshot_date, probability, model_version`
- Sample (3–5 lines):

```csv
customer_id,snapshot_date,probability,model_version
S0001,2025-09-01,0.421,2.0.0-xgb
...
```

---

## 6) Case study response
Attach your 1–3 page briefing (PDF/MD). Ensure it follows the sections in
`case_study/model_drift_case_study.md`:
- triage & hypotheses
- data checks (with SQL)
- drift analysis plan (with thresholds)
- parity checks & feature contract
- remediation/retraining plan
- monitoring & risks

---

## 7) Assumptions & notes
- List any assumptions, constraints, and deviations from the spec here.
- Mention any trade-offs you made and why.