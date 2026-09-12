# Optional independent Spark cloud reproduction

Historical recipe from the 2026-08-27 experiment. Recheck current runtime availability, permissions, and pricing before a new deployment. None of these commands was executed during submission cleanup. An account is NOT needed to read the included results.

The recorded deployment uses:

- project: `har-wisdm-student-2026`;
- region: `europe-west1`;
- private bucket: `gs://har-wisdm-student-2026-data`;
- keyless runtime service account: `har-spark-runner@har-wisdm-student-2026.iam.gserviceaccount.com`;
- Managed Service for Apache Spark runtime: `2.3` LTS;
- main batch: `har-watch-10s-20260827-055742`.

No password, API token, browser cookie or downloadable service-account key belongs in this directory.

## Student account prerequisites

For an independent deployment, the student needs a Google identity, a new project with billing, and `gcloud` CLI. Use a dedicated project rather than a personal production project. Configure a budget alert first. An alert sends notifications but does not stop resources automatically.

Authenticate interactively on the student's own computer:

```bash
gcloud auth login
gcloud auth list
gcloud config set project STUDENT_PROJECT_ID
```

The student should never receive somebody else's `~/.config/gcloud` directory or browser profile.

## Required APIs

```bash
gcloud services enable \
  dataproc.googleapis.com \
  storage.googleapis.com \
  iam.googleapis.com \
  logging.googleapis.com \
  compute.googleapis.com
```

## Resource pattern

Choose globally unique names and replace every placeholder:

```bash
PROJECT_ID='STUDENT_PROJECT_ID'
REGION='europe-west1'
BUCKET='STUDENT_UNIQUE_BUCKET'
SERVICE_ACCOUNT='har-spark-runner'
```

Create a regional, private bucket with uniform access and public-access prevention. Create a service account **without a key file**. The runtime identity needs Dataproc worker, logging writer and object access to this bucket. The submitting student needs permission to act as that service account.

IAM commands are intentionally not offered as a blind copy-paste script: the student must verify the current project and identity first, and role assignment affects cloud security. The exact effective permissions of the recorded deployment are documented in `deployment.md`.

## Upload layout

```text
gs://BUCKET/
├── code/
│   ├── cloud_pipeline.py
│   └── har_spark_cloud_YYYYMMDD.zip
├── raw/watch/
│   ├── accel/*.txt
│   └── gyro/*.txt
└── results/RUN_ID/
```

The Python zip must contain the `har_spark/` package at its root. Upload only the watch accelerometer and gyroscope files for the main experiment.

## Batch submission pattern

After verifying project, bucket objects, region, service account and billing:

```bash
gcloud dataproc batches submit pyspark \
  gs://BUCKET/code/cloud_pipeline.py \
  --project=PROJECT_ID \
  --region=europe-west1 \
  --batch=UNIQUE_BATCH_ID \
  --version=2.3 \
  --service-account=SERVICE_ACCOUNT_EMAIL \
  --staging-bucket=BUCKET \
  --py-files=gs://BUCKET/code/har_spark_cloud_YYYYMMDD.zip \
  --labels=purpose=university-har,experiment=watch-fused-10s \
  --properties=spark.dynamicAllocation.initialExecutors=2,spark.dynamicAllocation.minExecutors=2,spark.dynamicAllocation.maxExecutors=4 \
  --async \
  -- \
  --raw-path='gs://BUCKET/raw/watch/*/*.txt' \
  --output-path=gs://BUCKET/results/UNIQUE_RUN_ID \
  --device=watch \
  --window-seconds=10 \
  --minimum-samples=150 \
  --seed=42
```

Note that `--staging-bucket` takes a bare bucket name, while code/data arguments use `gs://` URIs.

## Read-only monitoring

```bash
gcloud dataproc batches describe UNIQUE_BATCH_ID \
  --project=PROJECT_ID \
  --region=europe-west1 \
  --format='yaml(state,stateMessage,stateTime,runtimeInfo)'
```

Results are written under the unique output prefix. Do not rerun into the same prefix and do not delete failed-run evidence before diagnosing it.

## Cost and post-grading checklist

1. Confirm the Spark batch reached a terminal state.
2. Inspect Billing reports and budget notifications.
3. Download any results needed for submission.
4. Decide whether to retain or remove the bucket and project.
5. Remove temporary student IAM access after grading.

Stopping or deleting cloud resources is a separate destructive action. Verify the exact project and obtain the project owner's approval before doing it.

