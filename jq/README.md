Real cheatsheet:  https://gist.github.com/olih/f7437fb6962fb3ee9fe95bda8d2c8fa4

Manual: https://stedolan.github.io/jq/manual/#Basicfilters

More examples: https://www.howtogeek.com/529219/how-to-parse-json-files-on-the-linux-command-line-with-jq/


To print the first SAL event from Avro's payload:
```
jq -s '.[0]'
```

To select events occured before a specific time: 
```
jq '. | select(.EventTime<=1609805693483)' payload
```

To select and obtain only "EventDetails"
```
jq '. | select(.EventTime<=1609805693483) | .EventDetails ' payload
```

To select and obtain only several fields: 
```
jq '. | select(.EventTime<=1609805693483) | .EventTime, .Trigger' payload
```

To apply several filters in selection, and obtain only several fields: 
```
jq '. | select(.EventTime<=1609805693483) | select(.AffectedEntities.array[].EntityType!="SystemUser") | .EventId, .AffectedEntities.array[].EntityType '  payload
```
