# What is this?

A small repository to complete the kubernetes part of the data engineer onboarding.

**Note:** the instructions require Airbyte version 0.44.4, however this version is significantly older than the latest versions. This has impacts on compatibility with GKE, general stability and security. Knowing this, I choose to use the most recent version of Airbyte. 

This version requires a bit more resources due to its micro-services architecture, which is why the requests/limits are not exactly configured as instructed.

## Technical specifications

> **Define the workload: specify the appropriate Compute Engine VM types for this operation and determine the minimum operational resources (providing specific details).**

[Airbyte documentation](https://docs.airbyte.com/platform/using-airbyte/getting-started/oss-quickstart#:~:text=For%20best%20performance%2C%20run%20Airbyte,toward%20supporting%20lower%20resource%20environments.) recommends at least 4 CPUs and 8 GBs of RAM across the cluster to deploy Airbyte correctly.

We can go with two instances of **n4-standard-2** machine types, which fits Airbyte's requirements. This represents 126€/month.

The configuration is available [here](https://cloud.google.com/products/calculator?hl=fr&dl=CjhDaVE0WkRNeVpHRmtOaTA0WVRZM0xUUmpNMll0WVRZME9DMWxaak14T1RNM09UZzJOekVRQVE9PRAIGiRGMzhDMzZGNi1GNkY2LTQ5RUMtQTA3QS1CNDAxN0UyNERENzk=).

However, to deploy Airbyte properly on production, we also need the following GCP services:

- A bucket to store logs and sync states.
- A Cloud SQL database to store connection configurations and backups as well as easing the load on the Cluster.
- Google Secret Manager, as passwords in external databases are stored in unencrypted plain-text.
- [Optionnal] - A Static IP and LoadBalancer. if multiple users needs to access the Airbyte platform with auth, instead of port-forwarding.

The costs of Cloud Storage and Secret Manager are negligible, and the cost of a Cloud SQL Postgres machine is not very high, especially on small machines like **db-g1-small**. You can find a cost estimate (30€/month) [here](https://cloud.google.com/products/calculator?hl=fr&dl=CjhDaVJtT0dNMVpUWTVOQzFqTldRekxUUmhORGt0WVdJMU1pMWxOemhpWW1WbE4yTmxOVEVRQVE9PRAHGiRCNzZFMkU2Mi02RkU3LTQzRjYtOEZGNS1ERDcwQkM3MjVBOUE).

Overall, this configuration respects the 200€/month asked by the client.

**Warning:** this accounts for only one production environment, you must double this estimation if you want to set up a preproduction environment as well. 

## Maintenance

From [Best Practices for Upgrading Clusters](https://docs.cloud.google.com/kubernetes-engine/docs/best-practices/upgrading-clusters):

- Channels: Stable or Regular as it's the most adapted to production workloads
- Maintenance Windows: ideally at night (2AM to 6AM) but we need  more information about orchestration and expected refresh frequencies.
- Strategy: Surge Upgrades are the default in Autopilot clusters (which I think we're going to use here.)

## Setup

For this use case we are going to use Minikube to mimic a GKE Cluster. 

1. Install and Setup Minikube

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
```

Ensure that you have the docker engine running and start Minikube in a configuration that replicates the **n4-standard-2** we elected earlier (if you can):

```bash
minikube start --driver=docker --cpus 4 --memory 8192
```

You also want to [install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) and [Helm](https://helm.sh/docs/intro/install) on your machine, so you can interact with the cluster and use Charts to deploy Airbyte. 

## Airbyte Deployment

1. **Add and update the Airbyte Helm repository:**

```bash
helm repo add airbyte https://airbytehq.github.io/helm-charts
helm repo update
```

2. **Create a namespace for Airbyte:**

```bash
kubectl create namespace airbyte-ns
```

3. **Pull the ```values.yaml``` file into the repository**

```bash
mkdir airbyte
helm show values airbyte/airbyte > ./airbyte/values.yaml
```

4. **Adjust the ```values.yaml``` file to fit your use case**

- For the temporal and server pods, adjust the resources (memory and cpu). You can consider increasing the resources, because in most recents versions of Airbyte there is no webapp anymore but higher resources requirements.

- For the server pod, add a snippet to change it to a loadbalancer service.

PS: if your computer does not have enough resources consider increasing the ```initialDelaySeconds``` of the server liveness and readyness probes to avoid having your pod repeatedly spawnkilled.

6. **Install Airbyte:**

```bash
helm install airbyte airbyte/airbyte --namespace airbyte-ns --values ./airbyte/values.yaml
```

7. **Expose Airbyte to your localhost**

Use the Minikube tunnel feature with this command:

```bash
minikube tunnel
```

Keep the terminal in which you ran the command open, else the tunnel will close.

8. **Use Lens to monitor your cluster**

If you want to, you can use [Lens](https://lenshq.io/) to visualy monitor the cluster.

Install Lens, then copy cluster configuration with:

```bash
kubectl config view --flatten
```

And then paste it into Lens, you should see your cluster appear.
