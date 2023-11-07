## Alert says "namenode last checkpoint error" for both active and standby Namenodes.

Careful! Ensure there are no running components that depend on HDFS. Then, on both nodes:
```
sudo su - hdfs
hdfs dfsadmin -fs hadoopnn[1..2]-${fqdn} -safemode enter
hdfs dfsadmin -fs hadoopnn[1..2]-${fqdn} -saveNamespace
hdfs dfsadmin -fs hadoopnn[1..2]-${fqdn} -safemode leave
#take a look at fsimage files:
ls -la /data1/dfs/nn/current/fsimage*
```
 
## HDFS Pending Deletion Block Alerts

This is not an issue in general. 

When a file is deleted in HDFS, the corresponding blocks are marked for deletion. However, the actual removal of these blocks may be delayed due to various reasons, such as cluster configuration, network issues, or resource constraints. During this period, the blocks are considered "pending deletion."

If there is a need to force deletion, first check the files:
```
hdfs fsck / -files -blocks -locations | grep PENDING_DELETE > files_for_deletion
hdfs fsck / -files -blocks -locations | grep UNDER_RECOVERY> files_under_recovery
```

## "NameNode is not formatted"

```
sudo -u hdfs hdfs namenode -format
```

## Cannot start secondary NameNode 
"Encountered exception loading fsimage
java.io.FileNotFoundException: No valid image files found"

"Directory /data1/hadoop/hdfs/journalnode/hadoopcluster is in an inconsistent state: Can't format the storage directory because the current directory is not empty."

# on the secondary namenode:
```
$ hdfs namenode -bootstrapStandby -force
```

The helpful post: [Can not start HDFS which its data was deleted externally](https://community.cloudera.com/t5/Support-Questions/Can-not-start-HDFS-which-its-data-was-deleted-externally/td-p/212566)

```
#Stop the Hdfs service if its running.
 
#Start only the journal nodes (as they will need to be made aware of the formatting)
 
#On the namenode (as user hdfs)
 
# su - hdfs
#Format the namenode
 
$ hadoop namenode -format
#Initialize the Edits (for the journal nodes)
 
$ hdfs namenode -initializeSharedEdits -force
#Format Zookeeper (to force zookeeper to reinitialise)
 
$ hdfs zkfc -formatZK -force
#Using Ambari restart the namenode
 
#If you are running HA name node then
 
#On the second namenode Sync (force synch with first namenode)
 
$ hdfs namenode -bootstrapStandby -force
#On every datanode clear the data directory which is already done in your case
 
#Restart the HDFS service.
```

## Sys DB and Information Schema not created yet

Ambari doesn't care about actual DB, it just checks the file /etc/hive/sys.db.created

```
if os.path.isfile("/etc/hive/sys.db.created"):
    result_code = 'OK'
    label = OK_MESSAGE
  else:
    result_code = 'CRITICAL'
    label = CRITICAL_MESSAGE
```