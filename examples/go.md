OctoDB in Go
============

The [octodb/go-sqlite3](https://github.com/octodb/go-sqlite3) wrapper allows Go programs to interface with the pre-installed OctoDB library


### Instructions

First follow the instructions to compile and install OctoDB or use the pre-compiled binaries for your platform

Then download the wrapper using this command:

    go get github.com/octodb/go-sqlite3

> **Note:** The wrapper uses the already installed SQLite library containing OctoDB. So it must be present on the device.

For more information check the instructions at the [go-sqlite3](https://github.com/octodb/go-sqlite3) repository


### Example Code

```go
package main

import (
  "database/sql"
  _ "github.com/octodb/go-sqlite3"
  "github.com/buger/jsonparser"
  "time"
)

func main() {
  db, err := sql.Open("sqlite3", "file:test.db?node=secondary&connect=tcp://127.0.0.1:1234")
  if err != nil {
    println("failed to open the database: " + err.Error())
    return
  }
        
  // check if the db is ready
  for {
    rows, err := db.Query("PRAGMA sync_status")
    if err != nil {
      println("failed to get sync_status: " + err.Error())
      return
    }
    rows.Next()
    var result string
    err = rows.Scan(&result)
    if err != nil {
      println("failed to scan the result: " + err.Error())
      return
    }
    rows.Close()
    println(result)
    isReady, _ := jsonparser.GetBoolean([]byte(result), "db_is_ready")
    if isReady {
      break
    }
    time.Sleep(1000 * time.Millisecond)
  }

  // now we can access the db
  println("OK, ready!")
}
```

Check also the go-sqlite3 [usage examples](https://github.com/octodb/go-sqlite3/tree/master/_example)
