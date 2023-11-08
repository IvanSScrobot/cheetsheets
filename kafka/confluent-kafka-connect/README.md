## Rebalance kafka-confluent-connect consumers

1. Get current configuration:
```
curl ${consumer_host}:8083/connectors
curl ${consumer_host}:8083/connectors/connector-name/config | jq . > connector.config
```
2. Change connector.config to be a proper JSON by adding at the top of the file:
```
{
    "name": "connector-name",
    "config":
```
and "}" at the end of the file

3. Remove existing connector and re-create it:
```
curl -XDELETE ${consumer_host}:8083/connectors/connector-name
curl -XPOST ${consumer_host}:8083/connectors -H "Content-Type: application/json" -d @~/connector.config
```

4. Check the results:
```
curl  ${consumer_host}:8083/connectors/connector-name/status | jq .
```

### To restart a single task
```
curl -XPOST ${consumer_host}:8083/connectors/connector-name/tasks/1/restart
```

### Links:
https://docs.confluent.io/5.4.1/connect/references/restapi.html#rest-api-task-restart

https://docs.confluent.io/platform/current/connect/concepts.html#connectors

https://www.confluent.io/blog/incremental-cooperative-rebalancing-in-kafka/