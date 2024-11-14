+++
date = 2024-02-16
title = "Raspberry Pi 5's NVMe is a Killer Feature"
description = ""
cover = "images/2024/02/RPi5.jpg"
slug = "pi-5-nvme-killer-feature"
tags = ["homelab","raspberrypi"]
+++

Running a Kubernetes cluster on a few Raspberry Pi 4s has been a rewarding but challenging experience.  The biggest limitation was storage - each Pi was booting from an SD card, which, while convenient, proved to be a massive bottleneck.  SD cards are not only slow, but also wear out quickly under constant read/write operations.  Over time, I experienced several cases of data corruption which brought that cluster to its knees.  In fact, I've got almost 1TB worth of dead (or dying) MicroSD cards in a bench drawer from my endless sacrifices to the Non-Volatile Memory Gods.  Constant awareness of this flaw from the resulting expensive heap of silicon pictured below made it very hard to me to trust that cluster for anything beyond experimentation.

{{< figure src="/images/2024/02/sdcards.png" position="center" >}}

When the Raspberry Pi 5 was announced, its PCIe 3.0 lane immediately caught my eye.  I've toyed with USB-attached SATA and NVMe options in the past, but the limitations and fickleness of USB always made those solutions prone to failure in any number of fun and unpredictable ways.  After years of begging and pleading with SD cards and eMMC modules to behave themselves, we finally have a solution!  When I took delivery of my Pi5, I paired it with a new [Argon ONE V3 NVMe case](https://argon40.com/products/argon-one-v3-m-2-nvme-case) and was incredibly pleased with the result.

By supporting NVMe storage as a boot option, the Pi 5 significantly improves both performance and durability, making it much more feasible to run small-scale workloads without fear of data loss or SD card failure.  This improvement opens up exciting possibilities for homelabs - for small labs and test clusters, it's no longer an ill-fated endeavour to move storage-intensive applications like OpenSearch and Graylog from a traditional x86 rack server to more compact and power-efficient boards like the Pi.  My hyperconverged Ceph cluster on Proxmox, which I used to present durable storage to the KubePi cluster, can go back to being mostly idle save for a small number of highly-available VMs running on the hypervisors.  For me, this upgrade means my homelab cluster can move beyond hobby-level projects and start handling more serious workloads.