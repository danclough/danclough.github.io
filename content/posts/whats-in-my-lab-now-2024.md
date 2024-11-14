+++
date = 2024-04-03
title = "What's In My Lab Now, 2024 Edition"
slug = "whats-in-my-lab-now-2024"
cover = "images/2024/04/rack.png"
tags = ["homelab"]
+++

Almost four years ago (agh!) I published [What's In My Lab]({{< ref "/posts/whats-in-my-lab" >}}), an overview of the systems and software I use at home for my own personal infrastructure.  You might think that at a certain point you achieve some kind of serenity, and the desire to modify or expand is sated.  Hah!  Guess again.
<!--more-->
### Hardware
I've continued with the trend of using low-power off-the-shelf systems.  The beauty of a product line like Dell's OptiPlex Micro is that the design is essentially unchanging from one generation to the next.  Where I used to have a range of models - 7040, 5050, etc. - I've slowly upgraded them all to the same generation of 3060s.  One extra feature I've added that you won't find on a stock OptiPlex Micro is a [secondary NIC from DFRobot](https://www.dfrobot.com/product-2318.html) connected to the WiFi M.2 slot, giving each Micro an extra 1GbE port for redundancy or a dedicated storage network.

There are only so many hard drives you can cram into an OptiPlex Micro - one, to be exact, of the 2.5" variety - so my storage needs are still served by a 2U SuperMicro rackmount server sporting 12 3.5" SAS drives split across two ZFS pools.

Since my last post, I've also added a Home Automation server built on a [SeeedStudio reServer](https://www.seeedstudio.com/reThings-reServer-c-2006.html), a vertical small form factor machine with space for two 3.5" SATA hard drives and an array of connectivity and expansion options tailored for edge computing applications.  This server runs HomeAssistant, Z-Wave and Zigbee hubs, and the Frigate NVR software which uses Intel's Rocket Lake integrated GPU to perform object detection and inferencing using the OpenVINO framework.

I also have two vintage systems from SGI and Sun which I occasionally power on and take for a spin.

|**System**|**CPU**|**Memory**|
|:-|:-|:-|
|OptiPlex 3060|Intel Core i5-8500T|32GB DDR4|
|OptiPlex 3060|Intel Core i5-8500T|32GB DDR4|
|OptiPlex 5060|Intel Core i5-8500T|32GB DDR4|
|SuperMicro CSE-826|Dual Intel Xeon E5-2630v4|64GB DDR4|
|reServer i31115|Intel Core i3 1115G4|32GB DDR4|
|Raspberry Pi 5|ARM Cortex-A76 2.4GHz|8GB LPDDR4X|
|Sun Fire V100|550MHz UltraSPARC IIi|1GB PC133|
|Cobalt "RaQFive"|StarFive JH7110 SoC 1.5GHz|8GB DDR LPDDR4|
|SGI O2|MIPS R5000 180MHz|1GB 133MHz SDRAM|

### Hypervisors

[Proxmox VE](https://www.proxmox.com/) continues to punch well above its weight class.

{{< figure src="/images/2024/04/homelab_2024.png" position="center" >}}

### Services

Not much has changed here, but some services have been retired and replaced with other products.

* HAProxy
    * Three very lightweight LXD containers serving as a virtual network load balancer
    * HAProxy acts as a TCP and HTTP load balancer
    * Keepalived manages virtual IPs that failover between the two containers
* MariaDB Galera Cluster
    * Three Debian LXD containers running MariaDB with Galera multi-master replication
    * Clients access the MariaDB cluster using a VIP and server pool managed by the HAProxy cluster
    * I wrote a [monitoring daemon in Go](https://github.com/danclough/mysql-healthcheck) that provides HTTP healthcheck capabilities for HAProxy to determine the health of each MariaDB instance
* Ceph
    * Each Proxmox host runs a Ceph mon (monitor), mgr (manager), and mds (metadata service)
    * Each host also runs a Ceph OSD on an internal 2.5" enterprise SATA SSD
* Active Directory
    * Two Windows Server 2022 VMs across each host provide Active Directory Domain Services and DNS
    * One Windows Server 2022 VM hosts an Active Directory Certificate Services intermediate CA for my [private CA hierarchy]({{< ref "/posts/implementing-a-private-ca-for-home-use" >}})
* Kubernetes
    * 3 worker nodes running [K3s](https://k3s.io/), a lightweight edge-focused Kubernetes distribution
    * The K3s cluster is highly-available, with the K8s API accessible through the HAProxy load balancer cluster
    * Workloads on Kubernetes can leverage the external MariaDB and Ceph clusters for persistence

In addition to these clustered services, I also deployed a few services for better management and visibility across my environment:
* Ansible AWX
    * Manages automation for VMs, LXD containers, and physical hosts
    * I use a number of open-source roles from the Ansible Galaxy community, and even contribute bugfixes and improvements to a few as I'm able!
* Graylog Open
    * I use a few different Graylog inputs to receive data from all of my endpoints:
      * Syslog data from rsyslog
      * GELF from Docker engines and Kubernetes pods
      * Beats for Windows event logging with Winlogbeat
    * Graylog's Sidecar functionality made it significantly easier to manage my Winlogbeat collectors in a central place, rather than relying on deploying winlogbeat via GPO!
* InfluxDB
    * Aggregates time-series monitoring data from telegraf clients on Linux
* Grafana
    * Observability into my entire infrastructure
    * Presents metrics from InfluxDB in customized dashboards for each of my home lab services