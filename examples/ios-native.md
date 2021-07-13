OctoDB on iOS Native
====================

Here are sample Xcode projects using the [SQLite.swift](https://github.com/stephencelis/SQLite.swift) wrapper

They include the free version of OctoDB with some limitations (1 table per database + no encryption). These limitations are not present on the full libraries


### SQLite.swift - SQL

This project has a simple interface using SQL:

```swift
try db.execute("CREATE TABLE IF NOT EXISTS Tasks (id INTEGER PRIMARY KEY, name TEXT, done INTEGER)")

try db.run("INSERT INTO Tasks (name, done) VALUES (?,?)", name, Int(done!))

for row in try db.prepare("SELECT * FROM Tasks") {
    let id = row[0] as! Int64
    let name = row[1] as! String
    let done = Int(row[2] as! Int64)
    print("id: \(id) name: \(name) done: \(done)")
}
```

[Download the project](http://octodb.io/download/OctoDB-iOS-Xcode-SQLite.swift-SQL.tar.gz)


### SQLite.swift - Objects

This project has an interface using objects:

```swift
let tasks = Table("tasks")
let id = Expression("id")
let name = Expression("name")
let done = Expression("done")

try db.run(tasks.create(ifNotExists: true) { t in
    t.column(id, primaryKey: true)
    t.column(name)
    t.column(done)
})

try db.run(tasks.insert(name <- taskname, done <- isdone!))

for task in try db.prepare(tasks) {
    print("id: \(task[id]) name: \(task[name]) done: \(task[done])")
}
```

[Download the project](http://octodb.io/download/OctoDB-iOS-Xcode-SQLite.swift-Objects.tar.gz)



### INSTRUCTIONS

The above projects implement secondary nodes. To test them you need to run one or more primary nodes (backend)

1.  Install OctoDB on a computer

2.  Start a SQLite shell as the primary node binding to a TCP address

3.  Modify the TCP address to connect to on the project code

4.  Build and run the project on the simulator or device


To open the database on the primary node we use a command like this:

    sqlite3 "file:test.db?node=primary&bind=tcp://0.0.0.0:1234"
