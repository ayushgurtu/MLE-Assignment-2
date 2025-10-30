# MLE Assignment 2
 
 --- IGNORE ---

This repository contains an end-to-end machine learning pipeline for loan default prediction. It demonstrates a production-like MLOps workflow using the medallion architecture (Bronze → Silver → Gold) orchestrated by Apache Airflow. The pipeline performs data ingestion, feature engineering, model training, batch inference, monitoring, visualization, and action assessment.


## Quick start

Start the platform and DAGs:

```powershell
docker-compose up --build
```

Open the Airflow UI at http://localhost:8080 (user: `admin`, pass: `admin`). Enable the DAG named `dag` and monitor runs.

To stop:

```powershell
docker-compose down
```

## What this project implements

- Medallion architecture: Bronze (raw) → Silver (clean) → Gold (ML-ready)
- Two competing models (a Logistic Regression baseline and a Gradient Boosting model)
- Temporal validation with a 4-window split (Train / Validation / Test / OOT)
- Monthly batch inference and monitoring via Airflow
- Metric-based monitoring with dual-threshold governance (business vs DS thresholds)
- Automatic visualizations and decision/action reports


## Key concepts

- Temporal (4-window) validation: 12 months train, 3 months validation, 2 months test, 1 month OOT. Requires 18 months minimum to run training.
- MOB (Month-on-Book) = 6 labeling: predictions are made at MOB=0; labels are observed at MOB=6. Monitoring therefore joins predictions from 6 months ago with current labels.
- ShortCircuitOperators in the DAG prevent running steps when prerequisites are missing (e.g., models missing → skip inference).

## Monitoring & governance

This project uses a dual-threshold, 3-level action framework:

- P0 (business critical): ROC-AUC — triggers immediate retrain if below business threshold
- P1 (business important): Accuracy — raises investigation if below DS threshold
- P2 (DS operational): F1-score — monitoring only

Actions:

- `monitor` — everything nominal
- `active_monitoring` — investigate and watch more frequently
- `retrain` — schedule immediate retrain

Action decisions are produced by `scripts/evaluate_monitoring_action.py` and written to `scripts/outputs/actions/`.

## Model artifacts

Each trained model produces a folder in `model_store/` with the following files:

- `model.pkl` — serialized model
- `preprocessing.pkl` — transformers (imputers/scalers/encoders)
- `metadata.json` — training metadata and metrics
- `features.json` — feature list
- `feature_importance.csv` — (if available)

## Troubleshooting (common issues)

- Tasks skipped: check DAG logs to see which ShortCircuitOperator fired. This is expected for early snapshot dates (insufficient data) or before models exist.
- Airflow not available: ensure containers are running (`docker-compose ps`) and that port 8080 is free.
- Container initialization: inspect `docker-compose logs airflow-init` for errors.

If you need help reproducing a specific failure, include the Airflow task log and the snapshot date you were running.

## Contribution & development notes

- Keep documentation updated when changing pipeline behavior.
- Add tests for data transforms when adding features.
- Follow naming conventions: `{layer}_{table}.py` for data scripts; `model_{id}_{action}.py` for ML tasks.

## License

Educational project for SMU MiTB CS611 Machine Learning Engineering (MLE).

Licensed under MIT License. See `LICENSE` file for details.
