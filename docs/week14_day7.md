# Day 7: Deployment Capstone — End-to-End ML Pipeline

Welcome to Day 7, the Week 14 capstone. This week you have built every piece of a production ML system separately: FastAPI, Docker, cloud deployment, MLflow, monitoring, and CI/CD. Today you combine them into one cohesive project — a Wine Quality Classifier deployed end-to-end.

## The Target Architecture

```
Training Data
     │
     ▼
train_and_save.py ──→ MLflow (log metrics + artifact)
     │
     ▼
iris_model.joblib
     │
     ▼
evaluate.py ──────→ Quality Gate (exit 1 if accuracy < threshold)
     │
     ▼
app.py ────────────→ FastAPI (POST /predict + logging)
     │
     ▼
Dockerfile ────────→ Docker Image
     │
     ▼
GitHub Actions ────→ CI/CD (auto-build + auto-deploy)
     │
     ▼
Railway ───────────→ Live URL (24/7)
```

## The Dataset: Wine Quality

The [Wine Quality dataset](https://archive.ics.uci.edu/ml/datasets/wine+quality) contains 11 chemical measurements for wines and a quality score from 3–8. We will frame it as binary classification: quality ≥ 6 = "good", quality < 6 = "not good".

## Complete Project Code

```python
# train_and_save.py
import pandas as pd
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.metrics import accuracy_score, f1_score, classification_report
import joblib
import mlflow
import mlflow.sklearn

mlflow.set_experiment("wine-quality-classifier")

url = "https://archive.ics.uci.edu/ml/machine-learning-databases/wine-quality/winequality-red.csv"
df = pd.read_csv(url, sep=";")

df["good"] = (df["quality"] >= 6).astype(int)
X = df.drop(["quality", "good"], axis=1)
y = df["good"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

with mlflow.start_run():
    params = {"n_estimators": 150, "max_depth": 4, "learning_rate": 0.1}
    mlflow.log_params(params)

    pipeline = Pipeline([
        ("scaler", StandardScaler()),
        ("model", GradientBoostingClassifier(**params, random_state=42))
    ])
    pipeline.fit(X_train, y_train)

    y_pred = pipeline.predict(X_test)
    acc = accuracy_score(y_test, y_pred)
    f1  = f1_score(y_test, y_pred)

    mlflow.log_metric("accuracy", acc)
    mlflow.log_metric("f1_score", f1)
    mlflow.sklearn.log_model(pipeline, "model")

    print(f"Accuracy: {acc:.4f} | F1: {f1:.4f}")
    print(classification_report(y_test, y_pred, target_names=["not good", "good"]))

joblib.dump(pipeline, "wine_model.joblib")
print("Model saved.")
```

```result
Accuracy: 0.8156 | F1: 0.8491
              precision    recall  f1-score   support

    not good       0.75      0.71      0.73       116
        good       0.85      0.87      0.86       204

    accuracy                           0.82       320
```

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
import joblib
import numpy as np
import json, logging
from datetime import datetime, timezone

app = FastAPI(title="Wine Quality Classifier API")
model = joblib.load("wine_model.joblib")

logging.basicConfig(filename="predictions.log", level=logging.INFO, format="%(message)s")
logger = logging.getLogger("predictions")

class WineFeatures(BaseModel):
    fixed_acidity: float
    volatile_acidity: float
    citric_acid: float
    residual_sugar: float
    chlorides: float
    free_sulfur_dioxide: float
    total_sulfur_dioxide: float
    density: float
    pH: float
    sulphates: float
    alcohol: float

@app.get("/")
def health():
    return {"status": "ok", "model": "Wine Quality GradientBoosting"}

@app.post("/predict")
def predict(features: WineFeatures):
    data = np.array([[
        features.fixed_acidity, features.volatile_acidity, features.citric_acid,
        features.residual_sugar, features.chlorides, features.free_sulfur_dioxide,
        features.total_sulfur_dioxide, features.density, features.pH,
        features.sulphates, features.alcohol
    ]])
    prediction = model.predict(data)[0]
    probability = model.predict_proba(data)[0].max()
    label = "good" if prediction == 1 else "not good"

    result = {"prediction": label, "confidence": round(float(probability), 4)}
    logger.info(json.dumps({
        "ts": datetime.now(timezone.utc).isoformat(),
        "input": features.model_dump(),
        "output": result
    }))
    return result
```

```python
# evaluate.py
from sklearn.metrics import accuracy_score
from sklearn.datasets import make_classification
import joblib, sys, pandas as pd
from sklearn.model_selection import train_test_split

THRESHOLD = 0.78

url = "https://archive.ics.uci.edu/ml/machine-learning-databases/wine-quality/winequality-red.csv"
df = pd.read_csv(url, sep=";")
df["good"] = (df["quality"] >= 6).astype(int)
X = df.drop(["quality", "good"], axis=1)
y = df["good"]
_, X_test, _, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

model = joblib.load("wine_model.joblib")
acc = accuracy_score(y_test, model.predict(X_test))
print(f"Accuracy: {acc:.4f} (threshold: {THRESHOLD})")
sys.exit(0 if acc >= THRESHOLD else 1)
```

## Testing the Live API

```bash
curl -X POST "http://localhost:8000/predict" \
  -H "Content-Type: application/json" \
  -d '{
    "fixed_acidity": 7.4, "volatile_acidity": 0.7, "citric_acid": 0.0,
    "residual_sugar": 1.9, "chlorides": 0.076, "free_sulfur_dioxide": 11.0,
    "total_sulfur_dioxide": 34.0, "density": 0.9978, "pH": 3.51,
    "sulphates": 0.56, "alcohol": 9.4
  }'
```

```result
{"prediction": "not good", "confidence": 0.8234}
```

## What You Built This Week

| Day | Skill | Tool |
|-----|-------|------|
| 1 | Model serving | FastAPI |
| 2 | Containerization | Docker |
| 3 | Cloud deployment | Railway |
| 4 | Experiment tracking | MLflow |
| 5 | Monitoring | Logging + Evidently |
| 6 | Automation | GitHub Actions |
| 7 | Capstone | Full pipeline |

## Wrapping Up Week 14

Congratulations — you now know how to take any trained model and turn it into a production-grade system with logging, monitoring, and automated deployment. This is the operational foundation every AI engineer needs.

But your models so far have all been trained from scratch on small datasets. The industry has shifted dramatically: instead of training models, most AI engineers now *use* models that have already been trained on trillions of tokens — **Large Language Models**. Next week in **Week 15: Large Language Models & Prompt Engineering**, we enter the era of modern AI engineering.
