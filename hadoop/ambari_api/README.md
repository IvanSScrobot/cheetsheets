[Ambari API Reference v1](https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md)


Intro in API: https://www.zylk.net/en/web-2-0/blog/-/blogs/getting-ambari-metrics-via-curl-and-ambari-rest-api


## To get specific fields from Namenode:
```
https://${manager_hostname}:8443/api/v1/clusters/HadoopCluster/hosts/${nn_hostname}/host_components/NAMENODE?fields=metrics/dfs/FSNamesystem

```

## To get an active NN:
```

CLUSTER_LOGIN="$CLUSTER_USER:$CLUSTER_PASSWD"
CLUSTER_NAME=$(curl -s -u "$CLUSTER_LOGIN" -H "X-Requested-By: ambari" -X GET "${CLUSTER_URL}/api/v1/clusters" | jq -r .items[0].Clusters.cluster_name)
 
NAME_NODES=$(curl -s -u "$CLUSTER_LOGIN" -H "X-Requested-By: ambari" -X GET "${CLUSTER_URL}/api/v1/clusters/${CLUSTER_NAME}/services/HDFS/components/NAMENODE" | jq -r '.host_components[].href')
 
for nn in $NAME_NODES
do
   href_hastate=$(curl -s -u "$CLUSTER_LOGIN" -H "X-Requested-By: ambari" -X GET $nn | jq -r '.host.href +" "+ .metrics.dfs.FSNamesystem.HAState')
   host_href=$(echo $href_hastate | tr -s ' ' | cut -f1 -d ' ')
   hastate=$(echo $href_hastate | tr -s ' ' | cut -f2 -d ' ')
   host_ip_name=$(curl -s -u "$CLUSTER_LOGIN" -H "X-Requested-By: ambari" -X GET $host_href | jq -r '.Hosts | .ip + " " +.host_name')
   ip=$(echo $host_ip_name | tr -s ' ' | cut -f1 -d ' ')
   node_name=$(echo $host_ip_name | tr -s ' ' | cut -f2 -d ' ')
   echo $node_name $ip $hastate
done
```

## To abort a hung ambari operation

https://community.cloudera.com/t5/Community-Articles/How-to-Abort-a-hung-ambari-operation/ta-p/244887

```
#get IDs
https:/${manager_hostname}:8443/api/v1/clusters/HadoopCluster/requests
 
#Abort jobs:
 
curl -u admin:admin -i -H "X-RequestedBy:ambari" -X PUT -d '{"Requests":{"request_status":"ABORTED","abort_reason":"Aborted by user"}}' https://${manager_hostname}:8443/api/v1/clusters/HadoopCluster/requests/<REQUEST_ID>;

```