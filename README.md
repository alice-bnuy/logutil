# logutil

A minimal, structured logger for Go — easy to configure, human-friendly console output, and optional JSON file logs.

## Highlights
- Structured output (JSON to file) and human-readable console logs
- Add fields with `WithField(s)` and optional context support
- Log levels: DEBUG, INFO, WARN, ERROR, FATAL
- Simple daily rotation via filename format

## Installation
```sh
go get github.com/alice-bnuy/logutil
```

## Quick start
Create a logger, add fields, and log across levels:

```go
package main

import (
    "errors"
    "time"

    "github.com/alice-bnuy/logutil"
)

func main() {
    // 1) Create a local logger (you can also use the global one)
    logger, err := logutil.NewLogger(logutil.LoggerConfig{
        Level:          logutil.InfoLevel,   // or control via LOG_LEVEL
        LogDir:         logutil.LogsDirPath, // "logs" by default
        EnableConsole:  true,                // friendly console output
        EnableFile:     false,               // enable for structured JSON logs
        IncludeCaller:  true,                // include file:line:function
        FileNameFormat: "2006-01-02",        // simple daily rotation
    })
    if err != nil {
        panic(err)
    }
    defer logger.Close()

    // 2) Add fields (clones the logger with the new fields)
    reqLogger := logger.WithField("requestID", "abc-123")

    reqLogger.Info("Server started")
    reqLogger.Debug("This only shows in DEBUG level")

    // 3) Formatted log
    reqLogger.Infof("Connected to database in %d ms", 42)

    // 4) Error with context
    if err := doWork(); err != nil {
        reqLogger.WithField("job", "email-sender").ErrorWithErr("Failed to process job", err)
    }
}

func doWork() error {
    time.Sleep(120 * time.Millisecond)
    return errors.New("deadline exceeded")
}
```

## Global logger (optional)
Prefer global helpers? Initialize once and use shortcuts:

```go
_ = logutil.InitializeGlobalLogger(logutil.LoggerConfig{
    Level:         logutil.InfoLevel,
    LogDir:        logutil.LogsDirPath,
    EnableConsole: true,
    EnableFile:    true,
    IncludeCaller: true,
})
defer logutil.CloseGlobalLogger()

logutil.Infof("App started at %s", "localhost:8080")
logutil.WithField("userID", 42).Warn("High request rate detected")
```

## Tips
- Adjust level via `LOG_LEVEL` or set it in `LoggerConfig`.
- Enable `EnableFile` to emit structured JSON logs for aggregators.
- Use `WithContext(ctx)` to attach values like `requestID`, `userID`, etc.

## License
AGPL-3.0
