+++
title = 'How to import new TLS certificate into UNIFI wifi controller'
date = 2024-04-11T10:53:52+03:00
draft = false
+++
This is how I fix the unifi controller in terminal:

 - `openssl pkey -in example.key -traditional -out transformed-private.key`
 - `java -jar /usr/lib/unifi/lib/ace.jar import_key_cert transformed-private.key example.crt example.ca`
