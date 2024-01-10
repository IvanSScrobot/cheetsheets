Official docs: https://www.postgresql.org/docs/current/runtime-config-connection.html#RUNTIME-CONFIG-CONNECTION-SSL



manually performed the following steps on sinci1 to set up secure access to ERS postgres. We are already doing similar steps for establishing secure access to ranger kms schema in hdp postgres.

To start Postgres server in SSL mode.

Added server certificate and private key to server's data directory.
server.key (private key) was taken from SecretServer and was kept at /data1/ers_db/pgsql/11.5/data_ers/. Only postgres user should have read access to the key.
 server.crt (server certificate) was taken from SecretServer and it is the first certificate in the chain.
```
sudo cp server.key server.crt /data1/ers_db/pgsql/11.5/data_ers/
sudo chown postgres:postgres /data1/ers_db/pgsql/11.5/data_ers/server.key
sudo chmod og-rwx /data1/ers_db/pgsql/11.5/data_ers/server.key
sudo chown postgres:postgres /data1/ers_db/pgsql/11.5/data_ers/server.crt
``` 

parameters set in postgresql.conf
ssl = on
ssl_cert_file = '/data1/ers_db/pgsql/11.5/data_ers/server.crt'
ssl_key_file = '/data1/ers_db/pgsql/11.5/data_ers/server.key'

Restart postgres
sudo systemctl restart postgresql-11.service 
Verify SSL connection
$ psql -U my_db_app -h db1.dev-domain.net my_db
Password for user my_db_app: 
psql (11.5)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.
Client Configuration


To check Postgres certs with openssl, we need version 1.1.1 It's not available in Centos 7, but it's in Ubuntu 18

openssl s_client -starttls postgres -connect db1.dev-domain.net:5432 -showcerts
 
# or
 
echo quit | openssl s_client -starttls postgres -connect localhost:5432  | openssl x509 -text -noout  |  grep -i not || true
https://dba.stackexchange.com/questions/108710/how-to-examine-postgresql-servers-ssl-certificate

https://stackoverflow.com/questions/63508872/upgrading-centos-7-to-openssl-1-1-1-by-yum-install-openssl11