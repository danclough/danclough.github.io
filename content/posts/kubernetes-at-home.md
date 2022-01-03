+++
author = "Dan"
date = 2021-01-19T14:33:00Z
description = ""
draft = false
slug = "kubernetes-at-home"
title = "Kubernetes @ Home"

+++


Now that I've shared some of my physical infrastructure in my home lab, I want to share the service topology for the specific services I run at home for home automation, secure storage, and even this website.

{{< figure src="/images/2021/05/service-topology-1.png" >}}

## Services

As you can see, a number of the specific services I run are focused around infrastructure and system management.  Services such as Grafana and Kibana are essential for monitoring and observability for my entire home lab environment.  A few of the other services not pictured, for brevity:
* **phpIPAM** - Network IP management and tracking database
* **LDAP Self-Service Portal** - A simple self-service web application for managing LDAP/Active Directory passwords
* **phpMyAdmin** - Web-based management interface for my external MariaDB/Galera database cluster

## Storage
One recent addition to my lab environment is [Ceph](https://ceph.io/).

One of the most difficult and frustrating problems of managing containerized applications across multiple hosts is data persistence and replication - especially when operating on bare metal.  In my professional experience managing platforms such as *Azure Kubernetes Service*, I've been able to leverage the built-in StorageClass drivers such as `AzureFile` and *Azure Managed Disks* for each managed Kubernetes environment.

When you're on bare metal, however, you're left to fend for yourself.  I found Ceph much more complicated to configure at first than GlusterFS, but more resilient and overall easier to integrate into my K8s environment.

## Ingress and Load Balancing

I've been using Traefik as an Ingress Controller for Docker for several years, and it's greatly simplified the ingress routing configuration of each of my services.

Traefik has built-in ACME support which allows me to utilize Let's Encrypt for solutions that I host on an external domain (i.e., this website and my NextCloud instance).  For services on my internal LAN domain, I employ [cert-manager](https://cert-manager.io/) to automatically generate and manage TLS certificates from an internal Certificate Authority based on certificate requests from any workloads deployed to my cluster.

Similarly to storage, another hard problem of bare-metal K8s is the lack of a network-level load balancer to bring ingress traffic to your cluster.  K8s doesn't have any integrated functionality to provide a cluster-wide, externally-accessible IP address for Services.  The most effective solution to this problem that I've found thus far is [MetalLB](https://metallb.universe.tf/).  MetalLB interacts with your external network infrastructure - whether through Layer 2 ARP or Layer 3 routing protocols - to provide Services with an external IP address through the existing LoadBalancer Service type.



