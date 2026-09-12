# Configuration folder guide

These YAML files supply settings to the **local command-line workflow**: `cli.py` selects a file, `config.py` loads it, and `pipeline.py` passes its settings to the Spark, data-processing and model functions.

| File | Experiment | Where used |
|---|---|---|
| [local.yaml](local.yaml) | Phone accelerometer, 5-second windows | Default CLI configuration when `--config` is omitted |
| [fused.yaml](fused.yaml) | Both phone sensors, 5-second windows | Select with `--config configs/fused.yaml` |
| [fused_10s.yaml](fused_10s.yaml) | Both phone sensors, 10-second windows | Select with `--config configs/fused_10s.yaml` |
| [watch_fused_10s.yaml](watch_fused_10s.yaml) | Both watch sensors, 10-second windows | Main local experiment; example below |
| [smoke.yaml](smoke.yaml) | One input file and tiny models | Development check via `--config configs/smoke.yaml`; not an accuracy benchmark and may lack participants for all splits |

After [setup](../README.md), run from `supporting/`:
```bash
python -m har_spark.cli --config configs/watch_fused_10s.yaml run
```
This example starts processing and training and creates new results; it is not a read-only check. `prepare` writes features; `train` reads prepared features; `run` processes raw data and trains.

**Cloud is separate:** [cloud_pipeline.py](../cloud/cloud_pipeline.py) takes command-line arguments, not these YAML files. See the [cloud guide](../cloud/README.md).
