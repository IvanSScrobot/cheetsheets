## Some useful commands:
```
curl 'localhost:11000/ ...
GET /oozie/versions
GET /oozie/v0/admin/status
GET /oozie/v0/admin/os-env
GET /oozie/v0/admin/java-sys-properties
GET /oozie/v0/admin/configuration
``` 
 
For more details: https://oozie.apache.org/docs/3.1.3-incubating/WorkflowFunctionalSpec.html


## Resume suspended job:
```
sudo su oozie
oozie job -oozie https://localhost:<8080>/oozie -resume 14-20090525161321-oozie-joe
```

## Re-running a job that was killed

There are times where an execution of a coordinator job fails and is in the killed state. An easier way to re-run that instance is via the oozie rest API. If you only want to re-run a single failure without changing any configuration then this will be slightly easier than the above steps.

ssh to hadoopmgt1-${fqdn}

Retrieve the coordinator job id:

curl -s 'localhost:11000/oozie/v2/jobs?jobtype=coord&filter=status%3DRUNNING%3B' | python -mjson.tool | grep -a1 "coordJobId"

the above will list all of the running coordinator jobs, find the one that matches the job type you
want to re-run

Execute the job using the following command:
$ oozie job -rerun <coord_Job_id> -date <nominal date> -oozie http://localhost:11000/oozie

where <nominal date> is in the format 2017-11-02T08:00Z and is the workflow instance property oozie_sla_nominalTime
```
<property>
<name>oozie_sla_nominalTime</name>
<value>2017-11-02T08:00Z</value>
</property>
```

See https://oozie.apache.org/docs/3.1.3-incubating/DG_CoordinatorRerun.html

- All tasks for that job will be executed over again
- Coordinator Rerun will only use the original configs from first run.
- Coordinator Rerun will not re-read the coordinator.xml in hdfs.