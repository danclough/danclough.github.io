+++
title = "Linux Notes"
slug = "linux"
+++
{{% bc %}}
# Table of Contents
{{% toc %}}

## Security
### Certificates
---
#### Adding certificates to the trusted CAs
Different Linux distributions have their own ways and locations of storing trusted CAs and self-signed certificates.  Table below for quick reference.

|**Distribution**|**Import location**|**Update utility**|
|---|---|---|
|Debian<br> Ubuntu<br> Raspberry Pi OS<br> Kali Linux|`/usr/local/share/ca-certificates`|`update-ca-certificates`|
|RHEL<br> Fedora<br> Amazon Linux<br> AlmaLinux<br> CentOS<br> Rocky Linux|`/etc/pki/ca-trust/source/anchors`|`update-ca-trust`|
|Arch Linux|`/etc/ca-certificates/trust-source/anchors`|`trust anchor --store <cert file>` or `update-ca-trust`|
---