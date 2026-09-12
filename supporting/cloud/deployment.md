# Evidence provenance

The following is a historical deployment note for 2026-08-27, NOT a fresh cloud-state check. The saved metrics and run summary are included; an original full batch API response and billing export were not found in the inspected cloud evidence directory. Resource/IAM/budget assertions below are historical notes, not independently reverified on 2026-09-11. Optional web-dashboard sections were omitted.

# Recorded Google Cloud deployment

## Scope and safeguards

- Dedicated project: `har-wisdm-student-2026` (`ACTIVE`, billing enabled).
- Region: `europe-west1`.
- Project-scoped €10 budget alert with 50%, 80% and 100% thresholds.
- Budget alerts notify; they are not an automatic spending cap.
- No downloadable service-account key was created.
- No credentials, tokens or browser data are stored in the project directory.

## Storage

- Bucket: `gs://har-wisdm-student-2026-data`.
- Regional location: `europe-west1`.
- Uniform bucket-level access: enabled.
- Public access prevention: enforced.
- Uploaded watch raw data: 405,960,139 bytes.
- Initial uploaded objects: 104 (102 sensor text files + 2 code artifacts).
- Successful result prefix: `gs://har-wisdm-student-2026-data/results/cloud-watch-10s-20260827-055742`.
- Result artifacts: 122 objects, 8,405,131 bytes.

## Identities and least privilege

### Spark runtime

`har-spark-runner@har-wisdm-student-2026.iam.gserviceaccount.com`

- `roles/dataproc.worker` on the dedicated project.
- `roles/logging.logWriter` on the dedicated project.
- `roles/storage.objectAdmin` on the one data bucket.
- The submitting user has `roles/iam.serviceAccountUser` on this service account.


## Managed Spark batch

| Field | Value |
|---|---|
| Batch ID | `har-watch-10s-20260827-055742` |
| Run ID | `cloud-watch-10s-20260827-055742` |
| State | **SUCCEEDED** |
| Created | 2026-08-27 05:58:00 UTC |
| Completed | 2026-08-27 06:42:47 UTC |
| End-to-end elapsed | **44 min 47 s** |
| Runtime | **2.3.39** (2.3 LTS family) |
| Driver | 4 cores, 9600 MiB |
| Executors | dynamic 2–4; 4 cores and 9600 MiB each |
| Usage | **13.38 DCU-hours** |
| Shuffle-storage time | **1,355.11 GB-hours** |
| Input | `raw/watch/*/*.txt` |
| Windows | 10 s, at least 150 samples |
| Seed | 42 |

Usage conversions use the API's final values:

- `48,174,200 milliDcuSeconds / 1000 / 3600 = 13.3817 DCU-h`;
- `4,878,400 shuffleStorageGbSeconds / 3600 = 1,355.1111 GB-h`.

## Cloud model results

| Model | Accuracy | Macro-F1 | Weighted-F1 | Train | Inference | Test windows |
|---|---:|---:|---:|---:|---:|---:|
| Logistic Regression | 49.96% | 48.48% | 48.63% | 265.23 s | 12.09 s | 2,488 |
| Random Forest | **61.54%** | **61.09%** | **61.20%** | 1,911.81 s | 77.81 s | 2,488 |

Split counts were identical for both models: 10,894 train, 2,528 validation and 2,488 test windows. Logistic Regression matches the corresponding local metrics exactly.

The local Random Forest result was 62.14% accuracy and 61.90% macro-F1. The cloud result is lower by 0.60 and 0.81 percentage points, respectively. Because data counts, split, seed, schema and hyperparameters match, a plausible explanation is sensitivity of distributed tree construction to runtime/partition/order details. This is an inference; the run does not by itself prove a single cause. Importantly, both environments support the same experimental conclusion: Random Forest clearly outperforms Logistic Regression.


## Reproduction evidence

- Cloud metrics are preserved under `cloud/evidence/`.
- The submitted Python dependency zip was 9,183 bytes.
- The code bucket contained `cloud_pipeline.py` and `har_spark_cloud_20260827.zip`.
- The batch used a unique output prefix and did not overwrite prior artifacts.
- Local code passed Ruff and all four tests after the deployment work.

