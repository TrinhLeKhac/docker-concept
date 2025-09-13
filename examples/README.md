# Examples

## MLflow Smoke Test

The `smoke_test.py` script demonstrates how to:
- Connect to MLflow tracking server
- Train a machine learning model
- Log parameters, metrics, and artifacts
- Register models in MLflow Model Registry

### Usage:
```bash
# Start MLflow first
docker compose -f ../mlflow-docker-compose.yaml up -d

# Run the smoke test
python smoke_test.py [alpha] [l1_ratio]

# Example with custom parameters
python smoke_test.py 0.3 0.7
```

### What it does:
1. Downloads wine quality dataset from UCI ML repository
2. Splits data into train/test sets
3. Trains an ElasticNet regression model
4. Evaluates model performance (RMSE, MAE, R²)
5. Logs everything to MLflow tracking server
6. Registers model if using remote backend

### Expected Output:
```
Elasticnet model (alpha=0.500000, l1_ratio=0.500000):
  RMSE: 0.7931640229276851
  MAE: 0.6271946374319586
  R2: 0.10862644997792614
```

Check MLflow UI at http://localhost:5001 to see the logged experiment.