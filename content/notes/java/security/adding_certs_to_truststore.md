+++
title = "Adding Certificates to the TrustStore"
+++
Building a Dockerfile that needs to read {1..n} certificates from a given directory like `/usr/local/share/ca-certificates/extra` and load each certificate into Java's CA TrustStore.  This bash snippet will do the job:

```bash
for CERT_FILE in $(find /usr/local/share/ca-certificates/extra -type f); do
    echo "Adding $CERT_FILE to Java keystore"
    $JAVA_HOME/bin/keytool -importcert -trustcacerts -cacerts \
     -alias $(echo $CERT_FILE | xargs basename | sed 's#.*/##; s#[.][^.]*$##') \
     -file $CERT_FILE -keypass changeit -storepass changeit -noprompt
done
```