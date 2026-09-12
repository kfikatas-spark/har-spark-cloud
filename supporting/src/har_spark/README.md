# HAR Spark source folder guide

This package is used by the local CLI. Selected functions are also imported by the separate [cloud entry script](../../cloud/cloud_pipeline.py).

| File | Purpose | Used by / where |
|---|---|---|
| [cli.py](cli.py) | Select configuration and command | Local entry: `python -m har_spark.cli` or installed `har-spark` command |
| [config.py](config.py) | Load and validate YAML | `cli.py`; supplies settings to `pipeline.py` and `spark.py` |
| [spark.py](spark.py) | Create the Spark session | Local `pipeline.py`; cloud creates its own session |
| [ingest.py](ingest.py) | Parse and filter raw measurements | Local `pipeline.py` and `cloud/cloud_pipeline.py` |
| [features.py](features.py) | Window features, sensor fusion and participant splits | Local pipeline/model modules and the cloud script |
| [model.py](model.py) | Train, evaluate and save models/metrics | Local `pipeline.py`; cloud reuses its confusion/macro-metric helper, not its training coordinator |
| [pipeline.py](pipeline.py) | Coordinate preparation, training and output | Local `cli.py`; cloud has separate orchestration |
| [__init__.py](__init__.py) | Package initialization | Python when importing `har_spark` |

**Local flow:** CLI → YAML settings → pipeline → ingestion/features → model training/evaluation → saved results.

The [tests](../../tests/) directly check configuration loading, window features, participant splits, sensor fusion and model metrics. See [setup](../../README.md) and the fuller [code walkthrough](../../../CODE-WALKTHROUGH.md).
