# What is this?

A small repository to complete the kubernetes part of the data engineer onboarding.

## Technical specifications

> **Define the workload: specify the appropriate Compute Engine VM types for this operation and determine the minimum operational resources (providing specific details).**

[Airbyte documentation](https://docs.airbyte.com/platform/using-airbyte/getting-started/oss-quickstart#:~:text=For%20best%20performance%2C%20run%20Airbyte,toward%20supporting%20lower%20resource%20environments.) recommends at least 4 CPUs and 8 GBs of RAM across the cluster to deploy Airbyte correctly.

We can go with two instances of **n4-standard-2** machine types, which fits Airbyte's requirements. This represents 126€/month.

The configuration is available [here](https://cloud.google.com/products/calculator?hl=fr&dl=CjhDaVE0WkRNeVpHRmtOaTA0WVRZM0xUUmpNMll0WVRZME9DMWxaak14T1RNM09UZzJOekVRQVE9PRAIGiRGMzhDMzZGNi1GNkY2LTQ5RUMtQTA3QS1CNDAxN0UyNERENzk=).

However, to deploy Airbyte properly on production, we also need the following GCP services:

- A bucket to store logs and sync states.
- A Cloud SQL database to store connection configurations and backups as well as easing the load on the Cluster.
- Google Secret Manager, as passwords in external databases are stored in unencrypted plain-text.
- [Optionnal] - A dedicated external IP if multiple users needs to access the Airbyte platform with auth, instead of port-forwarding.

The costs of Cloud Storage and Secret Manager are negligible, and the cost of a Cloud SQL Postgres machine is not very high, especially on small machines like **db-g1-small**. You can find a cost estimate (30€/month) [here](https://cloud.google.com/products/calculator?hl=fr&dl=CjhDaVJtT0dNMVpUWTVOQzFqTldRekxUUmhORGt0WVdJMU1pMWxOemhpWW1WbE4yTmxOVEVRQVE9PRAHGiRCNzZFMkU2Mi02RkU3LTQzRjYtOEZGNS1ERDcwQkM3MjVBOUE).