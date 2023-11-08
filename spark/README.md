## Monitoring Spark

Spark health can be monitored in different ways: just checking that Spark daemons are up and running, manually checking Spark Web UI, or pulling Spark metrics. Spark metrics provide detailed info about Spark and applications running in Spark.

Spark’s metrics are decoupled into different instances corresponding to Spark components. Within each instance, you can configure a set of sinks to which metrics are reported. The following instances are currently supported:

* master: The Spark standalone master process.
* applications: A component within the master that reports on various applications.
* worker: A Spark standalone worker process.
* executor: A Spark executor.
* driver: The Spark driver process (the process in which your SparkContext is created).
* shuffleService: The Spark shuffle service.
* applicationMaster: The Spark ApplicationMaster when running on YARN.
* mesos_cluster: The Spark cluster scheduler when running on Mesos.

Each instance can report to zero or more sinks. Sinks are contained in the org.apache.spark.metrics.sink package:

* ConsoleSink: Logs metrics information to the console.
* CSVSink: Exports metrics data to CSV files at regular intervals.
* JmxSink: Registers metrics for viewing in a JMX console.
* MetricsServlet: Adds a servlet within the existing Spark UI to serve metrics data as JSON data.
* PrometheusServlet: (Experimental) Adds a servlet within the existing Spark UI to serve metrics data in Prometheus format.
* GraphiteSink: Sends metrics to a Graphite node.
- Slf4jSink: Sends metrics to slf4j as log entries.
* StatsdSink: Sends metrics to a StatsD node.
* Spark endpoint for MetricsServlet and PrometheusServlet:

| Component | Default port | Prometheus Endpoint               | JSON Endpoint                        |
| --------- | ------------ | --------------------------------- | ------------------------------------ |
| Driver    | 4040         | /metrics/executors/prometheus/    | /api/v1/applications/{id}/executors/ |
| Master    | 8080         | /metrics/applications/prometheus/ | /metrics/applications/json/          |
| Driver    | 4040         | /metrics/prometheus/              | /metrics/json/                       |
| Worker    | 8081         | /metrics/prometheus/              | /metrics/json/                       |
| Master    | 8080         | /metrics/master/prometheus/       | /metrics/master/json/                |


Note that both JSON and Prometheus metrics for Driver are not available if the default port 4040 is changed. 

