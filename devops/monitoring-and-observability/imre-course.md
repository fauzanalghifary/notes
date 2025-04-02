## Source

- https://courses.imrenagi.com/software-instrumentation

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