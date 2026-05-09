# Day 2: Containerizing with Docker

Welcome back. Yesterday you built an Iris prediction API with FastAPI. It runs perfectly on your machine. But the moment you hand it to a colleague, they hit dependency errors: wrong Python version, missing packages, different OS. This is the infamous "works on my machine" problem.

**Docker** eliminates it permanently.

## The Shipping Container Analogy

Before the 1950s, loading cargo onto ships was chaotic — every crate was a different shape, size, and material. The shipping industry was revolutionized by the **standardized container**: one universal box that fits on any ship, truck, or crane, regardless of what is inside.

Docker does this for software. You put your entire application — Python, packages, code, model file — into one standardized **container image**. That image runs identically on any machine that has Docker installed.

## Key Concepts

- **Image**: The blueprint (a recipe). Read-only. You build it once.
- **Container**: A running instance of an image. You can run thousands from one image.
- **Dockerfile**: A text file with instructions for building an image.
- **Registry**: A storage hub for images (Docker Hub, AWS ECR). Like GitHub, but for images.

## Step 1 — Install Docker

Download Docker Desktop from [docker.com](https://www.docker.com/products/docker-desktop/) and verify:

```bash
docker --version
```

```result
Docker version 24.0.5, build ced0996
```

## Step 2 — Write the Dockerfile

In the same folder as `app.py` and `iris_model.joblib`, create this file:

```dockerfile
# Dockerfile

# Start from an official Python image (slim = smaller download)
FROM python:3.11-slim

# Set the working directory inside the container
WORKDIR /app

# Copy and install dependencies first (Docker caches this layer)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the application code and model
COPY app.py .
COPY iris_model.joblib .

# Tell Docker which port the app listens on
EXPOSE 8000

# The command to run when the container starts
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

Create `requirements.txt`:

```text
fastapi==0.111.0
uvicorn==0.29.0
scikit-learn==1.4.2
joblib==1.4.2
numpy==1.26.4
pydantic==2.7.1
```

## Step 3 — Build the Image

```bash
docker build -t iris-api:v1 .
```

```result
[+] Building 23.4s (10/10) FINISHED
 => [1/5] FROM python:3.11-slim
 => [2/5] WORKDIR /app
 => [3/5] COPY requirements.txt .
 => [4/5] RUN pip install --no-cache-dir -r requirements.txt
 => [5/5] COPY app.py iris_model.joblib .
 => exporting to image: iris-api:v1
```

## Step 4 — Run the Container

```bash
docker run -p 8000:8000 iris-api:v1
```

```result
INFO:     Started server process [1]
INFO:     Uvicorn running on http://0.0.0.0:8000
```

The `-p 8000:8000` flag maps port 8000 on your laptop to port 8000 inside the container. Test it exactly like yesterday — the API behaves identically, but now it is running inside an isolated container.

```bash
curl -X POST "http://localhost:8000/predict" \
     -H "Content-Type: application/json" \
     -d '{"sepal_length": 6.3, "sepal_width": 2.5, "petal_length": 5.0, "petal_width": 1.9}'
```

```result
{"class_id": 2, "class_name": "virginica", "confidence": 0.98}
```

## Useful Docker Commands

```bash
# List all images on your machine
docker images

# List running containers
docker ps

# Stop a running container
docker stop <container_id>

# Remove an image
docker rmi iris-api:v1

# Run container in background (detached mode)
docker run -d -p 8000:8000 iris-api:v1
```

## Why Layer Order Matters

Notice the Dockerfile copies `requirements.txt` and runs `pip install` *before* copying `app.py`. This is deliberate. Docker caches each layer — if you only change `app.py`, Docker reuses the cached `pip install` layer and rebuilds in seconds instead of minutes. Always install dependencies before copying application code.

## Wrapping Up Day 2

You now have a self-contained, portable artifact: the `iris-api:v1` image. Zip it, ship it, or push it to Docker Hub and it will run anywhere. Tomorrow on **Day 3: Cloud Deployment**, we take that image and deploy it to the public internet so anyone with a URL can call your model.
