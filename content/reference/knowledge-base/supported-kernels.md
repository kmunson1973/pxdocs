---
title: Portworx Supported Kernels
keywords: portworx, install, configure, prerequisites, kernel, kernels
description: To install and configure PX with Docker user namespaces enabled, use the steps in this section. Find out more!
weight: 5
linkTitle: Portworx Supported Kernels
---

Portworx runs as a Docker or OCI container, available on the DockerHub. Portworx has a dependency on the kernel module, which must be installed on hosts.  Portworx is distributed with pre-built kernel modules for select Centos and Ubuntu Linux distributions. If your kernel version is not listed in the table below, Portworx attempts to download kernel headers to compile it’s kernel module. This can fail if the host sits behind a proxy.  If you already have the kernel-headers and kernel-devel packages installed the module will compile successfully.  If you do not have the packages you will need to first install them before restarting Portworx.  

To install the kernel headers and kernel development packages for kernels not listed in the table below.

#### CentOS

```text
yum install kernel-headers-`uname -r`  
yum install kernel-devel-`uname -r`

```

#### Ubuntu

```text
apt install linux-headers-$(uname -r)  

```

#### Portworx CentOS/RHEL supported kernels

Portworx Version|Latest Supported Kernel Version
-------------|-----------------------
2.0.0|3.10.0-957.el7.x86_64
1.7.2|3.10.0-957.el7.x86_64
1.7.1.1|3.10.0-862.14.4.el7.x86_64
1.7.1|3.10.0-862.14.4.el7.x86_64
1.7.0|3.10.0-862.14.4.el7.x86_64
1.6.1.4|3.10.0-862.14.4.el7.x86_64
1.6.1.3|3.10.0-862.14.4.el7.x86_64
1.6.1.2|3.10.0-862.14.4.el7.x86_64
1.6.1.1|3.10.0-862.14.4.el7.x86_64
1.6.1|3.10.0-862.14.4.el7.x86_64
1.6.0|3.10.0-862.11.6.el7.x86_64
1.5.1|3.10.0-862.11.6.el7.x86_64|

#### Portworx Ubuntu supported kernels

Portworx Version|Latest Supported Kernel Version
-------------|-----------------------
2.0.0|4.19.5-041905-generic
1.7.1.1|4.19.1-041901-generic
1.7.1|4.19.0-041900-generic
1.7.0|4.19.0-041900-generic
1.6.1.4|4.19.0-041900-generic
1.6.1.3|4.19.0-041900-generic
1.6.1.2|4.18.16-041816-generic
1.6.1.1|4.18.12-041812-generic
1.6.1|4.18.11-041811-generic
1.6.0|4.18.8-041808-generic
1.5.1|4.18.7-041807-generic
