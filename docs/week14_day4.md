# Day 4: MLOps Basics — Experiment Tracking with MLflow

Welcome to Day 4. Your API is live. But imagine this: you train a new model with different hyperparameters, it gets slightly worse accuracy, you deploy it anyway, and two weeks later you cannot find the original model or remember what settings produced it.

This happens constantly in ML teams. **MLflow** prevents it.

## The Lab Notebook Analogy

Scientists keep meticulous lab notebooks: every experiment recorded with exact conditions, results, and observations. Without them, a successful experiment is impossible to reproduce. MLflow is your ML lab notebook — it automatically records every training run: parameters, metrics, artifacts, and the model itself.

## Install MLflow

```bash
pip install mlflow
```

## Core Concepts

- **Run**: A single training execution
- **Experiment**: A named group of related runs (e.g., "iris-classifier-v1")
- **Parameter**: An input to the run (e.g., `n_estimators=100`)
- **Metric**: An output measurement (e.g., `accuracy=0.97`)
- **Artifact**: Any file output (e.g., the saved model, plots)

## Step 1 — Instrument Your Training Code

```python
# train_with_mlflow.py
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, f1_score
import mlflow
import mlflow.sklearn

# Name the experiment — all related runs group here
mlflow.set_experiment("iris-classifier")

# Hyperparameters to test
configs = [
    {"n_estimators": 50,  "max_depth": 3},
    {"n_estimators": 100, "max_depth": 5},
    {"n_estimators": 200, "max_depth": None},
]

iris = load_iris()
X_train, X_test, y_train, y_test = train_test_split(
    iris.data, iris.target, test_size=0.2, random_state=42
)

for config in configs:
    with mlflow.start_run():
        # Log parameters
        mlflow.log_param("n_estimators", config["n_estimators"])
        mlflow.log_param("max_depth", config["max_depth"])

        # Train
        model = RandomForestClassifier(**config, random_state=42)
        model.fit(X_train, y_train)
        y_pred = model.predict(X_test)

        # Log metrics
        acc = accuracy_score(y_test, y_pred)
        f1  = f1_score(y_test, y_pred, average="weighted")
        mlflow.log_metric("accuracy", acc)
        mlflow.log_metric("f1_score", f1)

        # Log the model as an artifact
        mlflow.sklearn.log_model(model, "model")

        print(f"n_estimators={config['n_estimators']}, max_depth={config['max_depth']} "
              f"→ acc={acc:.3f}, f1={f1:.3f}")
```

```result
n_estimators=50,  max_depth=3    → acc=0.967, f1=0.967
n_estimators=100, max_depth=5    → acc=1.000, f1=1.000
n_estimators=200, max_depth=None → acc=1.000, f1=1.000
```

## Step 2 — Launch the MLflow UI

```bash
mlflow ui
```

```result
[2024-01-15 10:23:45] INFO mlflow.server: Starting gunicorn...
[2024-01-15 10:23:45] INFO mlflow.server: Listening at: http://127.0.0.1:5000
```

Open `http://127.0.0.1:5000` in your browser. You will see all three runs side by side with their parameters and metrics — click any column to sort, click any run to see full details, including the logged model artifact.

## Step 3 — Load a Specific Run's Model

Every run gets a unique `run_id`. To load the best model:

```python
# load_best_model.py
import mlflow.sklearn

# Copy the run_id from the MLflow UI
run_id = "a1b2c3d4e5f6..."

model = mlflow.sklearn.load_model(f"runs:/{run_id}/model")
print(model.predict([[5.1, 3.5, 1.4, 0.2]]))
```

```result
[0]   # setosa
```

## Step 4 — The Model Registry (Production-Ready)

For teams, MLflow has a **Model Registry** to formally promote models through stages:

```python
# Register the best model
mlflow.register_model(
    model_uri=f"runs:/{run_id}/model",
    name="iris-classifier"
)
```

Then transition it to Production via the UI or API:

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()
client.transition_model_version_stage(
    name="iris-classifier",
    version=1,
    stage="Production"
)
```

Now your `app.py` can always load whatever is tagged as Production — no hardcoded file paths needed.

```python
# Load the registered Production model
model = mlflow.sklearn.load_model("models:/iris-classifier/Production")
```

## What MLflow Tracks Automatically

Beyond what you log manually, `mlflow.sklearn.log_model()` automatically captures:
- Python version
- Library versions (scikit-learn, numpy, etc.)
- A `conda.yaml` / `requirements.txt` for reproducibility
- The model's input/output schema

## Wrapping Up Day 4

You now have a complete experiment history: every model you ever trained, with every parameter and metric, reproducible at any time. Tomorrow on **Day 5: Monitoring**, we shift from training-time concerns to production concerns — what happens *after* your model is live and real users are sending real data.
