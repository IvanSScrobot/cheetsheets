## Dead region servers

1. Stop Hbase
2. Login to zookeeper using #hbase zkcli ( with a valid hbase ticket )
3. Delete the /hbase-secure znode. rmr /hbase-secure
4. Sideline the entries under HDFS dir. hdfs dfs -mv /hbase/MasterProcWALs/*  /tmp. ( Not sure if this was done earlier )
5. Start Hbase


## HMaster fails to start
Scenario: Master boot cannot progress, defaults to standby until zone comes online (https://learn.microsoft.com/pt-br/azure/hdinsight/hbase/hbase-troubleshoot-start-fails#scenario-master-startup-cannot-progress-in-holding-pattern-until-region-comes-online)
Problem
HMaster fails to start due to the following warning:

```
hbase:namespace,,<timestamp_region_create>.<encoded_region_name>.is NOT online; state={<encoded_region_name> state=OPEN, ts=<some_timestamp>, server=<server_name>}; ServerCrashProcedures=true. Master startup cannot progress, in holding-pattern until region onlined. 

```

Cause
HMaster will check the WAL directory on the zone servers before bringing zones back online OPEN . In this case, if that directory wasn't present, it wasn't starting

Resolution
Create this dummy directory using the command:sudo -u hbase hdfs dfs -mkdir /hbase-wals/WALs/<wn fqdn>,16000,1622012792000

Restart the Ambari UI HMaster service.



## Useful commands:

```
#start region balancer :
sudo hbase shell 

hbase(main):001:0> balance_switch true 

#check for inconsistencies after HBase is up:
hbase hbck -details 

#Recover inconsistencies:
hbase hbck -repair 

#!Avoid running hbck with –repair flag.
hbase hbck –fixMeta mytable 
```

More about hbck https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/admin_hbase_hbck.html

More about hbck2 tools: https://apache.googlesource.com/hbase-operator-tools/+/refs/heads/revert-66-HBASE-23927-tempfile/hbase-hbck2/README.md , https://github.com/apache/hbase-operator-tools/tree/master/hbase-hbck2#master-startup-cannot-progress-in-holding-pattern-until-region-onlined


The offline Meta repair:
```
sudo -u hbase hbase org.apache.hadoop.hbase.util.hbck.OfflineMetaRepair 
```

Hbase Azure Troubleshooting: https://learn.microsoft.com/en-us/azure/hdinsight/hbase/hbase-troubleshoot-start-fails 