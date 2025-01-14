# Quick Start Guide to Write Spark Connect Client Application in Go

## Add Reference to `ocean-spark-connect-go` Library

In your Go project `go.mod` file, add `ocean-spark-connect-go` library:
```
require (
	github.com/sigmarkarl/ocean-spark-connect-go/v34 master
)
```

In your Go project, run `go mod tidy` to download the library on your local machine.

## Write Spark Connect Client Application

Create `main.go` file with following code:
```go
package main

import (
	"flag"
	"log"

	"github.com/sigmarkarl/ocean-spark-connect-go/v34/client/sql"
)

var (
	remote = flag.String("remote", 
		//"sc://localhost:15002",
		//"ws://localhost:8080",
		"wss://api.spotinst.io/ocean/spark/cluster/osc-239fd6f0/app/spark-connect-7702c-paste/connect?accountId=act-12f6b1b9;token=mytoken",
		"the remote address of Spark Connect server to connect to")
)

func main() {
	flag.Parse()
	spark, err := sql.SparkSession.Builder.Remote(*remote).Build()
	if err != nil {
		log.Fatalf("Failed: %s", err.Error())
	}
	defer spark.Stop()

	df, err := spark.Sql("select 'apple' as word, 123 as count union all select 'orange' as word, 456 as count")
	if err != nil {
		log.Fatalf("Failed: %s", err.Error())
	}

	log.Printf("DataFrame from sql: select 'apple' as word, 123 as count union all select 'orange' as word, 456 as count")
	err = df.Show(100, false)
	if err != nil {
		log.Fatalf("Failed: %s", err.Error())
	}

	rows, err := df.Collect()
	if err != nil {
		log.Fatalf("Failed: %s", err.Error())
	}

	for _, row := range rows {
		log.Printf("Row: %v", row)
	}

	err = df.Write().Mode("overwrite").
		Format("parquet").
		Save("file:///tmp/spark-connect-write-example-output.parquet")
	if err != nil {
		log.Fatalf("Failed: %s", err.Error())
	}

	df, err = spark.Read().Format("parquet").
		Load("file:///tmp/spark-connect-write-example-output.parquet")
	if err != nil {
		log.Fatalf("Failed: %s", err.Error())
	}

	log.Printf("DataFrame from reading parquet")
	df.Show(100, false)

	err = df.CreateTempView("view1", true, false)
	if err != nil {
		log.Fatalf("Failed: %s", err.Error())
	}

	df, err = spark.Sql("select count, word from view1 order by count")
	if err != nil {
		log.Fatalf("Failed: %s", err.Error())
	}

	log.Printf("DataFrame from sql: select count, word from view1 order by count")
	df.Show(100, false)
}
```

## Start Spark Connect Server (Driver)

Download a Spark distribution (3.4.0+), unzip the folder, run command:
```
sbin/start-connect-server.sh --packages org.apache.spark:spark-connect_2.12:3.4.0
```

## Run Spark Connect Client Application
```
go run main.go
```

You will see the client application connects to the Spark Connect server and prints out the output from your application.
