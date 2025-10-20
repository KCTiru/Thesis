# 🧠 CNN Container Setup Guide

This guide explains how to build, configure, and run the **CNN-based image prediction service** used in this project.  
It uses **FastAPI** + **PyTorch** and runs inside a **Docker container** for reproducibility and offline capability.

---

## 📘 Overview

The CNN container is responsible for:
- Receiving captured images from the Raspberry Pi (`single.py`)
- Running inference using a trained PyTorch model (e.g., `resnet.pth`)
- Sending prediction results to the self-hosted **Supabase** backend

This container is designed to:
- Work **offline** in local research environments  
- Integrate easily with the rest of the system via REST API  
- Be easily **retrained** or replaced by future developers

---

## 🧩 Folder Structure

Expected structure inside `/app`:

```text
├── app/
│ └── server.py # FastAPI server that handles image inference
├── model/
│ └── resnet.pth # Trained PyTorch model (replaceable)
├── Dockerfile # Docker build configuration
```


---

## ⚙️ Step 1. Verify Prerequisites

Make sure you have the following installed:

| Tool | Version | Purpose |
|------|----------|----------|
| Docker | ≥ 24.x | Runs the containerized service |
| Python | ≥ 3.10 | (Optional) For local testing before Docker |
| Git | Latest | To clone and manage code |

If running on a Raspberry Pi or local machine, ensure you can connect to your self-hosted Supabase backend.

---

## 🧱 Step 2. Review the Dockerfile

Here’s the Dockerfile included in this project:

```dockerfile
FROM python:3.10-slim

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender1 \
    curl \
    && rm -rf /var/lib/apt/lists/*

COPY ./model /model
COPY ./app /app

RUN pip install --no-cache-dir \
    torch==2.8.0+cpu torchvision==0.23.0+cpu --extra-index-url https://download.pytorch.org/whl/cpu && \
    pip install --no-cache-dir \
    fastapi \
    "uvicorn[standard]" \
    python-multipart \
    Pillow \
    psycopg2-binary

EXPOSE 8080
ENV MODEL_PATH=/model/resnet.pth
CMD ["uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8080"]
```

🧠 Key Notes:

Uses CPU-only PyTorch (for lightweight deployment)

Exposes port 8080

The model file is read from /model/resnet.pth

Starts the FastAPI app defined in server.py

---

## 🚀 Step 3. Build the Container

Run this from the project root or /inference directory:
```bash
docker build -t cnn-service .
```
This command:

Downloads dependencies

Copies your app and model files

Packages everything into a single container image named cnn-service

---

## ▶️ Step 4. Run the Container

Once built, start the container:
```bash
docker run -d -p 8080:8080 cnn-service
```

Confirm it’s running:
```bash
docker ps
```

You should see:
```bash
CONTAINER ID   IMAGE          COMMAND                  PORTS                    NAMES
<id>           cnn-service    "uvicorn server:app…"    0.0.0.0:8080->8080/tcp   cnn-service
```