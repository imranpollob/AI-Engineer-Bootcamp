# Day 3: Cloud Deployment

Welcome to Day 3. You have a Docker image that runs your model API. Now it only exists on your laptop — the moment you shut your machine off, the API vanishes. Today we move it to the cloud so it runs 24/7 on a server you do not have to manage.

## Cloud Options for ML APIs

| Platform | Free Tier | Easiest For |
|----------|-----------|-------------|
| **Railway** | $5 credit/month | Quick deploys from GitHub |
| **Render** | 750 hrs/month | Always-on web services |
| **Fly.io** | 3 shared VMs free | Global edge deployment |
| **AWS/GCP/Azure** | Limited free tiers | Enterprise/scale |

We will use **Railway** — it is the fastest path from a Docker image to a live URL with zero server configuration.

## Step 1 — Push Your Code to GitHub

Railway deploys directly from a GitHub repository. Create a new repo and push your project:

```
iris-api/
├── app.py
├── train_and_save.py
├── iris_model.joblib
├── requirements.txt
└── Dockerfile
```

```bash
git init
git add .
git commit -m "Initial Iris API"
git remote add origin https://github.com/YOUR_USERNAME/iris-api.git
git push -u origin main
```

## Step 2 — Deploy on Railway

1. Go to [railway.app](https://railway.app) and sign in with GitHub
2. Click **New Project → Deploy from GitHub Repo**
3. Select your `iris-api` repository
4. Railway detects the `Dockerfile` automatically and starts building

Railway will give you a URL like `https://iris-api-production.up.railway.app` within 2-3 minutes.

## Step 3 — Add a railway.toml (Optional but Recommended)

A `railway.toml` file gives you fine-grained control over the deployment:

```toml
# railway.toml
[build]
builder = "DOCKERFILE"
dockerfilePath = "Dockerfile"

[deploy]
startCommand = "uvicorn app:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/"
healthcheckTimeout = 30
restartPolicyType = "ON_FAILURE"
```

Notice `$PORT` — Railway injects the port via an environment variable. Update `app.py` to respect it:

```python
# app.py (updated start command — no code change needed, just pass --port $PORT in railway.toml)
```

Update the Dockerfile CMD accordingly:

```dockerfile
CMD ["sh", "-c", "uvicorn app:app --host 0.0.0.0 --port ${PORT:-8000}"]
```

## Step 4 — Environment Variables

Secrets (API keys, database passwords) should never be hardcoded. Railway has a Variables tab where you set them:

```
MODEL_PATH = iris_model.joblib
LOG_LEVEL = info
```

Access them in Python:

```python
import os

model_path = os.getenv("MODEL_PATH", "iris_model.joblib")
model = joblib.load(model_path)
```

## Step 5 — Test the Live API

Once deployed, test your public URL:

```bash
curl -X POST "https://iris-api-production.up.railway.app/predict" \
     -H "Content-Type: application/json" \
     -d '{"sepal_length": 5.1, "sepal_width": 3.5, "petal_length": 1.4, "petal_width": 0.2}'
```

```result
{"class_id": 0, "class_name": "setosa", "confidence": 1.0}
```

Your model is now live on the internet. Anyone, anywhere, can call it.

## What Happens on Every Deploy

```
GitHub push → Railway detects change
           → Pulls new code
           → Runs: docker build -t iris-api .
           → Runs new container
           → Health check passes → Traffic switched to new container
           → Old container removed
```

This is **zero-downtime deployment** — users never experience an outage during updates.

## Wrapping Up Day 3

You have a live, public-facing ML API running on managed infrastructure. Push a commit, Railway redeploys automatically. But there is a problem you cannot see yet: you have no idea how the model is *actually performing* in production. Are predictions accurate? Is the data drifting? Tomorrow on **Day 4: MLflow**, we add experiment tracking so you can compare model versions and know exactly which one is in production.
