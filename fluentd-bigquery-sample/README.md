# Fluentd + Google BigQuery Getting Started Sample

In this sample, it explains the following steps to set up a [Fluentd](http://www.fluentd.org/) + [Google BigQuery](https://cloud.google.com/bigquery/) integration in a [Docker](https://www.docker.com/) container that sends [nginx](http://nginx.org/en/) web server access log to the BigQuery in realtime.

- Sign Up for BigQuery
- Creating a Docker-ready instance of Google Compute Engine
- Creating a dataset and table on Google BigQuery
- Run the sample Docker container

## Sign Up for BigQuery

If you have not signed up with BigQuery yet, follow the instruction in [Sign Up for BigQuery](https://cloud.google.com/bigquery/sign-up) page to create a project.

To set up a billing, select the project, click `Billing & settings` and `Enable billing` button. Enter billing profile accordingly.

## Creating a Google Compute Engine instance

```
gcloud compute --project "gcp-samples" instances create "kaz-docker0" \
  --zone "asia-east1-c" --machine-type "n1-standard-2" \
  --network "default" --maintenance-policy "MIGRATE" \
  --scopes "https://www.googleapis.com/auth/devstorage.read_only" \
  "https://www.googleapis.com/auth/bigquery" \
  --image container-vm-v20140929 --image-project google-containers
```

## Creating a dataset and table on Google BigQuery

## Run the sample Docker container



