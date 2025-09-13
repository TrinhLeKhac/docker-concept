# Docker MLOps Platform

This project demonstrates Docker containerization concepts and provides a complete MLOps platform setup using MLflow and Trino with Docker Compose.

## Project Structure

```
docker/
├── mlflow/                          # MLflow Docker configurations
│   ├── Dockerfile                   # Standard MLflow container
│   ├── Dockerfile-multistage-build  # Optimized multi-stage build
│   ├── Dockerfile-cmd-vs-entrypoint # CMD vs ENTRYPOINT demonstration
│   └── .dockerignore               # Docker ignore file
├── docker-user/                     # User management demonstration
│   └── Dockerfile                   # Example of creating non-root user
├── examples/                       # Example scripts and usage
│   ├── smoke_test.py               # MLflow smoke test script
│   └── README.md                   # Examples documentation
├── docs/                           # Additional documentation
│   └── DOCKER_CONCEPTS.md          # Docker concepts explained
├── .env                            # Environment variables
├── requirements.txt                # Python dependencies
├── mlflow-docker-compose.yaml      # MLflow with PostgreSQL setup
├── trino-docker-compose.yaml       # Trino with PostgreSQL setup
└── README.md                       # This file
```

## Environment Configuration

The `.env` file contains all necessary environment variables:

```bash
# PostgreSQL Database Configuration
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=mlflowdb
POSTGRES_USER=mlopsvn
POSTGRES_PASSWORD=mlopsvn

# MLflow Configuration
BACKEND_STORE_URI=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}
MLFLOW_VERSION=2.3.2
```

## Docker Concepts Demonstrated

### 1. Multi-Stage Build Optimization

**Purpose**: Reduce final image size by separating build dependencies from runtime dependencies.

#### Before (Standard Build)
```bash
docker build -t before_msb -f mlflow/Dockerfile --build-arg MLFLOW_VERSION=2.3.2 mlflow
docker run -p 5000:5000 before_msb
```

#### After (Multi-Stage Build)
```bash
docker build -t after_msb -f mlflow/Dockerfile-multistage-build --build-arg MLFLOW_VERSION=2.3.2 mlflow
docker run -p 5000:5000 after_msb
```

**Benefits**:
- Smaller final image size (alpine-based vs slim-based)
- Cleaner runtime environment
- Better security (fewer packages in final image)

### 2. CMD vs ENTRYPOINT

The `Dockerfile-cmd-vs-entrypoint` demonstrates the difference:
- **ENTRYPOINT**: Makes container executable, cannot be overridden
- **CMD**: Provides default arguments, can be overridden at runtime

### 3. User Management

The `docker-user/Dockerfile` shows how to create and use non-root users for security best practices.

## Docker Compose Services

### MLflow Platform (`mlflow-docker-compose.yaml`)

This setup provides a complete MLflow tracking server with PostgreSQL backend:

**Services**:
- **postgresql**: Database backend for MLflow
  - Image: `postgres:13`
  - Port: `5432` (host) → `5432` (container)
  - Persistent storage via Docker volume
  - Health checks enabled
  
- **mlflow**: MLflow tracking server
  - Built from local Dockerfile
  - Port: `5001` (host) → `5000` (container)
  - Depends on PostgreSQL health check
  - Persistent artifact storage

**Features**:
- Health checks for both services
- Service dependencies (MLflow waits for PostgreSQL)
- Persistent data volumes
- Custom network for service communication

#### Usage:
```bash
# Start MLflow platform
docker compose -f mlflow-docker-compose.yaml up -d

# View logs
docker compose -f mlflow-docker-compose.yaml logs -f

# Stop services
docker compose -f mlflow-docker-compose.yaml down

# Stop and remove volumes (WARNING: This deletes all data)
docker compose -f mlflow-docker-compose.yaml down -v
```

**Access**: MLflow UI available at http://localhost:5001

### Trino Platform (`trino-docker-compose.yaml`)

This setup provides a Trino query engine with PostgreSQL:

**Services**:
- **trino**: Distributed SQL query engine
  - Image: `trinodb/trino:410`
  - Port: `8090` (host) → `8080` (container)
  
- **postgresql**: Database for Trino metadata
  - Image: `postgres:13`
  - Port: `5433` (host) → `5432` (container)

#### Usage:
```bash
# Start Trino platform
docker compose -f trino-docker-compose.yaml up -d

# View logs
docker compose -f trino-docker-compose.yaml logs -f

# Stop services
docker compose -f trino-docker-compose.yaml down
```

**Access**: Trino UI available at http://localhost:8090

## Testing the Setup

### MLflow Smoke Test

Use the provided smoke test script to verify MLflow functionality:

```bash
# Ensure MLflow is running
docker compose -f mlflow-docker-compose.yaml up -d

# Run smoke test
cd examples/
python smoke_test.py 0.5 0.5
```

The script:
- Downloads wine quality dataset
- Trains an ElasticNet model
- Logs parameters, metrics, and model to MLflow
- Demonstrates model registry functionality

## Best Practices Implemented

1. **Multi-stage builds** for optimized image sizes
2. **Health checks** for service reliability
3. **Service dependencies** for proper startup order
4. **Persistent volumes** for data durability
5. **Environment variables** for configuration management
6. **Custom networks** for service isolation
7. **Non-root users** for security
8. **Proper labeling** for image metadata

## Common Commands

```bash
# Build individual images
docker build -t mlflow-app -f mlflow/Dockerfile --build-arg MLFLOW_VERSION=2.3.2 mlflow/

# View running containers
docker ps

# View all containers
docker ps -a

# View images
docker images

# Clean up unused resources
docker system prune

# View logs for specific service
docker compose -f mlflow-docker-compose.yaml logs mlflow

# Scale services (if needed)
docker compose -f mlflow-docker-compose.yaml up -d --scale mlflow=2
```

## Troubleshooting

### Common Issues:

1. **Port conflicts**: Ensure ports 5001, 5432, 8090, 5433 are available
2. **Permission issues**: Check Docker daemon permissions
3. **Network issues**: Verify Docker network configuration
4. **Volume issues**: Check Docker volume permissions

### Debug Commands:

```bash
# Check service health
docker compose -f mlflow-docker-compose.yaml ps

# Execute commands in running container
docker compose -f mlflow-docker-compose.yaml exec mlflow bash

# View detailed logs
docker compose -f mlflow-docker-compose.yaml logs --tail=100 -f mlflow
```