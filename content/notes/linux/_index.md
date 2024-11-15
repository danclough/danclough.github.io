+++
title = "Linux Notes"
slug = "linux"
+++
# Table of Contents
{{% toc %}}


## Security
### Certificates
#### Adding certificates to the trusted CAs
Different Linux distributions have their own ways and locations of storing trusted CAs and self-signed certificates.  Table below for quick reference.

|**Distribution Family**|**Distributions**|**Import location**|**Update utility**|
|-|-|-|-|
|Debian|Kali Linux<br> Raspberry Pi OS<br> Ubuntu|`/usr/local/share/ca-certificates`|`update-ca-certificates`|
|RHEL/Fedora|Amazon Linux<br> AlmaLinux<br> CentOS<br> Rocky Linux|`/etc/pki/ca-trust/source/anchors`|`update-ca-trust`|
|Arch Linux||`/etc/ca-certificates/trust-source/anchors`|`trust anchor --store <cert file>` or `update-ca-trust`|