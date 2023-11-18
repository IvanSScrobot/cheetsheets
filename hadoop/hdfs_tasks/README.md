Docs: https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HDFSCommands.html#fsck

## Recover the lease for the file
When you do "hdfs dfs -cat file1" from the command line, you get the exception saying that it "Cannot obtain block length for LocatedBlock". Usually this means the file is still in being-written state, i.e., it has not been closed yet, and the reader cannot successfully identify its current length by communicating with corresponding DataNodes.

Suppose you're pretty sure the writer client is dead, killed, or lost connection to the servers. You're wondering what else you can do other than waiting.

```hdfs debug recoverLease -path <path-of-the-file> [-retries <retry-times>]```

This command will ask the NameNode to try to recover the lease for the file, and based on the NameNode log you may track to detailed DataNodes to understand the states of the replicas. The command may successfully close the file if there are still healthy replicas. Otherwise we can get more internal details about the file/block state. Please refer to https://community.hortonworks.com/questions/37412/cannot-obtain-block-length-for-locatedblock.html for discussion, especially answer made by @Jing Zhao.

This is a lightweight operation so the server should not crash if you run it. This is an idempotent operation so the server should not crash if you run the this command multiple times against the same path.

## Recover the lease for many files

https://blog.cloudera.com/understanding-hdfs-recovery-processes-part-1/
https://clouderatemp.wpengine.com/blog/2015/03/understanding-hdfs-recovery-processes-part-2/

```
# get a listing of the files with an open lease
hdfs fsck $PATH_TO_CHECK -openforwrite -files -blocks -locations | grep OPENFORWRITE | grep -v tmp |  awk '{print $1}' > files-to-recover-lease.txt

# loop over the listing of files with an open lease
for LINE  in $(cat files-to-recover-lease.txt); do echo "Recovering lease for: $LINE" && hdfs debug recoverLease -path $LINE -retries 2 && echo ""; done
```

## Trigger block report on DataNodes
You think a DataNode is not stable and you need to update, or you think there is a potential unknown bug in name-node (NN) replica accounting and you need to work around. As an operator, if you suspect such an issue, you might be tempted to restart a DN, or all of the DNs in a cluster, in order to trigger full block reports. It'd be much lighter weight if instead you could just manually trigger a full BR instead of having to restart the DN and therefore need to scan all the DN data dirs, etc.

hdfs dfsadmin -triggerBlockReport [-incremental] <datanode_host:ipc_port>
This command is to help you. If "-incremental" is specified, it will be incremental block report (IBR). Otherwise, it will be a full block report.

## Verify block metadata
Say you have a replica, and you don't know whether it's corrupt.

hdfs debug verify -meta <metadata-file> [-block <block-file>]
This command is to help you verify a block's metadata. Argument "-meta <metadata-file>" is the absolute path for the metadata file on the local file system of the data node. Argument "-block <block-file>" is an optional parameter to specify the absolute path for the block file on the local file system of the data node.

## Dump NameNode's metadata
You want to dump NN's primary data structures.

hdfs dfsadmin -metasave filename
This command is to save NN's meta data to filename in the directory specified by hadoop.log.dir property. "filename" is overwritten if it exists in the command line. The filename will contain one line for each of the following:

1. Datanodes heart beating with Namenode
2. Blocks waiting to be replicated
3. Blocks currently being replicated
4. Blocks waiting to be deleted

## Get specific Hadoop config
You want to know one specific config. You're smart to use Ambari UI while you need a web browser, which you don't always have as you SSH to the cluster from home. You turn to the configuration files and search for the config key. Then you find something special like in the configuration files. For example,

Embedded file substitutions. XInclude in the XML files are popular in Hadoop world, you know.
Property substitutions. Config A's value refers to config B's value, and again, config C's value.
You're in an urgent issue and tired of parsing the configuration files manually. You can do better. "Any scripting approach that tries to parse the XML files directly is unlikely to accurately match the implementation as its done inside Hadoop, so it's better to ask Hadoop itself." It's always been true.

```
hdfs getconf -confKey <key>
```

This command is to show you the actual, final results of any configuration properties as they are actually used by Hadoop. Interestingly, it is capable of checking configuration properties for YARN and MapReduce, not only HDFS. Tell your YARN friends about this. For more information, please refer to stackoverflow discussion, especially answers made by @Chris Nauroth.



OOZIE error: https://oozie.apache.org/docs/3.1.3-incubating/core/apidocs/src-html/org/apache/oozie/ErrorCode.html 

## find the biggest files:
```
hdfs fs -du /path | sort -n -r | head -n 20qq
```

## Look for corrupted avro files in HDFS
```
$ spark-shell
import com.databricks.spark.avro._re
import org.apache.avro.generic.GenericRecord
import org.apache.avro.mapred.{AvroInputFormat, AvroWrapper}
import org.apache.hadoop.io.NullWritable
import org.apache.hadoop.fs.{FileSystem, Path}
val pathObj = new Path("/events/2021/03/05")
  
val fs = FileSystem.get(sc.hadoopConfiguration)
val wantedFilesInFolder = fs.listStatus(pathObj).map(p => p.getPath.toString)
  
wantedFilesInFolder.foreach { file =>
    try { 
        val countEvents = sc.hadoopFile[ AvroWrapper[GenericRecord], NullWritable, AvroInputFormat[GenericRecord] ](file).count
        // println ("checked " + file + ", number of event count:" + countEvents)
    } catch {
        case _: Throwable => println ( "ERROR! corrupted file found " + file)
```

## avro
1)to run avro-tools:

```
java -jar avro-tools-1.9.1.jar
```

2)to get avro file from HDFS (to find a link fo an avro file, go to cloudera Web UI):

```
hdfs dfs -get /sin/sal/salqa1/events/2020/08/27
```

3) to extract the schema:
```
java -jar avro-tools-1.9.1.jar getschema filename.avro
```


4) to extract the content:

```
java -jar avro-tools-1.9.1.jar tojson filename.avro
```