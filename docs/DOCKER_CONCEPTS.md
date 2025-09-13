# Docker Concepts Explained

## Multi-Stage Builds

Multi-stage builds allow you to use multiple FROM statements in your Dockerfile. Each FROM instruction can use a different base, and each begins a new stage of the build.

### Benefits:
- **Smaller final images**: Only runtime dependencies are included
- **Better security**: Fewer packages mean smaller attack surface
- **Cleaner separation**: Build tools separate from runtime environment

### Example:
```dockerfile
# Build stage
FROM python:3.9-slim AS compile-image
RUN pip install dependencies...

# Runtime stage  
FROM python:3.9-alpine AS runtime-image
COPY --from=compile-image /opt/venv /opt/venv
```

## CMD vs ENTRYPOINT

### ENTRYPOINT
- Makes container executable
- Cannot be overridden at runtime
- Best for containers that should always run the same command

### CMD
- Provides default arguments
- Can be overridden at runtime
- Best for providing default behavior that users might want to change

### Combined Usage:
```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--help"]
```

## Docker Compose Features

### Health Checks
Monitor container health and restart if needed:
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
```

### Service Dependencies
Control startup order:
```yaml
depends_on:
  database:
    condition: service_healthy
```

### Networks
Isolate services and enable communication:
```yaml
networks:
  app-network:
    driver: bridge
```

### Volumes
Persist data beyond container lifecycle:
```yaml
volumes:
  app-data:
    driver: local
```