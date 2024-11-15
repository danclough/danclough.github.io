+++
title = "Java Notes"
slug = "java"
+++
# Table of Contents
{{% toc %}}


## Security
### Certificates
---
#### Adding certificates to the TrustStore
Building a Dockerfile that needs to read {1..n} certificates from a given directory like `/usr/local/share/ca-certificates/extra` and load each certificate into Java's CA TrustStore.  This bash snippet will do the job:

```bash
for CERT_FILE in $(find /usr/local/share/ca-certificates/extra -type f); do
    echo "Adding $CERT_FILE to Java keystore"
    $JAVA_HOME/bin/keytool -importcert -trustcacerts -cacerts \
     -alias $(echo $CERT_FILE | xargs basename | sed 's#.*/##; s#[.][^.]*$##') \
     -file $CERT_FILE -keypass changeit -storepass changeit -noprompt
done
```
---
#### Investigating untrusted certificates
Running a Java application and receiving an error about an untrusted certificate - usually something like "unable to find valid certification path to requested target".  Add these options to the Java command line to spit SSL/TLS handshake and certificate data onto the console:
```bash
-Djavax.net.debug=ssl:handshake:verbose:keymanager:trustmanager
-Djava.security.debug=access:stack
```
Or, if you really like hard mode:
```bash
-Djavax.net.debug=all
```
---