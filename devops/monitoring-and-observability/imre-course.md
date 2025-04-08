## Source

- https://courses.imrenagi.com/software-instrumentation
- https://github.com/fauzanalghifary/logging-metrics-tracing-challenges
- https://github.com/fauzanalghifary/monitoring-and-observability-challenges

## Observability

- Logging
- Metrics
- Traces

## Logging

- Structured Logs
  - JSON, easier to parse

```
log := log.With()
    .WithField("user_id", userID)
    .WithField("user_name", userName)

log.Info("User logged in")
```

- Correlating logs
  - ensure that all logs from a single request are correlated
  - use a unique identifier for each request

```
func handler(w http.ResponseWriter, r *http.Request) {
    log := log.With().
      Str("request_id", uuid.New().String()).
      Logger()
      
    // creating a new context from the logger instance
    ctx := log.WithContext(r.Context())
    
    log.Info().Ctx(ctx).
        // ... omitting other code
        Msg("request received")
    doFirst(ctx)
}


func doFirst(ctx context.Context) {
    log := log.Ctx(ctx)
    log.Info().Msg("do first")
    doSecond(ctx)
}

func doSecond(ctx context.Context) {
    // one liner approach to use ctx for logging
    log.Ctx(ctx).Info().Msg("do second")
}

```

- Log Output
  - In container based application, logs are typically written to the container’s standard output stdout or standard error stderr, which is then collected by the log collector agent such as fluentd or fluentbit to collect all logs to centralized log system such as elasticsearch, Grafana Loki, or AWS CloudWatch, etc.
  - Now you might be wondering why is it useful to write logs to both console and file?

```
lf, err := os.OpenFile(
	"logs/app.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0666,
)
if err != nil {
	log.Fatal().Err(err).Msg("unable to open log file")
}

multiWriters := zerolog.MultiLevelWriter(os.Stdout, lf)
log.Logger = zerolog.New(multiWriters).With().Timestamp().Logger()
log.Info().Msg("hello world")
```

## Centralized Logging System

- Log collector agent
  - Collect all logs from applications and send them to a centralized location where you can see and query all the logs in one place.
  - Example: Fluentd, Fluentbit, Logstash, Grafana Loki, AWS CloudWatch Agent, etc.
- Collecting log files using Fluentbit
- Collecting container logs
- Grafana Loki
  - Log aggregation system to store and query the logs
  - Loki focuses on collecting, indexing, and searching logs, making it easier for users to analyze and troubleshoot issues within their applications and systems.
  - Logs from your applications are collected by a log collector agent and sent to Loki. Loki will act as persistent storage for the logs. 
  - You can define the log retention, storage type, etc. 
  - Later, you can query the logs using LogQL and visualize the logs using Grafana and LogCLI.
- Setting up Loki
- Sending logs to Loki
  - In previous module, you have seen how fluentbit collects logs from a file and container’s stdout and send it to its own stdout output. 
  - In this section, we will send the collected logs to loki.
  - query: `{app="http-service"} | json | line_format "{{.message}}"`
- LogQL
  - query language designed for querying logs in Loki

## Metrics and Prometheus

- Metrics
  - a measurement of a service captured at runtime.
  - few samples: CPU usage, memory usage, request count, error rate, number of orders per second, etc.
  - used to monitor, to understand the health of a service and to diagnose problems over time.
  - used to make decision how to act and improve the service.
  - to understand the root cause of issues during incidents/outage.
  - You need to look at the metrics from all of those components and try to match all the pieces together to understand the root cause of the problem.
  - Logs are a record of events that happened in your system. It is designed to be used for debugging and to get detailed explanation about what happened. While you can derived metrics from the logs, typically metrics are more efficient to be used for monitoring and to understand the current state of the system.
- Prometheus
  - particularly well-suited for monitoring cloud-native applications and microservices architectures.
- Setting up Prometheus
- Metrics types
  - Counter
  - Gauge
  - Histogram
  - Summary
- Metrics Query
- Prebuilt dashboard
  - https://grafana.com/grafana/dashboards/
- Exporter
  - software agent that collects metrics from systems and services and makes them available to Prometheus for monitoring and alerting purposes.
  - a standalone application that runs alongside the system or service that it is monitoring
  - How to get the data?
    - Instrumenting the application (`/metrics` endpoint)
    - External exporter
- Host & container metrics
- Instrumenting Go app

## Metrics and OpenTelemetry

- OpenTelemetry
  - an Observability framework and toolkit designed to create and manage telemetry data such as traces, metrics, and logs.
  - vendor- and tool-agnostic, meaning that it can be used with a broad variety of Observability backends, including open source tools like Jaeger and Prometheus, as well as commercial offerings like New Relic, Datadog, and Splunk.
  - OpenTelemetry is not an observability backend like Jaeger, Prometheus, or other commercial vendors. OpenTelemetry is focused on the generation, collection, management, and export of telemetry
  - A major goal of OpenTelemetry is that you can easily instrument your applications or systems, no matter their language, infrastructure, or runtime environment.
  - OpenTelemetry is designed to be vendor agnostic, meaning that you can use OpenTelemetry to instrument your code and export the metrics to any backend of your choice.
  - OpenTelemetry is not only for collecting metrics, but also traces and logs
  - So by using OpenTelemetry, you only need to learn a single set of APIs and conventions to collect all of your observability data.
- Instrumenting application
  - Meter Provider
  - Meter
  - Metric Exporter
  - Metric Instrument
- Automatic instrumentation
  - OpenTelemetry also provides a way to automatically instrument your code to generate metrics, traces, and logs for popular frameworks and libraries such as gRPC, HTTP, and PostgreSQL, Redis, etc
  - instead of doing many measurements, we can just install a library that will help you to instrument your code automatically