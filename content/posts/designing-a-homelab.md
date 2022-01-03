+++
date = 2020-08-30T12:43:52Z
description = ""
draft = false
cover = "images/2020/08/stef-westheim-ZGH6xd3usAs-unsplash.jpg"
slug = "designing-a-homelab"
title = "Designing a Homelab"

+++


Just as a motorhead might get their endorphine fix attending a vintage car show, I browse [r/homelab](https://reddit.com/r/homelab) almost daily and fawn over the beautifully-curated racks full of enterprise grade hardware sitting in some random person's apartment.  I often get the feeling that I won't be truly satisfied until I have a few racks of my own, full of routers, switches, servers, and disk shelves.

What would I do with them?  Aside from stress-testing my home's electrical wiring, I really don't know.

And with that, we have our first piece of advice for beginning homelabbers

1. **Don't Overthink It**

Browsing [r/homelab](https://reddit.com/r/homelab) is a blessing and a curse.  You want the shinys and the blinkenlights, but beauty is only skin deep.

2. **Be Goal-Oriented**

What do you want or need from your first lab?  Consider the examples below and good, inexpensive options for each.

* **Designing web sites** - static pages, PHP, Node.JS, etc.
    * Raspberry Pi 3 or 4 - $40
* **Small low-power media server** - Plex, XBMC, etc.
    * Raspberry Pi 4 (4GB) with a USB hard drive - $80
* **Working with containers** - Docker, Docker Compose
    * Raspberry Pi 4 (4-8GB) with a USB hard drive or SSD - $140
        * The 64-bit arm64 architecture is supported by many popular containers on Docker Hub and other public registries.
        * RPi 4 supports booting from USB devices rather than unreliable MicroSD, alleviating a common complaint of Pi users with write-heavy workloads.
        * The newest Pi 4 SKU comes with 8GB RAM - plenty of headroom for running multiple containers.
    * Small NUC-sized x86-64 PC - $200
* **Running VMs**
    * Any x86-64 PC running Proxmox VE, a free Linux QEMU-based hypervisor
    * Proxmox VE's hardware support is only limited by what Debian supports - which, as it turns out, is *a lot*.

3. **Don't Be Afraid To Break It**

As a hands-on learner, I retain information most effectively when I can apply it to a real problem.  As you change up, tear down, and rebuild the parts of your homelab, you might find a new solution to an old problem, or think up a new way of doing something you did before.

The beauty of a homelab is all right there in the name - **home**  **lab**.  It's yours to break and fix as you see fit, for learning or just for fun.  No service-level agreements to worry about, no stakeholders to notify, no change control processes to adhere to.

If you come to depend on the services you run in your homelab, start focusing your efforts on high availability.  A few examples to note:

* **Failover** - Set up a load balancer to have one IP or web URL point to two servers, so when one goes down the other takes over.  Often this happens without you even knowing, so set up monitoring while you're at it to know when a server experiences issues.
* **Fault Tolerance -** Are all your files stored on one hard drive?  Try setting up a RAID mirror so your services keep running if a drive dies.  Do you frequently experience power outages?  Install an **Uninterruptable Power Supply (UPS)** to provide backup power to your devices during a utility blackout.
* **Disaster Recovery -** How quickly can you get things back up and running after a server failure? **** Copy your services or container configs to another server using a scheduled backup process, and test the restore process periodically to ensure that it works.







