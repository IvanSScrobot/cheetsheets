## Trino CLI

The Trino CLI utility is a very useful tool for debugging and inspecting the state of the supervision data store. Place the executable JAR file in the $PATH.

```
wget https://repo1.maven.org/maven2/io/trino/trino-cli/432/trino-cli-432-executable.jar
sudo mv trino-cli-432-executable.jar /usr/local/bin/trino
sudo chmod +x /usr/local/bin/trino
trino --version
# change target server according to environment
trino --server=${trino_coordinator}:8080 --catalog=catalog_name --user=trino
```

https://trino.io/docs/current/client/cli.html 