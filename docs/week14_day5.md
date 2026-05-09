# Day 5: Monitoring in Production

Welcome to Day 5. Your model is deployed and handling real requests. But here is a sobering truth: ML models can silently fail. Accuracy degrades gradually as the real world drifts away from the training distribution — and if you are not watching, you will not notice until a user complains.

Production monitoring is your model's early-warning system.

## The Weather Station Analogy

A weather station does not just record today's temperature — it tracks trends over time and raises alerts when readings deviate from historical norms. Production monitoring does the same for your model: it tracks prediction patterns and input distributions, raising alerts when they drift from what the model was trained on.

## Two Levels of Monitoring

**Level 1 — Request Logging** (always do this): Record every prediction the model makes — inputs, outputs, timestamps, and response times. This is your raw data for everything else.

**Level 2 — Data Drift Detection** (do this once you have logged data): Compare the statistical distribution of incoming data to your training data. If they diverge, model accuracy is likely falling even if no one has reported a problem yet.

## Step 1 — Add Logging to Your FastAPI App

```python
# app.py (updated with logging)
from fastapi import FastAPI
from pydantic import BaseModel
import joblib
import numpy as np
import json
import logging
from datetime import datetime, timezone

app = FastAPI(title="Iris Classifier API")
model = joblib.load("iris_model.joblib")
class_names = ["setosa", "versicolor", "virginica"]

# Structured logger — writes JSON lines to predictions.log
logging.basicConfig(
    filename="predictions.log",
    level=logging.INFO,
    format="%(message)s"
)
logger = logging.getLogger("predictions")

class IrisFeatures(BaseModel):
    sepal_length: float
    sepal_width: float
    petal_length: float
    petal_width: float

@app.post("/predict")
def predict(features: IrisFeatures):
    data = np.array([[
        features.sepal_length, features.sepal_width,
        features.petal_length, features.petal_width
    ]])
    prediction = model.predict(data)[0]
    probability = model.predict_proba(data)[0].max()

    result = {
        "class_id": int(prediction),
        "class_name": class_names[prediction],
        "confidence": round(float(probability), 4)
    }

    # Log every prediction as a JSON line
    log_entry = {
        "timestamp": datetime.now(timezone.utc).isoformat(),
        "input": features.model_dump(),
        "output": result
    }
    logger.info(json.dumps(log_entry))

    return result
```

After a few requests, `predictions.log` contains:

```result
{"timestamp": "2024-01-15T10:30:00+00:00", "input": {"sepal_length": 5.1, "sepal_width": 3.5, "petal_length": 1.4, "petal_width": 0.2}, "output": {"class_id": 0, "class_name": "setosa", "confidence": 1.0}}
{"timestamp": "2024-01-15T10:30:45+00:00", "input": {"sepal_length": 6.3, "sepal_width": 2.5, "petal_length": 5.0, "petal_width": 1.9}, "output": {"class_id": 2, "class_name": "virginica", "confidence": 0.98}}
```

## Step 2 — Analyze Prediction Logs

```python
# analyze_logs.py
import json
import pandas as pd
from pathlib import Path

# Load all logged predictions
rows = []
for line in Path("predictions.log").read_text().strip().splitlines():
    entry = json.loads(line)
    row = entry["input"].copy()
    row["predicted_class"] = entry["output"]["class_name"]
    row["confidence"] = entry["output"]["confidence"]
    row["timestamp"] = entry["timestamp"]
    rows.append(row)

df = pd.DataFrame(rows)

print("Prediction Distribution:")
print(df["predicted_class"].value_counts())
print(f"\nMean Confidence: {df['confidence'].mean():.3f}")
print(f"Low Confidence Predictions (<0.7): {(df['confidence'] < 0.7).sum()}")
print(f"\nFeature Means (Production):")
print(df[["sepal_length", "sepal_width", "petal_length", "petal_width"]].mean())
```

```result
Prediction Distribution:
setosa        45
virginica     38
versicolor    17

Mean Confidence: 0.923
Low Confidence Predictions (<0.7): 3

Feature Means (Production):
sepal_length    5.84
sepal_width     3.06
petal_length    3.76
petal_width     1.19
```

## Step 3 — Detect Data Drift with Evidently

**Evidently** is an open-source library that compares reference data (training set) to production data and generates drift reports.

```bash
pip install evidently
```

```python
# drift_report.py
import pandas as pd
from sklearn.datasets import load_iris
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset

# Reference = training data
iris = load_iris()
reference = pd.DataFrame(iris.data, columns=iris.feature_names)

# Current = production data (loaded from your log analyzer)
# For this demo, we simulate a drift by shifting petal measurements
current = reference.copy()
current["petal length (cm)"] += 0.5   # Simulated drift

report = Report(metrics=[DataDriftPreset()])
report.run(reference_data=reference, current_data=current)
report.save_html("drift_report.html")
print("Drift report saved to drift_report.html")
```

Open `drift_report.html` in your browser — Evidently shows per-feature drift scores and highlights which features have changed significantly.

## Step 4 — Set a Confidence Alert

A simple but effective monitor: alert when confidence drops below a threshold.

```python
# Add to app.py
import os

CONFIDENCE_THRESHOLD = float(os.getenv("CONFIDENCE_THRESHOLD", "0.7"))

@app.post("/predict")
def predict(features: IrisFeatures):
    # ... (same as above)

    if probability < CONFIDENCE_THRESHOLD:
        logger.warning(json.dumps({
            "alert": "low_confidence",
            "confidence": float(probability),
            "input": features.model_dump()
        }))

    return result
```

In production, you would wire this warning to Slack, PagerDuty, or an email alert — any prediction the model is uncertain about gets flagged for human review.

## Wrapping Up Day 5

Your model now has eyes on it: every prediction is logged, feature drift is detectable, and low-confidence predictions trigger alerts. Tomorrow on **Day 6: CI/CD**, we automate the entire workflow so that when you push new training code to GitHub, the model automatically retrains, gets evaluated, and redeploys — all without manual steps.
