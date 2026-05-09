# Day 6: CI/CD for ML Pipelines

Welcome to Day 6. So far you have built every piece of the deployment puzzle manually: train the model, save it, build the image, push to GitHub, Railway redeploys. But "manually" does not scale. A professional ML pipeline runs these steps automatically whenever code changes — with quality gates that prevent bad models from reaching production.

This is **CI/CD**: Continuous Integration (catch problems early) and Continuous Deployment (ship automatically when checks pass).

## The Assembly Line Analogy

A car factory does not build one car at a time by hand. It has an assembly line: each station performs one step, the car only moves forward when the station's check passes, and a defect at any station stops the line before it reaches the customer. GitHub Actions is your ML assembly line.

## GitHub Actions Basics

GitHub Actions runs workflows defined as YAML files in `.github/workflows/`. A workflow triggers on events (push, pull request, schedule) and runs a series of **jobs** made of **steps**.

```
Trigger (push to main)
  └─ Job: train-and-deploy
       ├─ Step 1: Checkout code
       ├─ Step 2: Install dependencies
       ├─ Step 3: Train model
       ├─ Step 4: Evaluate — fail if accuracy < threshold
       ├─ Step 5: Build Docker image
       └─ Step 6: Push to Railway
```

## Step 1 — Project Structure

Add a `tests/` folder and evaluation script:

```
iris-api/
├── app.py
├── train_and_save.py
├── evaluate.py          ← New: quality gate
├── requirements.txt
├── Dockerfile
└── .github/
    └── workflows/
        └── ml_pipeline.yml
```

## Step 2 — Write the Evaluation Script

```python
# evaluate.py
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
import joblib
import sys

ACCURACY_THRESHOLD = 0.95

iris = load_iris()
X_train, X_test, y_train, y_test = train_test_split(
    iris.data, iris.target, test_size=0.2, random_state=42
)

model = joblib.load("iris_model.joblib")
y_pred = model.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)

print(f"Evaluation Accuracy: {accuracy:.4f}")
print(f"Threshold: {ACCURACY_THRESHOLD}")

if accuracy < ACCURACY_THRESHOLD:
    print("FAIL: Model did not meet accuracy threshold. Blocking deploy.")
    sys.exit(1)   # Non-zero exit code = pipeline failure
else:
    print("PASS: Model meets threshold. Proceeding to deploy.")
    sys.exit(0)
```

## Step 3 — Write the GitHub Actions Workflow

```yaml
# .github/workflows/ml_pipeline.yml
name: ML Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  train-evaluate-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python 3.11
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Train model
        run: python train_and_save.py

      - name: Evaluate model (quality gate)
        run: python evaluate.py
        # If this exits with code 1, the pipeline stops here.
        # The Docker build and deploy steps below never run.

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/iris-api:latest

      - name: Deploy to Railway
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}
        run: |
          npm install -g @railway/cli
          railway up --detach
```

## Step 4 — Add Secrets to GitHub

In your GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**

Add:
- `DOCKERHUB_USERNAME` — your Docker Hub username
- `DOCKERHUB_TOKEN` — Docker Hub access token (not your password)
- `RAILWAY_TOKEN` — from Railway dashboard → Account Settings → Tokens

## Step 5 — Push and Watch It Run

```bash
git add .
git commit -m "Add CI/CD pipeline"
git push origin main
```

Go to your GitHub repo → **Actions** tab. You will see the workflow running. Each step shows green or red. If the evaluation step fails, the deploy never happens — your production API stays on the last known-good version.

```result
✓ Checkout code                  (2s)
✓ Set up Python 3.11             (8s)
✓ Install dependencies           (34s)
✓ Train model                    (12s)
✓ Evaluate model (quality gate)  (3s)  ← Accuracy: 1.000 ≥ 0.95 → PASS
✓ Set up Docker Buildx           (2s)
✓ Build and push Docker image    (47s)
✓ Deploy to Railway              (18s)
```

## Wrapping Up Day 6

Your ML pipeline is now fully automated: push code → model trains → quality gate checks → Docker image builds → deploys to production, all without touching a single command manually. Tomorrow on **Day 7: Capstone**, you wire everything from this week into one cohesive end-to-end system and deploy a model you train yourself from scratch.
