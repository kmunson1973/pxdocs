---
title: Install on Docker Standalone
description: Learn how to run Porworx as a runC container.
keywords: portworx, px-developer, px-enterprise, plugin, install, configure, container, storage, runc, oci
noicon: true
weight: 1
series: px-docker-install
---

## Why OCI

Running Portworx as a runC container eliminates any cyclical dependencies between a Docker container consuming storage from the Portworx container. It also enables you to run your Linux containers without a Docker daemon completely, while still getting all of the advantages of a Linux container and cloud native storage from Portworx.

To install and configure PX to run directly with OCI/runC, please use the configuration steps described in this section.

{{<info>}}**Migrating from PX-Containers to PX-OCI**: If you already had PX running as a Docker container (Portworx 1.2.10 and lower) and now want to upgrade to runC, follow the instructions at [Migrate Portworx installed using Docker to OCI/runc](/install-with-other/docker/standalone/migrate-docker-to-oci). {{</info>}}

## Install {#install}

### Prerequisites {#prerequisites}

* _SYSTEMD_: The installation below assumes the [systemd](https://en.wikipedia.org/wiki/Systemd) package is installed on your system \(i.e. _systemctl_ command works\).
  * Note, if you are running Ubuntu 16.04, CentoOS 7 or CoreOS v94 \(or newer\) the “systemd” is already installed and no actions will be required.
* _SCHEDULERS_: If you are installing PX into **Kubernetes** or **Mesosphere DC/OS** cluster, we recommend to install the scheduler-specific Portworx package, which provides tighter integration, and better overall user experience.
* _FIREWALL_: Ensure ports 9001-9015 are open between the cluster nodes that will run Portworx.
* _NTP_: Ensure all nodes running PX are time-synchronized, and NTP service is configured and running.
* _KVDB_: Please have a clustered key-value database \(etcd or consul\) installed and ready. For etcd installation instructions refer this [doc](/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/etcd).
* _STORAGE_: At least one of the PX-nodes should have extra storage available, in a form of unformatted partition or a disk-drive.  Also please note that storage devices explicitly given to Portworx \(ie. `px-runc ... -s /dev/sdb -s /dev/sdc3`\) will be automatically formatted by PX.

The installation and setup of PX OCI bundle is a 3-step process:

1. Install PX OCI bits
2. Configure PX OCI
3. Enable and start Portworx service

### Step 1: Install the PX OCI bundle

{{% content "install-with-other/docker/shared/runc-install-bundle.md" %}}

### Step 2: Configure PX under runC

{{% content "install-with-other/docker/shared/runc-configure-portworx.md" %}}

### Step 3: Starting PX runC

{{% content "install-with-other/docker/shared/runc-enable-portworx.md" %}}

## Upgrading the PX OCI bundle {#upgrading-the-px-oci-bundle}

To upgrade the OCI bundle, simply re-run the [installation Step 1](/install-with-other/docker/standalone#step-1-install-the-px-oci-bundle) with the `--upgrade` option. After the upgrade, you will need to restart the Portworx service.

Below command upgrades your installation to the latest stable Portworx version.
```text
latest_stable=$(curl -fsSL 'https://install.portworx.com?type=dock&stork=false' | awk '/image: / {print $2}')
sudo docker run --entrypoint /runc-entry-point.sh \
    --rm -i --privileged=true \
    -v /opt/pwx:/opt/pwx -v /etc/pwx:/etc/pwx \
    $latest_stable --upgrade
sudo systemctl restart portworx
```

## Uninstalling the PX OCI bundle {#uninstalling-the-px-oci-bundle}

To uninstall the PX OCI bundle, please run the following:

```text
# 1: Remove systemd service (if any)
sudo systemctl stop portworx
sudo systemctl disable portworx
sudo rm -f /etc/systemd/system/portworx*

# NOTE: if the steps below fail, please reboot the node, and repeat the steps 2..5

# 2: Unmount oci (if required)
grep -q '/opt/pwx/oci /opt/pwx/oci' /proc/self/mountinfo && sudo umount /opt/pwx/oci

# 3: Remove binary files
sudo rm -fr /opt/pwx

# 4: [OPTIONAL] Remove configuration files. Doing this means UNRECOVERABLE DATA LOSS.
sudo chattr -ie /etc/pwx/.private.json
sudo rm -fr /etc/pwx
```

#### Logging and Log files {#logging-and-log-files}

The [systemd\(1\)](https://en.wikipedia.org/wiki/Systemd) uses a very flexible logging mechanism, where logs can be viewed using the `journalctl` command.

For example:

```text
# Monitor the Portworx logs
sudo journalctl -f -u portworx

# Get a slice of Portworx logs
sudo journalctl -u portworx --since 09:00 --until "1 hour ago"
```

However, if you prefer to capture Portworx service logs in a separate log file, you will need to modify your host system as follows:

```text
# Create a rsyslogd(8) rule to separate out the PX logs:
sudo cat > /etc/rsyslog.d/23-px-runc.conf << _EOF
:programname, isequal, "px-runc" /var/log/portworx.log
& stop
_EOF

# Create logrotate(8) configuration to periodically rotate the logs:
sudo cat > /etc/logrotate.d/portworx << _EOF
/var/log/portworx.log {
    daily
    rotate 7
    compress
    notifempty
    missingok
    postrotate
        /usr/bin/pkill -HUP syslogd 2> /dev/null || true
    endscript
}
_EOF

# Signal syslogd to reload the configurations:
sudo pkill -HUP syslogd
```

**Advanced usage: Interactive/Foreground mode**

Alternatively, one might prefer to first start the PX interactively \(for example, to verify the configuration parameters were OK and the startup was successful\), and then install it as a service:

```text
# Invoke PX interactively, abort with CTRL-C when confirmed it's running:
sudo /opt/pwx/bin/px-runc run -c MY_CLUSTER_ID \
    -k etcd://myetc.company.com:2379 \
    -s /dev/xvdb

[...]
> time="2017-08-18T20:34:23Z" level=info msg="Cloud backup schedules setup done"
> time="2017-08-18T20:34:23Z" level=info msg="Starting REST service on socket : /run/docker/plugins/pxd.sock"
> time="2017-08-18T20:34:23Z" level=info msg="Starting REST service on socket : /var/lib/osd/driver/pxd.sock"
> time="2017-08-18T20:34:23Z" level=info msg="PX is ready on Node: 53f5e87b... CLI accessible at /opt/pwx/bin/pxctl."
[ hit Ctrl-C ]
```
