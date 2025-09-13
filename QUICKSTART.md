# Quick Start Guide

## Prerequisites
- Docker and Docker Compose installed
- Python 3.9+ (for running examples)

## 1. Start MLflow Platform (Recommended)

```bash
# Start MLflow with PostgreSQL backend
docker compose -f mlflow-docker-compose.yaml up -d

# Check if services are healthy
docker compose -f mlflow-docker-compose.yaml ps

# Access MLflow UI
open http://localhost:5001
```

## 2. Test the Setup

```bash
# Install Python dependencies
pip install -r requirements.txt

# Run smoke test
cd examples/
python smoke_test.py

# Check results in MLflow UI
```

## 3. Start Trino Platform (Optional)

```bash
# Start Trino with PostgreSQL
docker compose -f trino-docker-compose.yaml up -d

# Access Trino UI
open http://localhost:8090
```

## 4. Explore Docker Concepts

```bash
# Compare image sizes
docker build -t standard -f mlflow/Dockerfile --build-arg MLFLOW_VERSION=2.3.2 mlflow/
docker build -t multistage -f mlflow/Dockerfile-multistage-build --build-arg MLFLOW_VERSION=2.3.2 mlflow/

# Check image sizes
docker images | grep -E "(standard|multistage)"
```

## 5. Clean Up

```bash
# Stop all services
docker compose -f mlflow-docker-compose.yaml down
docker compose -f trino-docker-compose.yaml down

# Remove volumes (WARNING: This deletes all data)
docker compose -f mlflow-docker-compose.yaml down -v
docker compose -f trino-docker-compose.yaml down -v
```