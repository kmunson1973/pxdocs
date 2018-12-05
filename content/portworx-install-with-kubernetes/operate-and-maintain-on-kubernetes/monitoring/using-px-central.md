---
title: Using PX Central
keywords: portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk, monitoring, prometheus, alertmanager, servicemonitor, grafana, lighthouse, px-central, px-kvdb
description: How to deploy and control Portworx using PX-Central
---

## Overview
With PX-Enterprise 2.0, Portworx just released PX-Central to simplify management, monitoring and metadata services for one or more Portworx Clusters on Kubernetes. Using this single pane of glass, enterprises can easily manage the state of their hybrid- and multi-cloud Kubernetes applications with embedded monitoring and metrics directly in the Portworx user interface. 

Portworx Clusters needs to be updated to PX-Enterprise 2.0 before using PX-Central. In the first release, PX-Central includes the following components:

* [Lighthouse](/reference/lighthouse) - Quick birds-eye view of your multi-cluster Portworx deployment to ensure things are running smoothly.
![LHMultiCluster](/img/LH- Multi-cluster.png)

* [Prometheus + Grafana + Alertmanager] (/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/monitoring/monitoring-px-prometheusandgrafana.1/) - Most effective pro-active monitoring solution to analyze change in key metrics over a period of time and alert on anomalies.
![grafanaclusterdashboard](/img/Grafana - cluster dashboard.png)

* **PX-kvdb**: Built-in key-value database for storing metadata belonging to Portworx services

## Deployment Scenarios

### Using PX-Central with a single Portworx Cluster

In this scenario, all the components are running alongside PX-Enterprise in the same cluster. The deployment links the clusterID for the Portworx cluster to the individual services within PX-Central. Use this scenario if you are installing for the first time, cluster size is small and you don’t plan to setup multiple clusters initially.

![singlelclustermodel](/img/PXCSingleCluster.png)

### Using PX-Central (dedicated mode) with multiple PX-Cluster

In this scenario, all the components are running on a dedicated set of nodes outside of the PX-Enterprise clusters. The deployment links the clusterID for the Portworx cluster(s) to the individual services within PX-Central. Use this scenario if you are installing larger clusters, plan to scale deployment by adding additional clusters in future cluster size is small and want centralized control.

![multiclustermodeded](/img/PXCMultiClusterDedicated.png)

### Using PX-Central (shared mode) with multiple PX-Clusters

In this scenario, all the components are running on one of the PX-Enterprise clusters and connected to others. The deployment links the clusterID for the Portworx cluster(s) to the individual services within PX-Central. Use this scenario only if you have a larger first cluster and are not sure when you will add additional clusters.

![multiclustermodesha](/img/PXCMultiClusterShared.png)

## Setting up PX-Central

PX-Central can be setup for any of the above deployment scenarios by following a few steps. Note that Portworx will also be installed as a part of this process for new installations.

### Pre-requisites

* Have a K8S cluster running
* Have the Prometheus Operator [spec](/samples/k8s/portworx-pxc-operator.yaml) installed.  

    ```text
    kubectl apply -f portworx-pxc-operator.yaml
    ```

### Single Cluster Installation for new PX installations

Once the prerequisites are fulfilled, please follow the instructions below.

1. Create a secret using this [template](/samples/k8s/portworx-pxc-alertmanager.yaml). Replace the values corresponding to your email settings

    ```text
    kubectl create secret generic alertmanager-portworx --from-file=portworx-pxc-alertmanager.yaml -n kube-system
    ```
2. Download the Portworx setup [spec](/samples/k8s/portworx-pxc-singlecluster.yaml) which includes PX-Central and PX-Enterprise.

3. Replace <unique id> in the command given below:

    ```text
    CLUSTER_ID=<unique id> envsubst < portworx-pxc-singlecluster.yaml | kubectl apply -f -
    ```

This should setup the following:

* a Portworx Cluster (including px-kvdb cluster)
* 1 Grafana Instance
* 1 Prometheus Instance
* 1 Lighthouse Instance
* 1 AlertManager instance

