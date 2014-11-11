# Fluentd + Google BigQuery Getting Started Sample

This sample explains how to set up a [Fluentd](http://www.fluentd.org/) + [Google BigQuery](https://cloud.google.com/bigquery/) integration in a [Docker](https://www.docker.com/) container that sends [nginx](http://nginx.org/en/) web server access log to the BigQuery in real time. The whole process may take only 20 - 30 minutes with the following steps:

- Sign Up for BigQuery
- Creating a dataset and table on Google BigQuery
- Creating an instance of Google Compute Engine (GCE)
- Run nginx + Fluentd with Docker container
- Execute BigQuery query

## Sign Up for BigQuery

(You can skip this section if you have done before)

- If you have not signed up with BigQuery yet, follow the instruction in [Sign Up for BigQuery](https://cloud.google.com/bigquery/sign-up) page to create a project.
- To set up a billing, select the project, click `Billing & settings` and `Enable billing` button. Enter billing profile accordingly.
- Open [BigQuery Browser Tool](https://console.developers.google.com/)
- Click `COMPOSE QUERY` button at top left and execute the following sample query with the tool to check you can access BigQuery.

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

- Create a file [schema.json](https://github.com/kazunori279/dockerfiles/blob/master/fluentd-bigquery-sample/schema.json) at your current directory with the following content:

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

- Reload the BigQuery Browser Tool page, select your project, `bq_test` dataset and `access_log` table. Confirm that the table has been created with the specified schema correctly.

## Creating a Google Compute Engine instance

- If you have not signed up with GCE yet, follow the instruction in [Sign Up for Google Compute Engine](https://cloud.google.com/compute/docs/signup) page to enable GCE.
- Run the follwoing command to set default project to your project id.

```
$ gcloud config set project YOUR_PROJECT_ID
```

- Run the following command to create a GCE instance named `bq-test`. This will take around 30 secs.

```
$ gcloud compute instances create "bq-test" \
  --zone "us-central1-a" --machine-type "n1-standard-1" \
  --network "default" --maintenance-policy "MIGRATE" \
  --scopes "https://www.googleapis.com/auth/devstorage.read_only" \
  "https://www.googleapis.com/auth/bigquery" \
  --image container-vm-v20140929 --image-project google-containers
```

## Run nginx + Fluentd with Docker container

- Enter the following command to log in to the GCE instance.

``` 
$ gcloud compute ssh bq-test --zone=us-central1-a
```

- In the GCE instance, run the following command (replace `YOUR_PROJECT_ID` with your project id). This will start downloading a Docker image `kazunori279/fluentd-bigquery-sample` which contains nginx web server with Fluentd.

```
$ sudo docker run -e GCP_PROJECT="YOUR_PROJECT_ID" -p 80:80 -t -i -d kazunori279/fluentd-bigquery-sample
```

- Open [Google Developers Console](https://console.developers.google.com/project) on a browser, choose your project and select `Compute` - `Compute Engine` - `VM instances`.

- Find `bq-test` GCE instance and click it's external IP link. On the dialog, select `Allow HTTP traffic` and click `Apply` to add the firewall rule. There will be an Activities dialow shown on the display with a message `Updating instance tags for "bq-test"`.

- After updating instance tags, click the external IP link again to access nginx server on the instance. It will show a blank web page titled "Welcome to nginx!". Click reload button several times.

## Execute BigQuery query

- Open BigQuery Browser Tool, click `COMPOSE QUERY` and execute the following query. You will see the requests from browser are recorded on access_log table (it may take a few minutes to receive the log for the first time).

```
SELECT * FROM [bq_test.access_log] LIMIT 1000
```

That's it! You've just confirmed the nginx log are collected by Fluentd, imported to BigQuery and shown on the Browser Tool. You may use Apache Bench tool or etc to hit the web page with more traffic to see how Fluentd + BigQuery can handle high volume logs in real time. It can support up to 10K rows/sec by default (and you can extend it to 100K rows/sec by requesting).

## Inside Dockerfile and td-agent.conf

If you take a look at the [Dockerfile](https://github.com/kazunori279/dockerfiles/blob/master/fluentd-bigquery-sample/Dockerfile), you can learn how the Docker container has been configured. After preparing an Ubuntu image, it installs Fluentd, nginx and the bigquery plugin.

```
FROM ubuntu:12.04
MAINTAINER kazunori279-at-gmail.com

# environment
ENV DEBIAN_FRONTEND noninteractive
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list

# update, curl, sudo
RUN apt-get update && apt-get -y upgrade
RUN apt-get -y install curl 
RUN apt-get install sudo

# fluentd
RUN curl -O http://packages.treasure-data.com/debian/RPM-GPG-KEY-td-agent && apt-key add RPM-GPG-KEY-td-agent && rm RPM-GPG-KEY-td-agent
RUN curl -L http://toolbelt.treasuredata.com/sh/install-ubuntu-precise-td-agent2.sh | sh 
ADD td-agent.conf /etc/td-agent/td-agent.conf

# nginx
RUN apt-get install -y nginx
ADD nginx.conf /etc/nginx/nginx.conf

# fluent-plugin-bigquery
RUN /usr/sbin/td-agent-gem install fluent-plugin-bigquery --no-ri --no-rdoc -V

# start fluentd and nginx
EXPOSE 80
ENTRYPOINT /etc/init.d/td-agent restart && /etc/init.d/nginx start && /bin/bash
```

In the [td-agent.conf](https://github.com/kazunori279/dockerfiles/blob/master/fluentd-bigquery-sample/td-agent.conf) file, you can see how to configure it to forward Fluentd logs to BigQuery plugin. It's as simple as the following:

```
<match nginx.access>
  type bigquery
  auth_method compute_engine

  project "#{ENV['GCP_PROJECT']}"
  dataset bq_test
  tables access_log

  time_format %s
  time_field time
  fetch_schema true
  field_integer time
</match>
```

Since you are running the GCE instance within the same GCP project of BigQuery dataset, you don't have to copy any private key file to the GCE instance for OAuth2 authentication. ` "#{ENV['GCP_PROJECT']}"` refers to your project id passed through the environment variable.

## Cleaning Up

- Execute the following command to delete GCE instance.

```
gcloud compute instances delete bq-test --zone=us-central1-a
```

- On BigQuery Browser Tool, click the drop down menu of `bq_test` dataset and select `Delete dataset`


