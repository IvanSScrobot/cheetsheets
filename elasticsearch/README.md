## How to Make Snapshots of Elasticsearch Data and Restore It 

https://qbox.io/blog/elasticsearch-data-snapshots-restore-tutorial/ 

## How to move elasticsearch data directory?

A. You need to move the elasticsearch folder, i.e. that's the folder which bears the same name as your cluster.name configured in the elasticsearch.yml file.

B. You need to modify the path.data setting in the elasticsearch.yml file to the new folder you've moved the data to.

So, say you are currently using /var/lib/elasticsearch and you want to move the data folder to /foo/bar, here is what you need to do:

```
mv /var/lib/elasticsearch /foo/bar
``` 
Then in elasticsearch.yml modify path.data to: ```path.data: /foo/bar```
You'll end up with your data being stored in /foo/bar/elasticsearch instead of /var/lib/elasticsearch. Make sure that the elasticsearch process can access your new folder.



Since Elastic Search 6.8, this practise of manual copying of the data directory has been invalidated; and, it is no longer guaranteed to work seamlessly. Even, if you manage to change the data directory location, and your cluster comes up without errors, the user defined indices created in the older data directory will not be detected. From 6.8, the reliable way to change the data directory path would be to take a snapshot from the older directory, and restore it in the new directory. Detailing the steps that one would perform to achieve this:

1. While still pointed to the older data path, register a snapshot registry and create a snapshot. The snapshot repository should be accessible from all the networks where copy and restore is desired. I have used a NFS path for this purpose. But, multiple other options like S3, Azure, HDFS are available for the snapshot storage. Refer https://www.elastic.co/guide/en/elasticsearch/reference/7.15/snapshots-register-repository.html for details on the other options.
```
PUT http://localhost:9200/_snapshot/mysnapshotregistry {
     "type": "fs",
     "settings": {
         "location": "/usr/share/elasticsearch/snapshotrepo",
         "compress": true
     }  }
```

2. Take backup of the older data by creating a snapshot in the registry created in step 1.
```
PUT http://localhost:9200/_snapshot/mysnapshotregistry/snapshot1?wait_for_completion=true

{   "indices": "employee, manager",   
    "ignore_unavailable": true,
    "include_global_state": false,   
    "metadata": {
    "taken_by": "binita",
    "taken_because": "testing creation of snapshot"
    } 
}
```
3. Stop the ElasticSearch service.
4. Update the data directory to the desired path in the elasticsearch.yml given by : path.data field. Remember to give ownership of the updated data directory to uid:gid = 1000:0. chown -R 1000:0 <updated data directory>
5. Start the ElasticSearch service.
6. Check that at this point of time, your cluster only contains the default indices. All user defined indices would be absent. Check: ```GET http://localhost:9200/_cat/indices```
7. Register a snapshot repository in the new data directory pointing to the same underlying NFS path referred in step 1.
```
 PUT http://localhost:9200/_snapshot/mysnapshotregistry2 {
     "type": "fs",
     "settings": {
         "location": "/usr/share/elasticsearch/snapshotrepo",
         "compress": true
     }  }
```
8. List all available snapshots from the registry created in step 7.
```
GET http://localhost:9200/_cat/snapshots/mysnapshotregistry2
```
9. Restore the snapshot into the updated data directory:
```
PUT http://localhost:9200/_snapshot/mysnapshotregistry2/snapshot1/_restore { "indices": "employee", "ignore_unavailable": true, "include_global_state": false, "index_settings": { "index.number_of_replicas": 0 }, "ignore_index_settings": [ "index.refresh_interval" ] }
```

