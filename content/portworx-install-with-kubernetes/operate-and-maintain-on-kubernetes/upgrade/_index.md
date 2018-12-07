---
title: Upgrade on Kubernetes
linkTitle: Upgrade
weight: 2
hidesections: true
keywords: portworx, container, Kubernetes, upgrade, update, storage, Docker, k8s, flexvol, pv, persistent disk
description: Steps on how to upgrade Portworx on Kubernetes
noicon: true
series: k8s-op-maintain
---

This guide describes the procedure to upgrade Portworx running as OCI container using [talisman](https://github.com/portworx/talisman).

To upgrade to the **2.0** release, run the curl command:
```text
curl -fsL https://install.portworx.com/2.0/upgrade | bash -s
```

To upgrade to the **1.7** release, run the curl command:
```
curl -fsL https://install.portworx.com/1.7/upgrade | bash -s
```

This runs a script that will start a Kubernetes Job to perform the following operations:

1. Runs a DaemonSet on the cluster which fetches the new Portworx image. This reduces the time Portworx is down between the old and new versions as the image is already pulled.
2. If the upgrade is from version 1.2 to 1.3 or 1.4, it will scale down all Deployments and StatefulSets that use shared Portworx PersistentVolumeClaims. If you are already at version 1.4 and upgrading to subsequent versions, this is not required.
3. Triggers RollingUpdate of the Portworx DaemonSet to the default stable image.
   * If the upgrade is from version 1.2 to 1.3 or 1.4, all application pods using Portworx PersistentVolumeClaims will be rescheduled to other nodes in the cluster before the new Portworx version starts on that node. If you are already at version 1.4 and upgrading to subsequent versions, this is not required.
4. Restore any Deployments or StatefulSets that were scaled down in step 2 back to their original replicas.

This script will also monitor the above operations.

### Customizing the upgrade process {#customizing-the-upgrade-process}

#### Specify a different Portworx upgrade image {#specify-a-different-portworx-upgrade-image}

You can invoke the upgrade script with the _-t_ to override the default Portworx image. For example below command upgrades Portworx to _portworx/oci-monitor:1.7.2_ image.

```text
curl -fsL https://install.portworx.com/upgrade | bash -s -- -t 1.7.2
```

#### Disable scaling down of shared Portworx applications during the upgrade {#disable-scaling-down-of-shared-portworx-applications-during-the-upgrade}

You can invoke the upgrade script with _–scaledownsharedapps off_ to skip scaling down Deployments and StatefulSets that use shared Portworx PersistentVolumeClaim.

For example:

```text
curl -fsL https://install.portworx.com/upgrade | bash -s -- --scaledownsharedapps off
```

{{<info>}}
**Reboot requirement:**  
By default, the upgrade process scales down shared applications as that avoids a node reboot when upgrading between major Portworx versions. Disabling that flag would mean the node would need a reboot before Portworx comes up with the new major version.
{{</info>}}

### Troubleshooting {#troubleshooting}

#### Find out status of Portworx pods {#find-out-status-of-portworx-pods}

To get more information about the status of Portworx daemonset across the nodes, run:

```text
kubectl get pods -o wide -n kube-system -l name=portworx
NAME             READY   STATUS              RESTARTS   AGE   IP              NODE
portworx-9njsl   1/1     Running             0          16d   192.168.56.73   minion4
portworx-fxjgw   1/1     Running             0          16d   192.168.56.74   minion5
portworx-fz2wf   1/1     Running             0          5m    192.168.56.72   minion3
portworx-x29h9   0/1     ContainerCreating   0          0s    192.168.56.71   minion2
```

As we can see in the example output above:

* looking at the STATUS and READY, we can tell that the rollout-upgrade is currently creating the container on the “minion2” node
* looking at AGE, we can tell that:
  * “minion4” and “minion5” have Portworx up for 16 days \(likely still on old version, and to be upgraded\), while the
  * “minion3” has Portworx up for only 5 minutes \(likely just finished upgrade and restarted Portworx\)
* if we keep on monitoring, we will observe that the upgrade will not switch to the “next” node until STATUS is “Running” and the READY is 1/1 \(meaning, the “readynessProbe” reports Portworx service is operational\).

#### Find out version of all nodes in Portworx cluster {#find-out-version-of-all-nodes-in-portworx-cluster}

One can run the following command to inspect the Portworx cluster:

```text
PX_POD=$(kubectl get pods -n kube-system -l name=portworx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $PX_POD -n kube-system /opt/pwx/bin/pxctl cluster list

[...]
Nodes in the cluster:
ID      DATA IP         CPU        MEM TOTAL  ...   VERSION             STATUS
minion5 192.168.56.74   1.530612   4.0 GB     ...   1.2.11.4-3598f81    Online
minion4 192.168.56.73   3.836317   4.0 GB     ...   1.2.11.4-3598f81    Online
minion3 192.168.56.72   3.324808   4.1 GB     ...   1.2.11.10-421c67f   Online
minion2 192.168.56.71   3.316327   4.1 GB     ...   1.2.11.10-421c67f   Online
```

* from the output above, we can confirm that the:
  * “minion4” and “minion5” are still on the old Portworx version \(1.2.11.4\), while
  * “minion3” and “minion2” have already been upgraded to the latest version \(in our case, v1.2.11.10\).

#### Manually restoring scaled down shared applications {#manually-restoring-scaled-down-shared-applications}

If the upgrade job crashes unexpectedly and fails to restore shared applications back to their original replica counts, you can run the following command to restore them.

```text
curl -fsL https://install.portworx.com/upgrade | bash -s -- --scaledownsharedapps off
```
