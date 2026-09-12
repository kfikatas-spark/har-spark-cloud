# Reproducing the WISDM HAR project

Run commands from this `supporting/` directory, preferably in a separate working copy. All training commands create new outputs. They are instructions, not commands automatically run when opening the submission.

## 1. What is included?

| Path | Purpose |
|---|---|
| `src/har_spark/` | Parsing → cleaning → window features → participant split → MLlib training/evaluation |
| `configs/` | The four measured experiment settings and a smoke configuration |
| `tests/` | Small synthetic-data tests; no raw dataset or cloud account required |
| `results/` | Four historical local experiments: metrics, confusion counts, saved Spark models and plots |
| `cloud/` | Spark entry script, saved cloud metrics, historical deployment notes and an optional rerun recipe |
| `scripts/generate_plots.py` | Regenerate charts from saved metrics; no training |
| `data/` | Dataset documentation and the historical download checksum; no bulk sensor data |

No web dashboard is required for this package. It is an offline analysis of recorded, labelled measurements—not a live watch streaming service.

## 2. Environment and quick verification

Python **3.11**, Java **17**, and PySpark **3.5.3** are the local baseline. Use a virtual environment; do not modify Debian's system Python. On another student's machine choose an equivalent user-owned venv path.

```bash
python3.11 -m venv "$HOME/.local/share/venvs/har-submission"
. "$HOME/.local/share/venvs/har-submission/bin/activate"
python -m pip install -e '.[dev,docs]'
python -m pytest -q
python -m har_spark.cli --help
```

`pyproject.toml` declares dependencies. Network access is needed to install packages; merely viewing the submitted PDFs/diagrams does not require it.

## 3. Obtain the data

Dataset: Gary M. Weiss, **WISDM Smartphone and Smartwatch Activity and Biometrics Dataset** (2019), UCI, DOI [10.24432/C5HK59](https://doi.org/10.24432/C5HK59), **CC BY 4.0**. Preserve attribution. See `data/DATASET-README.txt` for the original sensor/activity layout.

Download the complete dataset archive from the linked UCI record and save it as `data/raw/wisdm-dataset.zip`. The recorded checksum is for the **outer UCI bundle**, which contains `WISDM-dataset-description.pdf` and an inner `wisdm-dataset.zip`. Verify the outer bundle, then unpack the two layers into different directories:

```bash
(cd data/raw && sha256sum -c wisdm-dataset.zip.sha256)
unzip -n data/raw/wisdm-dataset.zip -d /tmp/har-wisdm-download
unzip -n /tmp/har-wisdm-download/wisdm-dataset.zip -d data/raw
```

Expected layout: `data/raw/wisdm-dataset/raw/watch/accel/*.txt` and `.../watch/gyro/*.txt`; phone experiments use the corresponding `phone/` directories. Stop and inspect a checksum mismatch rather than replacing the recorded checksum. A newer dataset release may differ.

## 4. Run the main local experiment

The global `--config` option goes **before** the subcommand:

```bash
python -m har_spark.cli --config configs/watch_fused_10s.yaml run
```

This reads both watch sensors, builds 10-second windows, splits participants, trains Logistic Regression and Random Forest, and writes a new timestamped `results/` directory. It does not overwrite the historical runs. Review `local[6]` and `4g` in the YAML if your machine has fewer resources; different resources affect timing.

To save features once and reuse them instead:

```bash
python -m har_spark.cli --config configs/watch_fused_10s.yaml prepare
python -m har_spark.cli --config configs/watch_fused_10s.yaml train
```

The prepared Parquet path must not already exist when running `prepare`. Do not delete historical outputs to bypass this protection; use a new path/configuration.

| Configuration | Historical run | Condition |
|---|---|---|
| `configs/local.yaml` | `20260827T053553Z` | Phone accelerometer, 5 s |
| `configs/fused.yaml` | `20260827T054008Z` | Phone accel + gyro, 5 s |
| `configs/fused_10s.yaml` | `20260827T054229Z` | Phone accel + gyro, 10 s |
| `configs/watch_fused_10s.yaml` | `20260827T054435Z` | Watch accel + gyro, 10 s |

Main local RF: accuracy **62.14%**, macro-F1 **61.90%**, 2,488 test windows. The 35/8/8 split counts people, not windows. Validation participants are held out, but the reported run uses fixed hyperparameters; it is not evidence of an automated validation search. Exact rerun values and timings can differ with runtime, partitioning and hardware.

## 5. Regenerate plots without training

Choose new, nonexistent output paths:

```bash
python scripts/generate_plots.py \
  --experiment 'Phone accel 5s=results/20260827T053553Z' \
  --experiment 'Phone fused 5s=results/20260827T054008Z' \
  --experiment 'Phone fused 10s=results/20260827T054229Z' \
  --experiment 'Watch fused 10s=results/20260827T054435Z' \
  --best results/20260827T054435Z --output /tmp/har-new-charts

```

The document-generation scripts are no longer included in this repository version. Their removal is a packaging change, not a change to the documents' provenance. Only the five final documents listed in the root README are included for the teacher. Editable document sources and the earlier scripts remain in Git history; their removal does not change the documents’ provenance.

## 6. Cloud evidence versus a new cloud run

`cloud/evidence/*_metrics.json` contains saved model metrics; `cloud/run_summary.json` identifies the historical successful batch. `cloud/deployment.md` is a human-readable historical note, not the original batch API response. No billing export is included. These files support the report's recorded experiment, not a claim about current resource state, actual charges, account ownership, or who personally performed the work.

Recorded cloud RF: accuracy **61.54%**, macro-F1 **61.09%**. This is distinct from the local result. The recorded batch took **44m 47s** and reports **13.38 DCU-h**, a resource-usage quantity, not a euro price. Shuffle storage usage is also not a network byte count.

For an independent cloud rerun, see `cloud/README.md`. A cloud rerun requires a Google identity, a billing-enabled project, appropriate permissions, and budget monitoring. Recheck current service availability and pricing first. Keep credentials and browser profiles private. No new cloud deployment is necessary simply to inspect this submission; whether a live demonstration or student-owned deployment is required is the examiner's decision.

To construct the dependency archive locally, after choosing a new destination:

```bash
(cd src && zip -r /tmp/har_spark_cloud_submission.zip har_spark -x '*/__pycache__/*' '*.pyc')
```

The archive must contain `har_spark/` at its root. Uploading it, assigning permissions, or submitting a batch are separate, potentially billable operations—not part of opening or verifying this package.
