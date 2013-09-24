go-logging
==========

go-logging is a simple logging library for Go which supports logging to
systemd.

### Examples
#### Default
This example uses the default log to log to standard out and (if available) to systemd:
```go
package main
import (
	"github.com/sakana/go-logging/log"
)

func main() {
	log("Hello World.")
	log("There's nothing more to this program.")
}
```

#### Using Sinks and Formats
```go
package main

import (
	"github.com/sakana/go-logging/log"
	"os"
)

func main() {
	l := log(
		log( os.Stderr,
			"%s: %s[%d] %s\n",
			[]string{"priority", "executable", "pid", "message"}))
	l.Info("Here's a differently formatted log message.")
}
```

#### Custom Sink
This example only logs messages with priority `PriErr` and greater.
```go
package main

import (
	"github.com/sakana/go-logging/log"
	"os"
)

func main() {
	l := log(
		&PriorityFilter{
			log,
			log.WriterSink(os.Stdout, log.BasicFormat, log.BasicFields),
		})
	l.Info("This will be filtered out")
	l.Info("and not printed at all.")
	l.Error("This will be printed, though!")
	l.Critical("And so will this!")
}

type PriorityFilter struct {
	priority log.Priority
	target   log.Sink
}

func (filter *PriorityFilter) Log(fields log.Fields) {
	// lower priority values indicate more important messages
	if fields["priority"].(log.Priority) <= filter.priority {
		filter.target.Log(fields)
	}
}
```

### Fields
The following fields are available for use in all sinks:
```go
"prefix"       string              // static field available to all sinks
"seq"          uint64              // auto-incrementing sequence number
"start_time"   time.Time           // start time of the log
"time"         string              // formatted time of log entry
"full_time"    time.Time           // time of log entry
"rtime"        time.Duration       // relative time of log entry since started
"pid"          int                 // process id
"executable"   string              // executable filename
```
In addition, if `verbose=true` is passed to `New()`, the following (somewhat expensive) runtime fields are also available:
```go
"funcname"     string              // function name where the log function was called
"lineno"       int                 // line number where the log function was called
"pathname"     string              // full pathname of caller
"filename"     string              // filename of caller
```

### Logging functions
All these functions can also be called directly to use the default log.
```go
func (*Logger) Log(priority Priority, message string)
func (*Logger) Logf(priority Priority, format string, v ...interface{})
func (*Logger) Emergency(message string)
func (*Logger) Emergencyf(format string, v...interface{})
func (*Logger) Alert(message string)
func (*Logger) Alertf(format string, v...interface{})
func (*Logger) Critical(message string)
func (*Logger) Criticalf(format string, v...interface{})
func (*Logger) Error(message string)
func (*Logger) Errorf(format string, v...interface{})
func (*Logger) Warning(message string)
func (*Logger) Warningf(format string, v...interface{})
func (*Logger) Notice(message string)
func (*Logger) Noticef(format string, v...interface{})
func (*Logger) Info(message string)
func (*Logger) Infof(format string, v...interface{})
func (*Logger) Debug(message string)
func (*Logger) Debugf(format string, v...interface{})
```

### Acknowledgements
This package is a mostly-from-scratch rewrite of
[ccding/go-logging](https://github.com/ccding/go-logging) with some features
removed and systemd support added. 

