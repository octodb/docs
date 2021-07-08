OctoDB on Swift
===============

We use the [OctoDB.swift](https://github.com/octodb/OctoDB.swift) wrapper

You can follow the instructions on the README to include it on your project

The repo lacks the binaries needed to use it. You need to copy these files to the OctoDB sub-folder:

    liboctodb.a
    libuv.a
    libbinn.a
    libsecp256k1-vrf.a

You can download pre-compiled static libraries or compile them by yourself

Here are the instructions to generate the files:

#### iOS

On Mac OSX you can clone the OctoDB repo and run:

    cd octodb
    ./makeios

#### Mac, Linux and Windows

Clone the OctoDB repo and run:

    cd octodb
    make static


### Example Code

```swift
import SQLite

let uri = "file:app.db?node=secondary&connect=server:port"
let db = try Connection(uri)

print("waiting approval...")
while (true) {
    let status = try db.scalar("pragma sync_status") as! String
    if status.contains("\"db_is_ready\": true") {
        print("the database is ready")
        break
    }
    sleep(1)  // use a timer instead
}

let stmt = try db.prepare("INSERT INTO users (email) VALUES (?)")
for email in ["betty@icloud.com", "cathy@icloud.com"] {
    try stmt.run(email)
}

for row in try db.prepare("SELECT id, email FROM users") {
    print("id: \(row[0]), email: \(row[1])")
}
```
