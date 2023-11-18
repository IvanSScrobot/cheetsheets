https://www.howtouselinux.com/post/certificate-chain
https://medium.com/@superseb/get-your-certificate-chain-right-4b117a9c0fce

Verify the list of certs in trust store:
```
keytool -list -v -keystore truststore.jks
openssl s_client -connect kafka1-ckafka1-int-nvan.dev-globalrelay.net:9094 -showcerts

openssl x509 -in certificate.pem -text -noout
```

Show public certs of a web-site a service (e.g. Kafka):
```
openssl s_client -connect kafka1-ckafka1-int-nvan.dev-globalrelay.net:9094 -showcerts

#the same with a trusted root certificate
openssl s_client -connect kafka3-ckafka1-int-nvan.dev-globalrelay.net:9094 -showcerts -CAfile ~/root-ca.pem
```

To verify certificates:
```
openssl verify ~/root-ca.pem

#pass a trusted CA certificate, like in GR 
openssl verify -CAfile ~/root-ca.pem ca2.pem

#pass a trusted CA certificate and an intermediate cert:
openssl verify -CAfile ca.pem -untrusted intermediate.cert.pem cert.pem
```
If we have multiple intermediate CA certficates, we can use the untrusted parameter multiple times like -untrusted intermediate1.pem -untrusted intermediate2.pem .

To create a chain of certificates:
```
cat cert.pem intermediate.pem > chain.pem
```

Who issues a cert:
```
openssl x509 -in ~/root-ca.pem -noout -issuer
```

To check the subject:
```
 openssl x509 -noout -subject -in ~/root-ca.pem
```
For the root cert, the issuer is equal to the subject. For a server cert, issuer = issuer of root cert, subject = name of the server.

To verify the hash sequence of certificates:
```
 openssl x509 -hash -issuer_hash -noout -in kafka4.pem 
 openssl x509 -hash -issuer_hash -noout -in ca2.pem
 openssl x509 -hash -issuer_hash -noout -in ~/root-ca.pem
```

https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html

