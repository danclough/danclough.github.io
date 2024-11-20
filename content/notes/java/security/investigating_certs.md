+++
title = "Investigating untrusted certificates"
+++
Running a Java application and receiving an error about an untrusted certificate - usually something like "unable to find valid certification path to requested target".  Add these options to the Java command line to spit SSL/TLS handshake and certificate data onto the console:
```bash
-Djavax.net.debug=ssl:handshake:verbose:keymanager:trustmanager
-Djava.security.debug=access:stack
```
Or, if you really like hard mode:
```bash
-Djavax.net.debug=all
```