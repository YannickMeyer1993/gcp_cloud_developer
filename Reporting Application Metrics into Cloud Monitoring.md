What is OpenCensus?
OpenCensus is an open source framework for metrics collection and distributed tracing. It offers the following benefits to its users:

* Low overhead data collection.
* Standard wire protocols and consistent APIs for handling metrics and trace data.
* Vendor interoperability via the OpenMetrics standard. OpenCensus can ingest into multiple backends in parallel, enabling incremental transitions and side-by-side comparison of backends.
* Correlation with traces and, in the future, log entries.

While multiple methods exist for reporting application metrics to Cloud Monitoring (see Other Methods for Reporting Application Metrics to Cloud Monitoring later in this lab), Google and Cloud Monitoring have adopted OpenCensus officially as the default mechanism for ingestion.

In the Cloud Console select Navigation menu > Compute Engine > VM instances, then click Create.
* Name: my-opencensus-demo
* Region: us-central1 (Iowa)
* Zone: us-central1-a
* Series: N1
* Machine type: n1-standard-1 (1vCPU, 3.75 GB memory)
* Identity and API access: Select Set access for each API, then for Stackdriver Monitoring API select Full from the dropdown menu.
* Firewall: Select both Allow HTTP traffic and Allow HTTPS traffic.

When your new Compute Engine VM instance launches, click SSH in line with the instance to open an SSH terminal to your instance.

`sudo curl -O https://storage.googleapis.com/golang/go1.10.2.linux-amd64.tar.gz`

`sudo tar -xvf go1.10.2.linux-amd64.tar.gz`

`sudo mv go /usr/local`

`sudo apt-get update`

`sudo apt-get install git`

`export PATH=$PATH:/usr/local/go/bin`

`go get go.opencensus.io`

`go get contrib.go.opencensus.io/exporter/stackdriver`

In the Cloud Console, click Navigation menu > Monitoring.
Run the Monitoring agent install script command in the SSH terminal of your VM instance to install the Cloud Monitoring agent.

`curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh`
`sudo bash add-monitoring-agent-repo.sh`

`sudo apt-get update`

`sudo apt-get install stackdriver-agent`

`curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh`
`sudo bash add-logging-agent-repo.sh`

`sudo apt-get update`

`sudo apt-get install google-fluentd`

`nano main.go`

```go
package main

import (
"fmt"
"math/rand"
"time"
)

func main() {
// Here's our fake video processing application. Every second, it
// checks the length of the input queue (e.g., number of videos
// waiting to be processed) and records that information.
for {
time.Sleep(1 * time.Second)
queueSize := getQueueSize()

                // Record the queue size.
                fmt.Println("Queue size: ", queueSize)
        }
}

func getQueueSize() (int64) {
// Fake out a queue size here by returning a random number between
// 1 and 100.
return rand.Int63n(100) + 1
}
```

`go run main.go`

To put in place the basic infrastructure for propagating metrics (stats) via OpenCensus, you need to define a measure, record it, then set up a view that allows the measure to be collected and aggregated.

You'll do all of the above in a couple of steps added to the main.go file.

Open your main.go file in your text editor.

Start by defining and recording the measure. Add the changes identified by ‘// [[Add this line]]’ or ‘// [[Add this block]] … // [[End: add this block]]’ to your file, then uncomment the added lines and remove the instructions:

```go
package main

import (
"context" // [[Add this line]]
"fmt"
"math/rand"
"time"

"go.opencensus.io/stats"  // [[Add this line]]
)

// [[Add this block]]
var videoServiceInputQueueSize = stats.Int64(
"my.videoservice.org/measure/input_queue_size",
"Number of videos queued up in the input queue",
stats.UnitDimensionless)
// [[End: add this block]]

func main() {
ctx := context.Background()  // [[Add: this line.]]

// Here’s our fake video processing application. Every second, it
// checks the length of the input queue (e.g., number of videos
// waiting to be processed) and records that information.
for {
time.Sleep(1 * time.Second)
queueSize := getQueueSize()

// Record the queue size.
// [[Add: next line.]]
stats.Record(ctx, videoServiceInputQueueSize.M(queueSize)) // [[Add]]
fmt.Println("Queue size: ", queueSize)
}
}

func getQueueSize() (int64) {
// Fake out a queue size here by returning a random number between
// 1 and 100.
return rand.Int63n(100) + 1
}
```

The go.opencensus.io/stats package contains all of the support you need to define and record measures.

In this example, videoServiceInputQueueSize is the measure. It's defined it as a 64-bit integer type. Each measure requires a name (the first parameter), a description, and a measurement unit.

A measure that has been defined also needs to be recorded. The stats.Record(...) statement sets the measure, videoServiceInputQueueSize, to the size queried, queueSize.

Bla weitere metric, wenn es klappen würde
