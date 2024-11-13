+++
date = 2023-08-16T09:31:00Z
description = ""
draft = false
cover = "images/2023/08/pexels-brett-sayles-4597280.jpg"
slug = "hyperconvergence-ceph-proxmox-homelab"
title = "The (Hyper)Convergence - Ceph + Proxmox"

+++

When I kicked off the latest iteration of my homelab project about 10 years ago, everything was harder.  Shared storage was a luxury that meant diving into expensive SAN solutions which were neither feasible nor affordable for anyone not running a data center.  Containers were still in buzzword territory, and their real-world application was confined either to early versions of Docker (pre-OCI, mind you!), or to cutting-edge cloud-native projects like Google's Borg.

At that time, virtualization was the future and VMware was the name in the game.  In my workplace, vSphere was so deeply entrenched in our infrastructure that it felt almost absurd to think about running anything else for my homelab.  VMware’s reliability, integrations, and comprehensive feature set made it an obvious choice for enterprises.  For a home lab, though, it wasn’t just about running VMs - it was about learning the ropes of managing real enterprise-grade infrastructure.  Thanks to generous licensing through the VMware Users Group, I ran VMware in my home lab for several years before the growing overhead of the vSphere stack pushed me to look for something else.  Perhaps the most painful part of trying to run VMware at home was the lack of reliable shared storage.

Here's a list of all the solutions I tried:
* **NFS** - NFS is the universal standard solution for "I have a file here and I want to use it over there".  Unfortunately, its performance is simply *not great* when it comes to high IO and synchronous workloads.
* **Mounting iSCSI storage to ESXi using targetcli** - This should have worked great, as iSCSI is clearly an industry standard.  But the issue here was resiliency.  I needed a physical machine to host the disks and present LUNs to the ESXi nodes, but that meant one more machine that couldn't be virtualized and was effectively another critical Jenga block in my infrastructure.
* **One Hypervisor To Rule Them All** - Putting all my eggs in one basket; a single hypervisor storing all my important VMs, with the other dedicated to only running the vSphere appliance.  I don't think I need to explain why this was a bad idea!

Fast forward to today, and my homelab has changed in ways I never imagined. Technologies that were once out of reach, like hyperconverged infrastructure and Ceph storage, are a major component of my current home lab.  Proxmox’s built-in support for Ceph has made it so much easier to get a resilient storage system up and running, all without the insane hardware costs or complexity that used to be a given.  It’s a reminder of how far the tech has come—and how much easier it’s become to experiment with advanced setups like this.  What used to be complex, expensive, and requiring a Ph.D. in Storage-ology now feels almost effortless.  That’s progress!

### Downgrading Hardware, Upgrading Expectations
When I decided to finally migrate off of ESXi, I took that opportunity to redesign my VM storage solution to a distributed model.  Instead of a vSphere cluster reading from a central NFS server, I set up a cluster of small, inexpensive micro workstations with fast local SSDs in a Ceph storage pool.  This [hyperconverged model](https://pve.proxmox.com/wiki/Deploy_Hyper-Converged_Ceph_Cluster) handles both block storage (VM disks) and file storage (ISOs) with configurable replication to survive the loss of any single host.

More importantly, the micro workstations each consume only ~20W under normal operating conditions.  Compared to a traditional rackmount server which runs around 200W at idle, I'm saving around 140W per hour while eliminating a major single point of failure in my infrastructure.

It’s been a fun project seeing what this mini powerhouse can handle.  I’m using it for everything from highly-available VMs to LXD containerized applications, and it’s holding up incredibly well.  Ceph also has native CSI integrations for Kubernetes, which means any workloads I run in my [Kubernetes cluster]({{< ref "whats-in-my-lab" >}}) can benefit from fast, fault-tolerant network storage.