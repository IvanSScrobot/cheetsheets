## To change JKS password:
keytool -storepasswd -new unity1 -keystore dev-activity-log.jks
#unity1 is a new keystore password
 
## To change a private key password:

sudo keytool -keypasswd -new PASSWORD -storepass PASSWORD -keystore dev-activity-log.jks -alias star_dev_domain_net
# PASSWORD is a new private password, the command will ask for an old key password

## to create a truststore 

keytool -import -file  gr.cert  -alias star_dev_domain_net  -keystore path_to_truststore.jks -storepass your_passw

## to check a cert:

openssl s_client -showcerts -connect servername:9083
keytool --list -keystore  truststore.jks -v

## To list all known/available CA certs:

openssl storeutl -noout -text -certs /etc/ssl/certs/ca-certificates.crt


## How to Remove Imported Certificates From Java Keystore:

#/usr/j2se/bin/keytool -delete -alias smicacert -keystore /usr/j2se/jre/lib/security/cacerts


## Default Java truststore: 

If the system property:
javax.net.ssl.trustStore
is defined, then the TrustManagerFactory attempts to find a file using the filename specified by that system property, and uses that file for the KeyStore. If the javax.net.ssl.trustStorePassword system property is also defined, its value is used to check the integrity of the data in the truststore before opening it.
If javax.net.ssl.trustStore is defined but the specified file does not exist, then a default TrustManager using an empty keystore is created.

If the javax.net.ssl.trustStore system property was not specified, then if the file
<java-home>/lib/security/jssecacerts
exists, that file is used. (See The Installation Directory <java-home> for information about what <java-home> refers to.) Otherwise,
If the file
<java-home>/lib/security/cacerts
exists, that file is used.
(If none of these files exists, that may be okay because there are SSL cipher suites which are anonymous, that is, which don't do any authentication and thus don't need a truststore.)
