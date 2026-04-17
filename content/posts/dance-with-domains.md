---
date: 2026-04-16
title: "A Dance with Domains: Introducing Samba to my Active Directory in 2026"
slug: "a-dance-with-domains"
cover: "images/2026/04/pexels-brett-sayles-2881224.jpg"
tags: ["homelab", "active-directory", "samba", "linux"]
---

In my previous post, [What's In My Lab Now, 2024 Edition]({{< ref "/posts/whats-in-my-lab-now-2024" >}}), I detailed my lab's Active Directory setup which relies on Windows Server 2022 VMs.  While that setup has served me perfectly well to date, recent advancements in Samba's support for Active Directory features, combined with a desire to reduce my need for software licenses in the future, presented an opportunity to experiment with cross-platform AD domains.
<!--more-->
## Why Samba, and why now?

Samba is a fantastic solution for integrating Linux and Windows systems as seamlessly as possible.  I've used Samba as a basic file server for Windows and Mac clients for the better part of 20 years with almost no fuss.  Even when I first set up my homelab AD domain in the mid-2010s, it was trivial to integrate Samba into the domain as a member server to handle authentication and authorization for file shares and even provide networked home directories over SMB.

The evolving roadmap of Active Directory is where things started to get a bit cloudy.  In the mid-2010's, everything enterprise started moving toward cloud-native, but Microsoft's Active Directory platform still seemed to have plenty of on-prem life left.  With the earlier advent of Azure AD, and its eventual rebrand to Entra ID, the lines kept blurring between on-prem and cloud and I'm now more eager to search for an alternative.

## Get On My (Functional) Level

Samba's support for AD features (indicated by tiers called _Functional Levels_) had plateaued for many years.  Samba versions 4.0 through 4.18 supported, at most, Functional Level 2008R2.  This changed with Samba release 4.19, which introduced compatibility for the Server 2012 functional levels, and more recently with Samba version 4.20 which added general compatibility with Server 2016.

While I can practically guarantee that _nothing_ I do in my lab comes close to needing 2016 Functional Level requirements, it still felt good to be on the latest and greatest.  Waiting for Samba's support to catch up wasn't about just being able to join a machine to the domain - Samba domain members never cared much about the functional levels in the first place - but if I decided to add a Linux-based domain controller, ensuring that Group Policy, Kerberos authentication, and replication behaved predictably across Windows and Linux was a top priority.

## Hurdles

There were three main components that I suspected would present the most challenge to my plans.

### Group Policies
The way Active Directory stores and replicates Group Policy Objects is through _Distributed File System Replication_, or DFS-R for short.  AD Domain Controllers present a SYSVOL share that each DC replicates seamlessly, and each client reads GPOs from their respective logon server.

Samba provides no support for DFS-R, so right out of the box, a hybrid domain will have inconsistencies in which some DCs contain GPOs and some do not.  There seem to be no ready-made solutions for replicating the SYSVOL from the domain's PDC Emulator host.  While Samba handles much of the heavy lifting of replicating LDAP objects around the domain, keeping the SYSVOL share in sync between Windows and Linux is left to the system admin.

To ensure my Linux DCs could replicate the latest GPOs and scripts without manual intervention, I pieced together a [fairly straightforward cron script]({{< ref "/notes/linux/samba/replicating_sysvol" >}}).  Using the existing Kerberos machine principal as the key, it obtains a Kerberos ticket and creates a SMB mount of the Windows PDC's SYSVOL share.  It uses `rsync` to pull those changes to the local Samba SYSVOL directory, and then uses Samba's built-in `sysvolreset` utility to ensure the replicated files have the proper NT ACLs applied.  It’s not the most elegant or foolproof solution, but for my homelab environment, it's reliable, flexible, and most importantly, functional.

### Certificate Services
Though most sprawling enterprises maintain their own PKI hierarchy separate from AD, smaller operations are often constrained in ways that justify using Active Directory Certificate Services to host an [online intermediate CA]({{< ref "/posts/implementing-a-private-ca-for-home-use" >}}) as a part of their domain.  There doesn't seem to be any limitation here on the part of Samba, save for Samba DCs not being able to auto-enroll with an ADCS CA for things like LDAPS.  If we temper our expectations a bit and simply aim for having _a valid certificate_ automatically issued and renewed on each DC, then dear reader, you will not be disappointed.

In my previous post on building a PKI hierarchy, I detailed how I moved away from manual certificate management in favor of a tiered approach: 
```
Offline Root → AD Certificate Services (Intermediate) → Smallstep (Issuing) → Clients
```

The challenge with adding Linux DCs is that they still need valid, trusted certificates for proper secured LDAP connections. I didn't want to be the guy manually generating CSRs every time a certificate approached its expiration date.

Instead, I leveraged Smallstep's `step-cli`.  Because my Smallstep CA provides an API endpoint and its CA is integrated into my existing chain of trust, I was able to automate the enrollment process for the Samba DCs.  Now, by using the `step-cli` in daemon mode, the Linux domain controllers handle their own certificate renewals automatically.  As an added benefit, the daemon renewal process allows me to reliably issue certificates with a lifetime of just a few days, rather than months or years.

### Smart Card Authentication
Finally, the most frustrating, yet least-documented challenge I'd face would be using Smart Card authentication on Windows against a Linux domain controller.  It became a grueling exercise in Kerberos troubleshooting; I spent hours testing configurations and digging through dated documentation, often getting tantalizingly close to a working solution only to have it still fall apart during a login.  Ask me how many times I audibly sighed as I read from the logs the words `PKINIT request but PKINIT not enabled`.

Ultimately, after weighing the large amount of time spent against the actual utility it would provide in my lab, I decided to scrap Smart Card authentication for now.  While the goal of enhanced security is always near the top of my list, sometimes you have to admit defeat, put it on the backburner, and come back later when you've recovered some precious sanity.

## The Result
The result of my experiment is a heterogeneous Active Directory domain that feels, for all intents and purposes, mostly homogeneous.  My Windows Server 2022 nodes and my Samba DCs coexist, sharing the same trust roots and the same policy definitions.  It’s another step toward the serenity I mentioned in previous posts, where the infrastructure itself is complex enough to be useful, but automated enough to stay out of my way.
