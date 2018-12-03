---
title: Disk Provisioning on Google Cloud Platform (GCP)
description: This page describes how to setup a production ready Portworx cluster in a Google Cloud Platform (GCP).
keywords: portworx, container, Kubernetes, storage, Docker, k8s, pv, persistent disk, gke, gce
weight: 2
linkTitle: GCP
noicon: true
---

{{<info>}}If you are running on GKE, visit [Portworx on GKE](/portworx-install-with-kubernetes/cloud/gke).{{</info>}}

The steps below will help you enable dynamic provisioning of Portworx volumes in your GCP cluster.

## Prerequisites

**Key-value store**

Portworx uses a key-value store for it's clustering metadata. Please have a clustered key-value database (etcd or consul) installed and ready. For etcd installation instructions refer this [doc](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/etcd).

**Firewall**

Ensure ports 9001-9015 are open between the nodes that will run Portworx. Your nodes should also be able to reach the port KVDB is running on (for example etcd usually runs on port 2379).

## Create a GCP cluster

To manage and auto provision GCP disks, Portworx needs access to the GCP Compute Engine API.   There are two ways to do this.

### Using instance priviledges

Give your instances priviledges to access the GCP API server.  This is the preferred method since it requires the least amount of setup on each instance.

### Using an account file

Alternatively, you can give Portworx access to the GCP API server via an account file and environment variables. First, you will need to create a service account in GCP and download the account file.

1. Create a service account in the "Service Account" section that has the Compute Engine admin role.
2. Go to IAM & admin  -> Service Accounts -> (Instance Service Account) -> Select "Create Key" and download the `.json` file

This json file needs to be made available on any GCP instance that will run Portworx.  Place this file under a `/etc/pwx/` directory on each GCP instance.  For example, `/etc/pwx/gcp.json`.

## Install

If you used an account file above, you will have to configure the PX installation arguments to access this file by way of it's environmnet variables.  In the installation arguments for PX, pass in the location of this file via the environment variable `GOOGLE_APPLICATION_CREDENTIALS`. For example, use `-e GOOGLE_APPLICATION_CREDENTIALS=/etc/pwx/gcp.json`.

If you installing on Kuberenetes, you can use a secret to mount `/etc/pwx/gcp.json` into the Portworx Daemonset and then expose `GOOGLE_APPLICATION_CREDENTIALS` as an env in the Daemonset.

Follow [these instructions](./) to install Portworx based on your container orchestration environment.

### Disk template

Portworx takes in a disk spec which gets used to provision GCP persistent disks dynamically.

A GCP disk template defines the Google persistent disk properties that Portworx will use as a reference. There are 2 ways you can provide this template to Portworx.

**1. Using a template specification**

The spec follows the following format:
```text
"type=<GCP disk type>,size=<size of disk>"
```

* __type__: Following two types are supported
    * _pd-standard_
    * _pd-ssd_
* __size__: This is the size of the disk in GB

See [GCP disk](https://cloud.google.com/compute/docs/disks/) for more details on above parameters.

Examples:

* `"type=pd-ssd,size=200"`
* `"type=pd-standard,size=200", "type=pd-ssd,size=100"`


**2. Using existing GCP disks as templates**

You can also reference an existing GCP disk as a template. On every node where PX is brought up as a storage node, a new GCP disk(s) identical to the template will be created.

For example, if you created a template GCP disk called _px-disk-template-1_, you can pass this in to PX as a parameter as a storage device.

Ensure that these disks are created in the same zone as the GCP node group.

### Limiting storage nodes

{{% content "cloud-references/auto-disk-provisioning/shared/asg-limit-storage-nodes.md" %}}