# gzap

[![Go Reference](https://pkg.go.dev/badge/github.com/alfonmga/gzap.svg)](https://pkg.go.dev/github.com/alfonmga/gzap)

Some parts of the source code were extracted from from [gin-contrib/zap](https://github.com/gin-contrib/zap) and [things-go/gzap](https://github.com/things-go/gzap). I made minor changes, like [using a specific log level based on the response status code](https://github.com/alfonmga/gzap/blob/b8f265ead149e7275daf60ce0b8ef316503a616c/main.go#L139-L145), to adapt it to the needs of my projects.

## Install

Via go get tool

```bash
$ go get -u github.com/alfonmga/gzap
```

## Usage

```go
package main

import (
	"errors"
	"fmt"
	"time"

	"github.com/gin-gonic/gin"
	"go.uber.org/zap"

	"github.com/alfonmga/gzap"
)

func main() {
	r := gin.New()

	logger, _ := zap.NewProduction()

	// Add a ginzap middleware, which:
	//   - Logs all requests, like a combined access and error log.
	//   - Logs to stdout.
	//   - RFC3339 with UTC time format.
	r.Use(gzap.Logger(logger,
		gzap.WithCustomFields(
			gzap.String("app", "example"),
			func(c *gin.Context) zap.Field { return zap.String("custom field1", c.ClientIP()) },
			func(c *gin.Context) zap.Field { return zap.String("custom field2", c.ClientIP()) },
		),
		gzap.WithSkipLogging(func(c *gin.Context) bool {
			return c.Request.URL.Path == "/skiplogging"
		}),
		gzap.WithEnableBody(true),
        // gzap.WithBodyLimit(utils.KB*110),
	))

	// Logs all panic to error log
	//   - stack means whether output the stack info.
	r.Use(gzap.Recovery(logger, true,
		gzap.WithCustomFields(
			gzap.Immutable("app", "example"),
			func(c *gin.Context) zap.Field { return zap.String("custom field1", c.ClientIP()) },
			func(c *gin.Context) zap.Field { return zap.String("custom field2", c.ClientIP()) },
		),
	))

	// Example ping request.
	r.GET("/ping", func(c *gin.Context) {
		c.String(200, "pong "+fmt.Sprint(time.Now().Unix()))
	})

	// Example when panic happen.
	r.GET("/panic", func(c *gin.Context) {
		panic("An unexpected error happen!")
	})

	r.GET("/error", func(c *gin.Context) {
		c.Error(errors.New("An error happen 1")) // nolint: errcheck
		c.Error(errors.New("An error happen 2")) // nolint: errcheck
	})

	r.GET("/skiplogging", func(c *gin.Context) {
		c.String(200, "i am skip logging, log should be not output")
	})

	// Listen and Server in 0.0.0.0:8080
	r.Run(":8080")
}
```
