+++
date = 2021-10-21T00:18:34Z
description = ""
draft = false
cover = "images/2021/10/kevin-ku-w7ZyuGYNpRQ-unsplash.jpg"
slug = "case-for-homelab-security"
title = "The Case for Home Lab Security"

+++


Perhaps unsurprisingly, as my home lab and local area network have matured over the years, both I and my family have come to depend on the assortment of services that I run within the four walls of my home.  We begin to take things for granted.

For instance, we put all of our important files - recipes, documents, photos, home videos, etc. - on "the server".  The contents of said server are invaluable to us, economically and emotionally.  I take very specific steps to ensure the safety of our data, including ZFS volume-level encryption and frequent encrypted backups to an off-site backup provider.  (Shoutout to [Backblaze B2](https://www.backblaze.com/b2/cloud-storage.html)!)

Having reliable, 24x7 access to all of my family's data is one hell of an accomplishment, but it's not enough any more.  I've seen too many software vulnerabilities in my still-young IT career to be able to blindly trust any single layer of application security.  For the same reasons that my data needs to remain safe and accessible, I also need to keep it secure from prying eyes and malicious actors.

The IT Infosec landscape of today is almost unrecognizable compared to when I started in IT just over 10 years ago.  Back then, vendors were clawing their way into our voicemails, inboxes, and conference rooms to pitch some all-encompassing enterprise security platform that promised to keep mysterious hooded hackers out of your unauthenticated SMB 1 file shares and publicly-accessible SharePoint sites.

Today, the vendors are still there and eager as ever to give you a four-hour sales demo - but as an industry we've also adopted a lot of common-sense policies that thankfully don't take a multi-million-dollar contract and six figures worth of computing resources to maintain.  Namely:

* Password-protect everything
* Enable Two-Factor Authentication (2FA)
* Configure Single Sign-On (SSO) wherever possible
* Perhaps most importantly, HTTPS **absolutely everywhere**

If you follow even just half of the policies above, I promise you - your lab will be just as secure, if not more secure, than the majority of corporate IT environments you'll come across.

That being said, here are the steps I'm taking to secure my home lab:

1. Implement an internal Public Key Infrastructure (PKI) for my local domain,
2. Secure all TLS-capable services on my local network with certificates from the internal PKI, and
3. Setup auto-renewal to allow short certificate lifespans (days to weeks) without manual intervention or rotation.

I fully intend for this blog to be a place for learning and sharing my experiences, so in a coming post I'll attempt to document every step of the process and write about a few of the hurdles I had to overcome.  I'll also share my thoughts on a few **** of the fantastic open-source products that I tried out along the way.

