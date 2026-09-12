# Project presentation — WISDM HAR with PySpark

In this presentation, I explain how the project processes sensor measurements, compares two activity-recognition models, and supports local and cloud execution.

## The project in one sentence

The application reads labelled phone or smartwatch sensor measurements, builds time-window features, separates participants into data splits, trains and evaluates activity-recognition models, and saves the results.

## Processing flow

**Read measurements → Build window features → Combine sensors → Split by participant → Train and evaluate → Save results**

The main Python package is [`supporting/src/har_spark/`](supporting/src/har_spark/).

| File | What it does |
|---|---|
| [`ingest.py`](supporting/src/har_spark/ingest.py) | Reads the original WISDM sensor lines, parses their fields, and selects valid records for the requested device and sensors. |
| [`features.py`](supporting/src/har_spark/features.py) | Calculates features for time windows, combines sensor features, and assigns participants to training, validation, and test splits. |
| [`model.py`](supporting/src/har_spark/model.py) | Builds the learning pipelines, trains Logistic Regression and Random Forest, and calculates evaluation metrics and confusion counts. |
| [`pipeline.py`](supporting/src/har_spark/pipeline.py) | Connects the processing stages into a complete run and saves the experimental outputs. |
| [`config.py`](supporting/src/har_spark/config.py) | Loads the settings that define an experiment, such as input paths, selected sensors, and window size. |
| [`spark.py`](supporting/src/har_spark/spark.py) | Creates the Spark session using the configured execution settings. |
| [`cli.py`](supporting/src/har_spark/cli.py) | Provides the command-line entry point, including `prepare`, `train`, and `run`. |

## Execution and experimental scope

- This repository contains executable project code, not only documents or screenshots.
- The small [real-data example](examples/wisdm-sample/) demonstrates the input format. Full experiments require the complete WISDM dataset.
- The documented validation split does not imply that automated hyperparameter tuning was performed.
- The saved results describe historical experiments.
- The cloud entry script and deployment instructions are in [`supporting/cloud/`](supporting/cloud/); the module table above describes the main local package.
- For installation, dataset download, and execution commands, use the [reproduction guide](supporting/README.md).
