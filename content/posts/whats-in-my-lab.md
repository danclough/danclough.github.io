+++
date = 2020-10-04T02:14:16Z
description = ""
draft = false
slug = "whats-in-my-lab"
title = "What's In My Lab"

+++


Like many people in IT, I've been running a home lab for several years.  My home lab has become progressively more complicated over the years as I've layered in new technologies that I want to explore and added new services to my home network.

### Hardware

My home lab runs mainly on low-power Dell OptiPlex ultra-small-form-factor (USFF) hardware.  I wanted this lab to be always-on - so small, quiet, and energy-efficient hardware is a must-have.  The notable exception is my NAS, a 2U Dell PowerEdge rackmount server with an array of 3.5" SAS spinning disks striped in a large ZFS RAID-Z2 pool.

### Hypervisors

For several years, I ran VMware ESXi mostly as a learning opportunity.  Over time I found that the overhead of running ESXi on low-spec commodity hardware was too great, and decided to shift my workloads over to [Proxmox VE](https://www.proxmox.com/).

{{< figure src="/images/2020/10/infra_services.png" position="center" >}}

### Services

Each physical system in my homelab runs the Debian-based Proxmox VE hypervisor.  The hypervisors host a mix of LXC-based containers and QEMU virtual machines.  From top to bottom:

* HAProxy
    * Two very lightweight LXD containers serving as a virtual network load balancer
    * HAProxy acts as a TCP and HTTP load balancer
    * Keepalived manages virtual IPs that failover between the two containers
* MariaDB Galera Cluster
    * Three Debian LXD containers running MariaDB with Galera multi-master replication
    * Clients access the MariaDB cluster using a VIP and server pool managed by the HAProxy cluster
    * I wrote a [monitoring daemon in Go](https://github.com/danclough/mysql-healthcheck) that provides HTTP healthcheck capabilities for HAProxy to determine the health of each MariaDB instance
* Ceph
    * Each Proxmox host runs a Ceph monitor
    * Each host also hosts a Ceph OSD on an internal 2.5" SATA disk
* Active Directory
    * Three MSDN Server 2019 Core VMs across each host provide Active Directory services
* Kubernetes
    * 3 control plane nodes with taints
    * 1 worker node
    * Workloads on Kubernetes are able to leverage the external MariaDB and Ceph clusters for persistence

In addition to these clustered services, I also deployed a few standalone services for better management and visibility across my environment:
* Chef Infra Server
    * Manages automation for VMs, LXD containers, and physical hosts
    * I've also open-sourced [a few of the cookbooks](https://github.com/danclough/chef-qemu_guest) I created to manage my lab systems
* Elastic ELK Stack
    * Logstash takes syslog data and Kubernetes logs from Filebeat and indexes them into Elasticsearch
    * A Kibana instance running on Kubernetes provides search and visualization capabilities
* InfluxDB
    * Aggregates time-series monitoring data from telegraf clients on Linux
    * Provides data to a Grafana instance running on Kubernetes



