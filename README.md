# First GCP Data Pipeline

My first end-to-end pipeline on Google Cloud: generate employee records, land them
in Cloud Storage, transform them in Cloud Data Fusion, and load the result into
BigQuery, with Airflow on Cloud Composer driving the whole thing on a daily
schedule.

[![GCP](https://img.shields.io/badge/Google_Cloud-4285F4?logo=googlecloud&logoColor=white)](https://cloud.google.com/)
[![Airflow](https://img.shields.io/badge/Airflow-017CEE?logo=apacheairflow&logoColor=white)](https://airflow.apache.org/)
[![BigQuery](https://img.shields.io/badge/BigQuery-669DF6?logo=googlebigquery&logoColor=white)](https://cloud.google.com/bigquery)
[![License: MIT](https://img.shields.io/badge/License-MIT-22C55E.svg)](LICENSE)

> **Related repo.** [`etl-pipeline-datafusion-airflow`](https://github.com/rk-chavali/etl-pipeline-datafusion-airflow)
> is **the canonical version** of this pipeline. It carries the architecture
> diagram and the fuller writeup. This repo is kept as the original first pass,
> which is why the Data Fusion pipeline name still has a typo in it.

## Flow

```
extract.py  ->  Cloud Storage  ->  Cloud Data Fusion  ->  BigQuery
 Faker-generated   employee_data.csv    mask + encode        employee table
 employee records                       sensitive fields
        \                                     /
         \______ Airflow DAG on Composer ____/
                   schedule: @daily
```

## The DAG

`dag.py` defines `employee_data`, running daily with no catchup:

| Task | Operator | Does |
|------|----------|------|
| `extract_data` | `BashOperator` | Runs `extract.py` from the Composer DAGs bucket |
| `start_pipeline` | `CloudDataFusionStartPipelineOperator` | Kicks off the Data Fusion pipeline in `us-central1` |

`extract_data` runs first, then `start_pipeline`.

## Data generation

`extract.py` uses Faker to synthesise 10,000 employee records with name, job
title, department, email, address, phone, salary, and a password field, writes
them to CSV, and uploads to Cloud Storage.

The password field exists so the Data Fusion pipeline has something meaningful to
mask. **All of it is synthetic.** No real personal data is involved.

## Deploy

1. Upload `extract.py` to `gs://<composer-bucket>/dags/scripts/`
2. Upload `dag.py` to `gs://<composer-bucket>/dags/`
3. Create the Data Fusion pipeline and match its name to the one in `dag.py`
4. Set the alert email in `default_args` to your own

## Notes

The pipeline name referenced in the DAG is `rk-fisrt-project`, typo included, so
it matches what was actually deployed. The canonical repo linked above is the
cleaner take.

## License

MIT, see [LICENSE](LICENSE).
