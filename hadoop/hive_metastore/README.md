# Hive HA Metastore

## Some links 
about Hive Metastore: 
https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.0.0/configuring-hive/content/hive_configure_the_apache_hive_metastore.html 

https://data-flair.training/blogs/apache-hive-metastore/#:~:text=What%20is%20Hive%20Metastore%3F,by%20using%20metastore%20service%20API.

##  Hive Metastore database
```
schematool -info -dbType postgres -userName hive -passWord hive -verbose 
#to re-init DB:
/data1/hdp/3.1.4.0-315/hive/bin/schematool -dbType postgres --verbose -initSchema -url jdbc:postgresql://${db_nodename}:5432/hive?useSSL=true -userName hive -passWord ${passwd}
```

Schematool docs: https://cwiki.apache.org/confluence/display/Hive/Hive+Schema+Tool