Visit Lighthouse on `http://<master_ip>:32678`, login with admin/Password1 and the cluster should be visible.

You can also view Prometheus on `http://<master_ip>:30900`, Grafana on `http://<master_ip>:30950`.

{{<info>}}The port values above are true if using NodePort and will be different for LoadBalancer.{{</info>}}

* Lighthouse GUI can now be used to manage the Portworx cluster. 
* Grafana and Prometheus have shortcuts from Lighthouse for easy access.
* Prometheus will scrape the nodes for metrics.
* AlertManager will notify using a set of default rules 
* Grafana will use Prometheus as it’s datasource and includes pre-built dashboards for Portworx cluster, node and volume monitoring.
* Grafana will also have pre-built etcd dashboard (from community) to monitor metadata cluster

###  Multi-Cluster Installation for new PX installations

If your cluster has more than 20 nodes or is resource intensive we recommend using this installation to create a dedicated monitoring cluster, and let this cluster monitor the others. If not consider using the single cluster installation (above). The steps below will help you setup PX-Central and first PX-Enterprise Cluster.

1. Create a secret using this [template](/samples/k8s/portworx-pxc-alertmanager.yaml). Replace the values corresponding to your email settings. 											

    ```text
    kubectl create secret generic alertmanager-portworx --from-file=portworx-pxc-alertmanager.yaml -n kube-system
    ```

2. Create a secret using this [template](/samples/k8s/portworx-pxc-prometheus-additional.yaml). Replace the values corresponding to your other K8S clusters.											

    ```text
    kubectl create secret generic additional-scrape-configs --from-file=portworx-pxc-prometheus-additional.yaml -n kube-system
    ```

3. Download the multi-cluster [spec](/samples/k8s/portworx-pxc-multicluster.yaml).

4. Replace <unique id> in the command given below with the desired clustername.

    ```text
    CLUSTER_ID=<unique id> envsubst < portworx-pxc-multicluster.yaml | kubectl apply -f -
    ```

You will now have the following:

* A Portworx Cluster
* 1 Grafana Instance
* 1 Prometheus Instance
* 1 Lighthouse Instance
* 1 AlertManager instance

Visit Lighthouse on `http://<master_ip>:<32768>`, login with admin/Password1 and the cluster should be visible. Note that the port values above are true if using NodePort and will be different for LoadBalancer.

{{<info>}}The port values above are true if using NodePort and will be different for LoadBalancer.{{</info>}}

* Prometheus will scrape the nodes
* AlertManager will report issues based on our given rules
* Grafana will use Prometheus as it’s datasource and has pre-baked dashboards

For subsequent clusters, do not use the multi-cluster spec above. Use the spec generator to create a new cluster (only) installation spec. Then in order to have Lighthouse manage this new cluster, add it to Lighthouse with the add cluster option. To enable Prometheus monitoring on this new cluster, update the prometheus-additional.yaml by adding external ip and nodeport for the cluster.

### Verifying the setup

Once all the pods are up and running, you can verify different components of PX-Central as follows.

*PX-kvdb*

Quick way to Identify nodes running px-kvdb is by running the pxctl command below on one of the cluster nodes

```text
pxctl sv kvdb members
```

This should display output like:

```text
Kvdb Cluster Members:
ID                    PEER URLs            CLIENT URLs            LEADER    HEALTHY    DBSIZE
91b988c5-7a6e-4d3d-9ca1-a54a388a0741    [http://XX.XX.XX.XX:9018]    [http://XX.XX.XX.XX:9019]    false    true    1.7 GiB
95899ae7-c884-47f8-8e1b-12474a77619d    [http://XX.XX.XX.XX:9018]    [http://XX.XX.XX.XX:9019]    false    true    1.7 GiB
9f70d2ae-a6e6-480c-9825-0764efe0c417    [http://XX.XX.XX.XX:9018]    [http://XX.XX.XX.XX:9019]    false    true    1.7 GiB
394fa1ae-7266-426a-99bc-5f87a5fd3e8e    [http://XX.XX.XX.XX:9018]    [http://XX.XX.XX.XX:9019]    false    true    1.7 GiB
5fc10a25-4bf4-4681-b737-bb8d4753cfb6    [http://XX.XX.XX.XX:9018]    [http://XX.XX.XX.XX:9019]    true    true    1.7 GiB
```

