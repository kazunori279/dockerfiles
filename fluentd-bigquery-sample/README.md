# Fluentd + Google BigQuery Getting Started Sample

This sample explains the following steps to set up a [Fluentd](http://www.fluentd.org/) + [Google BigQuery](https://cloud.google.com/bigquery/) integration in a [Docker](https://www.docker.com/) container that sends [nginx](http://nginx.org/en/) web server access log to the BigQuery in realtime.

- Sign Up for BigQuery
- Creating a dataset and table on Google BigQuery
- Creating an instance of Google Compute Engine (GCE)
- Run the sample Docker container

## Sign Up for BigQuery (If you have not done yet)

- If you have not signed up with BigQuery yet, follow the instruction in [Sign Up for BigQuery](https://cloud.google.com/bigquery/sign-up) page to create a project.
- To set up a billing, select the project, click `Billing & settings` and `Enable billing` button. Enter billing profile accordingly.
- Open [BigQuery Browser Tool](https://console.developers.google.com/)

![BigQuery Browser Tool](https://qiita-image-store.s3.amazonaws.com/0/38290/a2cb8721-b94e-20c4-2b78-1b0cc2fa0865.png)

- Execute the following sample query with the tool to check you can access BigQuery.

```
SELECT title FROM [publicdata:samples.wikipedia] WHERE REGEXP_MATCH(title, r'.*Query.*') LIMIT 100
```

## Creating a dataset and table on Google BigQuery

To create a dataset and table, you need to install `bq` command tool included in Cloud SDK. 

- Download and install the [Cloud SDK](https://cloud.google.com/sdk/).
- Authenticate your client by running:

```
$ gcloud auth login
```

- Create a dataset `bq-test` by executing the following command:

```
$ bq mk YOUR_PROJECT_ID:bq_test
```

- Create a file `schema.json` at your current directory with the following content:

```
[
  {
    "name": "agent",
    "type": "STRING"
  },
  {
    "name": "code",
    "type": "STRING"
  },
  {
    "name": "host",
    "type": "STRING"
  },
  {
    "name": "method",
    "type": "STRING"
  },
  {
    "name": "path",
    "type": "STRING"
  },
  {
    "name": "referer",
    "type": "STRING"
  },
  {
    "name": "size",
    "type": "INTEGER"
  },
  {
    "name": "user",
    "type": "STRING"
  },
  {
    "name": "time",
    "type": "INTEGER"
  }
]
```

- Execute the following command to create the table `access_log`.

```
$ bq mk -t YOUR_PROJECT_ID:bq_test.access_log ./schema.json
```

- Open the Browser Tool, select your project, `bq_test` dataset and `access_log` table. Confirm that the table has been created with the specified schema correctly.

## Creating a Google Compute Engine instance

- If you have not signed up with GCE yet, follow the instruction in [Sign Up for Google Compute Engine](https://cloud.google.com/compute/docs/signup) page to enable GCE.
- Run the following command to create a GCE instance named `bq-test`. This will take around 30 secs.

```
$ gcloud compute --project "YOUR_PROJECT_ID" instances create "bq-test" \
  --zone "us-central1-a" --machine-type "n1-standard-2" \
  --network "default" --maintenance-policy "MIGRATE" \
  --scopes "https://www.googleapis.com/auth/devstorage.read_only" \
  "https://www.googleapis.com/auth/bigquery" \
  --image container-vm-v20140929 --image-project google-containers
```

## Run the sample Docker container


