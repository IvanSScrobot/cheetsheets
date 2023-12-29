
Non-Ambari Cluster Installation Guide: https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.4.2/bk_installing_manually_book/content/ch_getting_ready_chapter.html

Automated Install with Ambari: https://docs.cloudera.com/HDPDocuments/Ambari-2.2.2.18/bk_ambari-installation/content/ch_Getting_Ready.html 

Ports: https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/cdh_ports.html 

Table 1.11. NameNode Heap Size Settings
| Number of Files in Millions | Total Java Heap (Xmx and Xms) | Young Generation Size (-XX:NewSize -XX:MaxNewSize) |
| --------------------------- | ----------------------------- | -------------------------------------------------- |
| < 1 million files           | 1024m                         | 128m                                               |
| 1-5 million files           | 3072m                         | 512m                                               |
| 5-10                        | 5376m                         | 768m                                               |
| 10-20                       | 9984m                         | 1280m                                              |
| 20-30                       | 14848m                        | 2048m                                              |
| 30-40                       | 19456m                        | 2560m                                              |
| 40-50                       | 24320m                        | 3072m                                              |
| 50-70                       | 33536m                        | 4352m                                              |
| 70-100                      | 47872m                        | 6144m                                              |
| 100-125                     | 59648m                        | 7680m                                              |
| 125-150                     | 71424m                        | 8960m                                              |
| 150-200                     | 94976m                        | 8960m                                              |



## Docs, links

Ambari official docs: https://ambari.apache.org/

Ambari docs on GitHub: https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/host-resources.md 

https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md

Ambari doc from Cloudera:  https://docs.cloudera.com/HDPDocuments/Ambari-2.6.1.5/bk_ambari-operations/content/working_with_hosts.html

Forum, what 'INSTALLED' status of a service means: https://community.cloudera.com/t5/Support-Questions/HDP-cluster-daily-health-report/td-p/205465

The Blog post: https://www.zylk.net/en/web-2-0/blog/-/blogs/getting-ambari-metrics-via-curl-and-ambari-rest-api?doAsUserId=fRCe/+1gy44=/blogs/rss?/blogs/rss?/-/blogs/rss?

The GitHub project with nice examples in Perl: https://github.com/HariSekhon/nagios-plugins#hadoop-ecosystem. Search for "check_ambari_" here: https://github.com/HariSekhon/nagios-plugins#hadoop-distributions

https://community.cloudera.com/t5/Community-Articles/Automate-HDP-installation-using-Ambari-Blueprints-Part-1/ta-p/246858

https://community.cloudera.com/t5/Community-Articles/Automate-HDP-installation-using-Ambari-Blueprints-Part-2/ta-p/246872

https://docs.cloudera.com/HDPDocuments/HDP3/HDP-3.1.4/index.html

https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/bk_ambari-installation/content/ambari_repositories.html

https://docs.cloudera.com/HDPDocuments/Ambari-2.7.4.0/bk_ambari-upgrade-major/content/ambari_upgrade_guide.html

# To manually clean servers
The use case: a datanode was taken out from HDP cluster, and now we need to clean it and bring back to the cluster. Ambari doesn't allow to re-install itself on the same nodes that were previously removed from the cluster, so we need to manually remove packages:

```
rm -rf /etc/yum.repos.d/ambari.repo /etc/yum.repos.d/HDP*
yum remove hive*
yum remove oozie*
yum remove pig*
yum remove zookeeper*
yum remove tez*
yum remove hbase*
yum remove ranger*
yum remove knox*
yum remove ranger*
yum remove storm*
yum remove hadoop*
 
for i in {2..61}; do mkdir -p /data$i/dfs/dn && chown -R hdfs:hadoop /data$i/dfs/dn && chmod -R 750 /data$i/dfs/dn; done
for i in {2..61}; do mkdir -p /data$i/yarn/local && chown yarn:hadoop  /data$i/yarn/local && chmod -R 755 /data$i/yarn/local; done
for i in {2..61}; do mkdir -p /data$i/yarn/log && chown yarn:hadoop  /data$i/yarn/log && chmod -R 755 /data$i/yarn/log; done
```

# To manually start datanode process when a node is not under Ambari management:
```
cd /data1/var/lib/ambari-agent/
./ambari-sudo.sh su hdfs -l -s /bin/bash -c 'ulimit -c unlimited ; /usr/hdp/3.1.4.0-315/hadoop/bin/hdfs --config /usr/hdp/3.1.4.0-315/hadoop/conf --daemon start datanode'
```