*Prometheus* 

Verify Prometheus is up and running by logging into Prometheus (external_ip:port). Click the ‘targets’ from Status dropdown as shown below to verify if all the targets are scraped. In the example below, you can see the PX-kvdb nodes (port:9019) and PX-Enterprise (port:9001) cluster nodes as targets

![prometheustargets](/img/PrometheusTargets.png)

*Grafana*

Verify that the grafana service is up and running by logging into Grafana (default username/password) using the (external_ip:port). Check if the datasource is correctly setup as shown below:

![prometheusdatasource](/img/PrometheusDataSource.png)

Next step is to make sure you are able to view the built-in dashboards for Portworx and PX-kvdb. Here is a screenshot from the PX-kvdb dashboard.

![kvdbdashboard](/img/KvdbDashboard.png)

## Limitations
**Issue** | **Description** | **Mitigation**|
----------|-----------------|---------------|
Separate logins for Lighthouse and Grafana | No single sign-on supported for Lighthouse and Grafana | Future release will address
Admin only Lighthouse | Lighthouse currently supports only one admin user | Future release will address

## Sizing guide (for metadata services)

PX-Central node configuration for running metadata services is described in the table below (minimum 3 nodes):


**Cluster Size** | **Description**| **CPU config (cores)** | **Memory Config (GB)**| **Max concurrent IOPS** | **Disk Bandwidth (MB/s)**|
-----------------|----------------|------------------------|-------------|-------------------------|--------------------------|
Small (upto 50 node cluster) | Serves fewer than 100 clients, fewer than 200 of requests per second, and stores no more than 100MB of data | 2 | 8 | 3600 | 56.25 |
Medium (upto 250 nodes cluster) | Serves fewer than 500 clients, fewer than 1,000 of requests per second, and stores no more than 500MB of data | 4 | 16 | 6000 | 93.75 | 
Large (upto 1000 nodes cluster) | Serves fewer than 1,500 clients, fewer than 10,000 of requests per second, and stores no more than 1GB of data | 8 | 32 | 8000 | 125 |
X-Large (upto 3000 nodes cluster) | Serves fewer than 1,500 clients, fewer than 10,000 of requests per second, and stores more than 1GB of data |  16 | 64 | 16000 | 250 |  

## Adding PX-Central to existing PX installations

For customers with existing Portworx deployment, it will require the following steps to add PX-Central:

- Identify set of nodes (minimum 3) with the required specifications to deploy PX-Central components.

- First, install the monitoring components (Prometheus, Alertmanager and Grafana) on these nodes to validate you can connect to existing PX clusters.

- Install Lighthouse. Add all the clusters to Lighthouse.

- Follow the process to migrate etcd from the existing to the new one installed as part of PX-Central. PX-kvdb needs underlying PX installation to complete before migration starts.

## Troubleshooting

### Unable to access Lighthouse

* Verify if the pod is running

    ```text
    kubectl get pods -n kube-system | grep px-lighthouse
    ```

    ```text
    kubectl describe pod <lighthouse pod id> -n kube-system
    ```

* Check IP and Port (port is 32678 if using NodePort and port is 80 if using Loadbalancer)

### Unable to see metrics in Grafana?

* Verify Portworx is up and running

    ```text
    kubectl get pods -n kube-system | grep portworx
    ```

* Verify Prometheus is running

    ```text
    kubectl get pods -n kube-system | grep prometheus
    ```
* Check targets in Prometheus
* Check stats in Prometheus
* If only volume metrics are not seen, make sure the volumes are attached/mounted.

### Unable to see any alerts reported via alertmanager?

  * Check if the  rules are correctly setup in Prometheus
  * Verify Portworx is up and running

    ```text
    kubectl get pods -n kube-system | grep portworx
    ```

### Not seeing notification emails?

Make sure you did set-up the email configuration in step1 above.
