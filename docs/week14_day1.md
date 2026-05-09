# Day 1: Serving Models with FastAPI

Welcome to Week 14! Over the last 13 weeks, you built a complete toolkit: Python, Math, Classical ML, and Deep Learning all the way through Transformers and Transfer Learning. But as Week 13 warned — a model sitting in a Jupyter Notebook is useless to the real world.

This week we fix that. We deploy.

## The Restaurant Analogy

Think of your trained model as a chef locked in a kitchen. Right now, no one can get food because there is no waiter, no menu, no way to place an order. A **REST API** is the waiter. It receives orders (input data) from the outside world, carries them to the kitchen (model), and brings back the food (predictions).

**FastAPI** is Python's fastest framework for building those waiters. It is modern, type-safe, and automatically generates interactive documentation for your API.

## Step 1 — Train and Save a Model

Before we can serve anything, we need a trained model saved to disk. We use `joblib` to save and load Scikit-Learn models.

```python
# train_and_save.py
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
import joblib

iris = load_iris()
X, y = iris.data, iris.target

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

print(f"Test Accuracy: {model.score(X_test, y_test):.2f}")
joblib.dump(model, "iris_model.joblib")
print("Model saved to iris_model.joblib")
```

```result
Test Accuracy: 1.00
Model saved to iris_model.joblib
```

## Step 2 — Build the FastAPI App

Install the dependencies first:

```bash
pip install fastapi uvicorn joblib scikit-learn
```

Now create the API:

```python
# app.py
from fastapi import FastAPI
from pydantic import BaseModel
import joblib
import numpy as np

app = FastAPI(title="Iris Classifier API")

# Load the model once at startup — not on every request!
model = joblib.load("iris_model.joblib")
class_names = ["setosa", "versicolor", "virginica"]

# Define the shape of incoming requests using Pydantic
class IrisFeatures(BaseModel):
    sepal_length: float
    sepal_width: float
    petal_length: float
    petal_width: float

@app.get("/")
def health_check():
    return {"status": "ok", "model": "Iris RandomForest"}

@app.post("/predict")
def predict(features: IrisFeatures):
    data = np.array([[
        features.sepal_length,
        features.sepal_width,
        features.petal_length,
        features.petal_width
    ]])
    prediction = model.predict(data)[0]
    probability = model.predict_proba(data)[0].max()

    return {
        "class_id": int(prediction),
        "class_name": class_names[prediction],
        "confidence": round(float(probability), 4)
    }
```

## Step 3 — Run and Test It

Start the server:

```bash
uvicorn app:app --reload
```

```result
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
INFO:     Application startup complete.
```

Test it with `curl` in a second terminal:

```bash
curl -X POST "http://127.0.0.1:8000/predict" \
     -H "Content-Type: application/json" \
     -d '{"sepal_length": 5.1, "sepal_width": 3.5, "petal_length": 1.4, "petal_width": 0.2}'
```

```result
{"class_id": 0, "class_name": "setosa", "confidence": 1.0}
```

FastAPI also gives you free interactive documentation at `http://127.0.0.1:8000/docs` — open it in your browser and test the API visually with real inputs.

## Why FastAPI over Flask?

| Feature | Flask | FastAPI |
|--------|-------|---------|
| Speed | Moderate | Very fast (async-native) |
| Validation | Manual | Automatic via Pydantic |
| Documentation | Manual | Auto-generated (Swagger UI) |
| Type hints | Optional | First-class |

For ML serving, FastAPI is the modern standard.

## Wrapping Up Day 1

You just turned your trained model into a live web service that anyone can call from anywhere. Tomorrow on **Day 2: Docker**, we solve the "but it works on my machine" problem by packaging this API into a container that runs identically everywhere — your laptop, a server in Tokyo, or a cloud VM in São Paulo.
