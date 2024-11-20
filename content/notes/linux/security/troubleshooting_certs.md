+++
title = "Troubleshooting SSL or TLS Certificates"
+++
##### Checking local certificates
```bash
## Issued and expiration dates
openssl x509 -in cert.pem -noout -dates

## SANs or other extentions
openssl x509 -in cert.pem -noout -ext subjectAltName # Shows all domains listed on the certificate
openssl x509 -in cert.pem -noout -ext basicConstraints,keyUsage # Handy for checking CAs

## Full details
openssl x509 -in cert.pem -noout -text
```

##### Checking certificates installed on a server
###### PREREQUISITE - Set hostname and port variables
Set HOST equal to the hostname you want to check the certificate for.

```bash
HOST=my.hostname.com
# SOCKET=my.hostname.com:443 # This is the hostname and port (or an IP address/port)
```
If you want to check against a server other than what the DNS address resolves to, define the SOCKET variable as well.  If you don't set it, my snippets below will just assume it's the same as $HOST and port 443.

```bash
###### Basic connection and negotiation details - protocol, ciphersuite, hash, and key strength
echo | openssl s_client -servername $HOST -connect ${SOCKET:-"$HOST:443"} -brief

###### Certificate chain
echo | openssl s_client -servername $HOST -connect ${SOCKET:-"$HOST:443"} 2>/dev/null

###### Issued and expiration dates
echo | openssl s_client -servername $HOST -connect ${SOCKET:-"$HOST:443"} 2>/dev/null | openssl x509 -noout -dates

###### SANs or other extensions
echo | openssl s_client -servername $HOST -connect ${SOCKET:-"$HOST:443"} 2>/dev/null | openssl x509 -noout -ext subjectAltName
echo | openssl s_client -servername $HOST -connect ${SOCKET:-"$HOST:443"} 2>/dev/null | openssl x509 -noout -ext basicConstraints,keyUsage

###### Full details of the host certificate
echo | openssl s_client -servername $HOST -connect ${SOCKET:-"$HOST:443"} 2>/dev/null | openssl x509 -text -noout 
```